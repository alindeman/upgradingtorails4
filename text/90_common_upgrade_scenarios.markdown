## Common Upgrading Scenarios

This section covers common error messages and other behavior that is likely to
occur when upgrading applications from Rails 3 to Rails 4.

---

<p><!-- Whitespace separation from HR --></p>

@@@ text
DEPRECATION WARNING: config.whiny_nils option is deprecated
and no longer works.
@@@

Rails 4 removed the whiny_nils feature. Read more about it in the [ActiveRecord
chapter](#whiny-nils).

To solve the deprecation warning, simply remove any lines that set
`config.whiny_nils`. Rails 3 added the configuration by default in
`config/environments/development.rb` and `config/environments/test.rb` by
default.

@@@ text
config.eager_load is set to nil. Please update your
config/environments/*.rb files accordingly:
@@@

Rails 4 introduces new settings for thread safety. This deprecation warning is
prompting you to set the value of the new `config.eager_load` setting. Read
more about it in the [Thread Safety](#thread-safety) chapter.
