## ActionController and ActionView

### <a id="strong-parameters"></a>strong_parameters

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

Next, consider a controller that creates users:

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

Because of this, Rails 4 adds mass-assignment protection at the controller
layer by default via the **strong_parameters** plugin, and these features
are available automatically in Rails 4.

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

**strong_parameters** requires that controllers explicitly slice out
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

#### Non-Scalar Values

**strong_parameters** requires special syntax to permit non-scalar values
such an array or hash.

Consider a system where users have interests such as "programming" or
"rugby." A user may have many interests:

@@@ ruby
# app/models/user.rb
class User < ActiveRecord::Base
  has_many :interests
end
@@@

<p></p>

@@@ ruby
# app/models/interest.rb
class Interest < ActiveRecord::Base
  belongs_to :user
end
@@@

Users are able to select their interests from a list in a multi-select:

@@@ ruby
# app/views/users/edit.html.erb
<%= form_for @user do |f| %>
  <%= f.label :interests, "Select your interests" %>
  <%= f.collection_select :interest_ids, Interest.all, :id, :name, {}, :multiple => true %>

  <%= f.submit %>
<%- end %>
@@@

When the form is submitted, the list of interest model IDs will arrive as
an array nested inside the `params` hash:

@@@ ruby
# params
{ :interest_ids => ["1", "2", "5"] }
@@@

To permit an array of IDs with **strong_parameters**, pass a hash key with the
name of the attribute and a value of an empty array:

@@@ ruby
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  # ...

  private

  def user_params
    params.require(:user).permit(:interest_ids => [])
  end
end
@@@

**strong_parameters** can accept deeply nested arrays and hashes, but needs to
be aware of their structure. For in-depth coverage of the syntax, scan the
examples in the [**strong_parameters**
README](https://github.com/rails/strong_parameters/blob/master/README.md).

#### <a id="unpermitted-attributes"></a>Behavior for Unpermitted Paramters

Any parameter that is not `permit`ted is removed from the hash of parameters
passed through **strong_parameters**. The model will simply not see it at all.

Furthermore, in the development and test environments, a message is logged
to the log file and `rails server` output. However, no error is raised:
the request continues.

@@@ text
# logs/development.log
Unpermitted parameters: admin
@@@

I found this lack of an error frustrating in development mode. In my Rails 4
applications, I would sometimes add a new field to the model and to a form, but
forget to tweak the `xxx_params` method in the controller.

Submitting the form with the new field would not result in an error, but the
new attribute would not be populated either. And, of course, it would take me
several minutes to figure out where the problem lay.

I now recommend changing the behavior of **strong_parameters** to raise an
exception when an unpermitted parameter is received, though only in
development mode. To do so, add a line to `config/environments/development.rb`:

@@@ ruby
# config/environments/development.rb
Widgets::Application.configure do
  # ...

  config.action_controller.action_on_unpermitted_parameters = :raise
end
@@@

With the configuration option set to `:raise`, it is much more obvious during
development when you forget to add a new field to the `permit`ted list.

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
controllers call `params.require.permit` before passing hashes to model methods.

Because **protected_attributes** and **strong_parameters** take very different
approaches to enforcing mass-assignment protection, it is not advisable to
attempt to use them together.

I recommend simply using **protected_attributes** when upgrading applications
that use `attr_accessible`, and continuing to use `attr_accessible` for
mass-assignment protection. At the end of the day, it is your decision as an
engineer, but I think in many cases it will be both high risk and low reward to
attempt to transition a sizable application from one paradigm to the other at
this time.

New applications, however, should use **strong_parameters** and
controller-enforced mass-assignment protection, as this appears to be the
convention going forward.

#### Use in Rails 3.2

**strong_parameters** is one of the many features of Rails 4 that can be used
by Rails 3.2 applications today.

I recommend that any new Rails 3.2 applications bring in the **strong_parameters**
gem and use it instead of `attr_accessible`.

To get started, simply add **strong_parameters** to the application's Gemfile:

@@@ ruby
# Gemfile
gem 'strong_parameters'
@@@

Install it via Bundler:

@@@ text
$ bundle install
@@@

Finally, disable the configuration option where Rails will add an implicit
`attr_accessible` if one is not specified explicitly:

@@@ ruby
# config/application.rb

# Change from true (default in 3.2.8) to false
config.active_record.whitelist_attributes = false
@@@

The application is now free to ditch `attr_accessible` in the model layer in
favor of `params.require.permit` in the controller layer. The application will already
be following the convention for Rails 4 and beyond.

### <a id="authenticity-tokens-in-remote-forms"></a>Authenticity Tokens for Remote Forms

Rails protects applications from a range of security issues. By default, Rails
requires forms submitted via HTTP verbs other than GET be accompanied with
an *authenticity token*.

An *authenticity token* prevents a malicious user from tricking the user's
browser into making an authenticated, destructive action from a source other
than your application. This attack is called *cross site request forgery*
(CSRF).

(As an aside, this is one reason why it is important for all requests that
change server state use verbs other than GET.)

Rails automatically embeds an automatically generated authenticity token in
forms.

![authenticity_token embedded in a form](../images/authenticity_token.png)

This behavior mostly stays the same in Rails 4; however, **Rails will no longer
embed an authenticity token into remote forms submitted via Ajax**
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

If your application does not add remote forms to fragment caches, you can
address this error and preserve graceful degradation of AJAX forms in Rails 4
by:

* Globally re-enabling embedded tokens by adding
  `config.action_view.embed_authenticity_token_in_remote_forms = true` in
  `config/application.rb`, or
* Re-enabling embedded tokens on a case-by-case basis by passing
  `authenticity_token: true` in the options to `form_for` (e.g., `form_for
  @obj, remote: true, authenticity_token: true`).

### Caching

Rails 4 extracts page caching and action caching to gems.

Page caching saved the response to a request and persisted the data to the
filesystem. The next time a request for the same controller action comes in,
the response would be served directly by the web server (rather than the
Rails application server).

Action caching was similar, but controller filters in the Rails application
would still be run. This allowed, for instance, the application to verify
that a user had access to the content before it was served.

Both page and action caching are complicated, and often better implemented by
out-of-process proxies. In Rails 4, applications that want to continue using
page caching and action caching will need to bring in the
`actionpack-page_caching` and `actionpack-action_caching` gems respectively
by adding them to `Gemfile`.

Notably, fragment caching--where smaller pieces of a view are cached--is *not*
deprecated in Rails 4. In fact, fragment caching is even improved. More on that
in the section describing [cache_digests](#cache-digests).

### <a id="xml-parsing"></a>XML Parsing

The release of Rails 3.2.11 was prompted in part because a vulnerability in XML
parsing allowed the clients to send requests that create symbols within the
Ruby environment, as well as evaluate embedded YAML. These and other closely
related vulnerabilities turned out to be extremely critical, allowing arbitrary
code to be executed in the context of unpatched Rails applications.

These vulnerabilities, combined with the fact that JSON seems to have
overshadowed XML as the lingua franca for Rails APIs, prompted the Rails core
team to extract XML parsing into a gem in Rails 4.

If your application accepts XML in the request body (note: this is distinct
from *rendering* XML as output), you will need to pull in the
`actionpack-xml_parser` gem:

@@@ ruby
# Gemfile
gem 'actionpack-xml_parser'
@@@

If your application does not need to accept XML input, I recommend leaving the
gem out in order to reduce the possibility that XML parsing will be the vector
for a yet undiscovered security vulnerability.

Applications that simply *render* XML do not need the gem.

### <a id="actionview-encoded_mail_to"></a>actionview-encoded\_mail\_to

Rails 3 included a little-known feature to obfuscate links to email addresses.
This is useful because spambots are known to troll the Internet looking for
these `mailto:` links.

The obfuscation was enabled by passing the `encode` option to `mail_to`,
usually in a view:

@@@ ruby
# app/views/about/index.html.erb
Contact us <%= mail_to "andy@example.com", "via email", encode: "hex" %>
@@@

When passed `encode: "hex"`, Rails will obfuscate the email address with HTML
entities (in this case `andy@example.com` is transformed into
`&#109;&#97;&#105;&#108;&#116;&#111;&#58;%61%6e%64%79@%65%78%61%6d%70%6c%65.%63%6f%6d`).

While a real browser will interpret the encoded email address without any
problems, it might trip up a naive spambot.

`encode` can also be set to `"javascript"`. In this case, the email address is
encoded into its hex form, but also embedded in a JavaScript snippet that is
invoked when the link is clicked. This will likely trip up a higher percentage
of spambots, but does now require JavaScript.

In Rails 4, support for `encode` has been extracted to a gem:
`actionview-encoded_mail_to`. To enable the feature, simply add the gem to
`Gemfile`:

@@@ ruby
# Gemfile
gem 'actionview-encoded_mail_to'
@@@
