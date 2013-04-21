## Common Upgrading Scenarios

This section covers common error messages and other behavior that is likely to
occur when upgrading applications from Rails 3 to Rails 4.

---

@@@ text
`attr_accessible` is extracted out of Rails into a gem.
Please use new recommended protection model for params
(strong_parameters) or add `protected_attributes` to your
Gemfile to use old one.
@@@

Rails 4 extracted `attr_accessible` and `attr_protected` into the
`protected_attributes` gem. To fix the error, add the `protected_attributes`
gem to your `Gemfile`.

Read more about it in the [`strong_parameters`](#strong-paramters) section.

---

@@@ text
... was passed as :conditions but is not callable.
Pass a callable instead: `conditions: -> { where(approved: true) }`
@@@

Rails 4 pushes you to use callable objects when passing conditions to
`validates_uniqueness_of`. More information is available in the
[eager-evaluated scopes](#eager-evaluated-scopes) section.

---

@@@ text
DEPRECATION WARNING: The following options in your Post.has_many
:recent_comments declaration are deprecated: :conditions. Please use a scope
block instead.
@@@

Rails 4 deprecates many options to `has_many`, `has_one`, and `belongs_to`.
Instead of using `:order` and `:conditions`, for instance, you pass a scope
wrapped in a lambda. More information is available in the [eager-evaluated
scopes](#eager-evaluated-scopes) section.

---

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

---

@@@ text
config.eager_load is set to nil. Please update your
config/environments/*.rb files accordingly:
@@@

Rails 4 introduces new settings for thread safety. This deprecation warning is
prompting you to set the value of the new `config.eager_load` setting. Read
more about it in the [Thread Safety](#thread-safety) chapter.

---

@@@ text
DEPRECATION WARNING: Active Record Observers has been extracted out of Rails
into a gem.  Please use callbacks or add `rails-observers` to your Gemfile to
use observers.
@@@

Rails 4 extracts observers to a gem. Read more about it in the
[Observers](#observers) section.

---

@@@ text
DEPRECATION WARNING: The Active Record auto explain feature has been removed.

To disable this message remove the `active_record.auto_explain_threshold_in_seconds`
option from the `config/environments/*.rb` config file.
@@@

Rails 4 removes Auto-EXPLAIN for inefficient queries. Read more about it in the
[Auto-EXPLAIN queries](#auto-explain-queries) section.

---

@@@ text
The provided regular expression is using multiline anchors (^ or $), which may
present a security risk. Did you mean to use \A and \z, or forgot to add the
:multiline => true option? (ArgumentError)
@@@

Rails 4 does not allow the `^` and `$` anchors with a `validates_format_of`
validation. Read more about it in the [validates\_format\_of with ^ and
$](#validates-format-of) section.

---

@@@ text
You don't have dalli installed in your application. Please add it to your
Gemfile and run bundle install

`rescue in lookup_store': Could not find cache store adapter for
mem_cache_store (cannot load such file -- dalli) (RuntimeError)
@@@

Your application caches data in memcached. Rails 4 no longer uses the
`memcache-client` gem as it has been superceded by the `dalli` gem. Remove the
`memcache-client` gem from Gemfile and add `dalli` instead. Read more about
it in the [caching with memcached](#caching-with-memcached) section.
