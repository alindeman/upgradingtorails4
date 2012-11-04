## Common Upgrading Scenarios

This section covers common error messages and other behavior that is likely to
occur when upgrading applications from Rails 3 to Rails 4.

---

@@@ text
undefined method `whitelist_attributes=' for ...
undefined method `mass_assignment_sanitizer=' for ...
@@@

TODO: Remove whitelist_attributes from config/application.
TODO: Remove mass_assignment_sanitizer from config/environments/development.rb
and config/environments/test.rb

@@@ text
DEPRECATION WARNING: config.whiny_nils option is deprecated
and no longer works.
@@@

@@@ text
config.eager_load is set to nil. Please update your
config/environments/*.rb files accordingly:
@@@

