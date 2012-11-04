## ActionController & ActionView

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
