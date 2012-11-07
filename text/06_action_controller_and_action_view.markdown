## ActionController & ActionView

### strong_parameters

Securing a Rails application normally involves protecting against a class
of attacks known as mass-assignment vulnerabilities.

The method for doing so has changed in Rails 4.

Consider a `User` model in Rails 3:

@@@ ruby
# app/models/user.rb
class User < ActiveRecord::Base
  attr_accessible :first_name, :last_name
end
@@@

`User` is protected against mass-assignment vulnerabilities. Specifically, methods
that accept a hash of attributes can only set `first_name` and `last_name`.

Consider a controller that creates users:

@@@ ruby
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  def create
    @user = User.new(params[:user])
    if @user.save
      redirect_to root_url
    else
      render action: "new"
    end
  end
end
@@@

The use of `attr_accessible` on the model prevents a client from attempting to
set attributes other than `first_name` and `last_name`.

For instance, if a malicious person manipulates the request so that
`params[:user] = { admin: "true" }`, the `admin` attribute on the newly created
user will *not* be set and the user will *not* become an admin. `User.new`
strips out any attributes not specified in the list of `attr_accessible`
attributes, protecting the application from mass-assignment vulnerabilities.

However, enforcing mass-assignment protection at the model layer makes models
difficult to work with in other parts of the application that do not involve
user input and therefore do not need mass-assignment protection. You cannot,
for instance, simply create a new admin user at the console with
`User.new(admin: true)` even though there is no danger in this situation.

Because of this, Rails 4 adds mass-assignment protection at the controller layer
by default via the **strong_parameters** gem.

In Rails 4, there is no mass-assignment protection in the model by default:

@@@ ruby
# app/models/user.rb
class User < ActiveRecord::Base
end
@@@

Instead, the controller is responsible for permitting only appropriate
attributes through:

@@@ ruby
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  def create
    @user = User.new(user_params)
    if @user.save
      redirect_to root_url
    else
      render action: "new"
    end
  end

  private

  def user_params
    params.require(:user).permit(:first_name, :last_name)
  end
end
@@@

**strong_parameters** requires that controllers expressedly slice out
attributes that clients are allowed to set before they are passed to the
model.

**strong_parameters** adds two notable methods to `params`:

<table>
  <tr>
    <th>
      require(:user)
    </th>
    <td>
      Requires that <code>params[:user]</code> is present (non-empty).
      If it <em>is</em> empty, an "HTTP 400 Bad Request" response is
      immediately returned.
    </td>
  </tr>
  <tr>
    <th>
      permit(:first_name, :last_name)
    </th>
    <td>
      Specifies that only <code>params[:user]</code> may only contain the
      <code>:first_name</code> and <code>:last_name</code> keys. If other keys
      are present, they are removed.
    </td>
  </tr>
</table>

Furthermore, Rails 4 will raise an error if a controller attempts to pass
`params` to a model method without explicitly permitting attributes via
`permit`.

#### Upgrading

While `attr_accessible` has been removed from Rails 4, it can be brought back
via the **protected_attributes** gem.

I recommend adding the **protected_attributes** gem to an application's Gemfile
when upgrading:

@@@ ruby
# Gemfile
gem 'protected_attributes', github: 'rails/protected_attributes'
@@@

The gem restores `attr_accessible` and disables the requirement that
controllers call `params.permit` before passing hashes to model methods.

Because **protected_attributes** and **strong_parameters** take very different
approaches to enforcing mass-assignment protection, it is not advisable to
attempt to use them together.

I recommend simply using **protected_attributes** when upgrading applications
that use `attr_accessible`, and continuing to use `attr_accessible` for
mass-assignment protection. While at the end of the day, it is your decision as
an engineer, I think in many cases it will be both high risk and low reward to
attempt to transition a sizable application from one paradigm to the other at
this time.

New applications, however, should use **strong_parameters** and
controller-enforced mass-assignment protection as this appears to be the
convention going forward.

### Authenticity Tokens for Remote Forms

Rails protects applications from a range of security issues. By default, Rails
requires forms submitted via HTTP verbs other than GET be accompanied with
an *authenticity token*.

An *authenticity token* prevents a malicious user from tricking the user's
browser into making an authenticated, destructive action from a source other
than your application. This attack is called *cross site request forgery*
(CSRF).

(As an aside, this is one reason why it is important for all requests that
change server state use verbs other than GET.)

Rails automatically embeds the an automatically generated authenticity token in
forms.

![authenticity_token embedded in a form](../images/authenticity_token.png)

This behavior mostly stays the same in Rails 4; however, **Rails will no longer
embed an authenticity token into remote forms submitted via Ajax by default**
(i.e., forms created with `form_for @obj, remote: true`).

Instead, Rails 4 will inject an authenticity token into the request via
JavaScript as it is on its way out to the server.

Remote forms submitted in browsers that support JavaScript will be unaffected;
however, **remote forms will no longer gracefully degrade if JavaScript is
disabled** as they previously did in Rails 3.

Specifically, a remote form submitted in a browser with JavaScript disabled
will raise an `InvalidAuthenticityToken` error:

![InvalidAuthenticityToken error](../images/authenticity_fail.png)

While this change may initially appear to have no upsides, forms without
embedded authenticity tokens may now be added to fragment caches. Without an
authenticity token, the form markup is no longer specific to a user, so it
can be cached and reused for all users, speeding up the application.

If you need to preserve graceful degradation of AJAX forms in Rails 4, you
can do either one of the following:

* Globally re-enable embedding tokens by adding
  `config.action_view.embed_authenticity_token_in_remote_forms = true` in
  `config/application.rb`.
* Re-enable embedding tokens on a case-by-case basis by passing
  `authenticity_token: true` in the options to `form_for` (e.g., `form_for
  @obj, remote: true, authenticity_token: true`).

### Caching

TODO: Rails 4 removes action caching and page caching.

### Disabling Asset Pipeline

TODO: Disabling the asset pipeline is easier. Simply remove the gems; no need
for configuration options.
