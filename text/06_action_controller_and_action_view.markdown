## ActionController & ActionView

### Authenticity Tokens for Remote Forms

Rails protects applications from a range of security issues. By default, Rails
requires that forms submitted via HTTP verbs other that GET be accompanied with
an *authenticity tokens*.

The *authenticity token* prevents a malicious user from tricking the user's
browser into making an authenticated, destructive action from a source other
than your application. This attack is called *cross site request forgery*
(CSRF).

As an aside, this is one reason why it is important that all requests that
change server state use verbs other than GET.

Rails automatically embeds the an automatically generated authenticity token in
forms.

![authenticity_token embedded in a form](../images/authenticity_token.png)

This behavior mostly stays the same in Rails 4; however, **Rails will no longer
embed the authenticity token in forms submitted via AJAX by default**
(`form_for @obj, remote: true`).

Instead, Rails 4 will inject the authenticity token into the request via
JavaScript as its on its way out to the server.

Remote forms submitted via JavaScript will be unaffected; however, **the
form will no longer gracefully degrade if JavaScript is disabled** as they
previously did in Rails 3.

If a user with JavaScript disabled attempts to submit a form created with
`form_for @obj, remote: true`, an `InvalidAuthenticityToken` error will be
raised:

![InvalidAuthenticityToken error](../images/authenticity_fail.png)

The change makes it possible to add forms to fragment caches. Forms with
an embedded authenticity token cannot be cached because the token is unique
for each user.

If you need to preserve graceful degradation of AJAX forms in Rails 4:

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
