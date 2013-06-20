## <a id="before-you-upgrade"></a>Before You Upgrade

This handbook focuses on upgrading a Rails 3 application to a Rails 4
application.

To use it most effectively, there are a few things you should know first,
before you dive into your `Gemfile` and run `bundle update rails`.

### Test Suite

Higher level tests, those that drive your application from the outside, become
essential during an upgrade process. Any medium to large application needs at
least some coverage before an upgrade to Rails 4 is attempted.

If your application has controller tests (also called functional tests) or
integration tests (also called request specs or feature specs), you are off to
a great start. Run these tests as you go through the upgrade process, to verify
breaking changes don't get pushed out to production. If your application lacks
these kinds of tests, consider writing some before upgrading.

### <a id="upgrade-checklist"></a>Checklist

The handbook goes into detail about each major change in Rails 4. I recommend
you read through the first section entirely before attempting an upgrade.

Afterward, or after you have upgraded an application or few, you should use
this checklist to navigate to specific sections in the order that I expect you
will need them during the upgrade process.

1. [Upgrade to Ruby 1.9.3 or 2.0.0](#ruby-193)
1. [Upgrade to the latest version of Rails 3.2](#rails-32)
1. [Upgrade bundler](#bundler)
1. [Check for gem incompatibilities using `rails4_upgrade`](#rails4_upgrade)
1. [Upgrade Rails itself](#upgrading-rails-itself)
1. [Add gems that have extracted functionality from Rails 3](#deprecation-gems)
1. [Upgrade **rspec-rails** if you use RSpec as your testing framework](#rspec)
1. [Add **dalli** if you cache data in memcache](#caching-with-memcache)
1. [Add binaries and binstubs for `rails` and `rake`](#binstubs)
1. [Upgrade plugins to gems or move code to `lib/`](#plugins)
1. [Tweak any routes that use `match` without `:via => :verb`](#routing-match)
1. [Audit any chained uses of `Relation#order`, as new orders are now prepended rather than appended](#relation-order)
1. [Change `^` and `$` to `\A` and `\z` (respectively) when using `validates_format_of`](#validates-format-of)
1. [Decide whether graceful degradation of remote forms is important to your application and, if so, enable the option to embed authenticity tokens in forms](#authenticity-tokens-in-remote-forms)
1. [Add any image assets in `lib/` or `vendor/` to the precompilation list](#precompiled-images)

Many configuration options have changed or been removed in Rails 4. You will
need to make changes to files like `config/application.rb`,
`config/environments/development.rb`, `config/environments/test.rb`, and
`config/environments/production.rb`:

1. [Remove the `whiny_nils` setting from all environment configuration files](#whiny-nils)
1. [Remove the `auto_explain_threshold_in_seconds` setting from all environment configuration files](#auto-explain-queries)
1. [Add new thread-safety configuration options](#thread-safety)
1. [Update JavaScript and CSS compression options](#js-compress)

Some functionality from earlier versions of Rails has been deprecated: while
your application may continue to operate correctly, you will see warnings.
After you have addressed the concerns in the first checklist, consider
addressing deprecated features:

1. [Modernize Rails 2 finder syntax](#rails2-finder-syntax)
1. [Modernize dynamic finders](#dynamic-finders)
1. [Change eager-evaluated scopes to use lambdas](#eager-evaluated-scopes)
1. [Audit any uses of `Relation#all`](#relation-all)
1. [Address any uses of `Relation#includes` with conditions on the joined table](#relation-includes)

After your Rails 4 application is running smoothly in production, consider
making these changes:

1. [Upgrade digitally signed cookies to encrypted cookies](#encrypted-cookies)
1. [Remove any extracted gems that your application does not need](#extracted-gems)
