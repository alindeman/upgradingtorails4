## Introduction

Even though Rails 4 is not yet released, it is important to start thinking
about how easy or difficult it will be to upgrade your applications. It may
be possible to nip certain problems in the bud now to save pain later.

My goal with this book is to provide a clear path for upgrading Rails 3
applications to Rails 4.

The book is split into three major parts:

1. Deprecations: features removed in Rails 4 or marked for removal in a future
   release
2. New Features: novel features in Rails 4 to be aware of, especially when
   upgrading
3. Common Upgrading Scenarios: specific issues you might encounter when
   upgrading a Rails application, and how to fix them

Also included is an [Upgrade Checklist](#upgrade-checklist) which links to
important sections that are relevant when upgrading an application. I recommend
you read through the Deprecations section entirely first, then use the
checklist as a guide when upgrading your applications.

### Beta

This book is an evolving work in progress, so please excuse any typos or
errors.

It is also possible that since Rails 4 is currently in development, certain
versions of the book may contain inaccuracies. I keep a close watch on the
commit log for Rails 4, so these sorts of errors should be resolved in future
updates.

If you find errors or have questions, please reach out to me via email at
<rails4@andylindeman.com>.

### Website

To purchase additional copies of this book--or a team license--visit
<http://upgradingtorails4.com>. Thank you so much for supporting my work.

### Cover Photo

The cover photo is copyright [Jeramey
Jannene](http://www.flickr.com/photos/compujeramey/168102810/) and used under
the [CC BY 2.0
license](http://creativecommons.org/licenses/by/2.0/).

### Changelog

#### 0.10.1: April 29, 2013

* Bumps gem versions throughout the book for the release of Rails 4.0.0.rc1.

#### 0.10: April 26, 2013

* Added a [Before You Upgrade](#before-you-upgrade) section highlighting some
  things to consider before upgrading, and moves the upgrading checklist to
  this section.
* Updated information about [binaries and bundle binstubs](#binstubs), including
  information about how to upgrade if you are already using binstubs.
* Removed details about the [`rails test` command](#new-testing) since it has
  been removed in favor of running tests through `rake` (as is the norm in
  Rails 3).
* Rails 4 [removes the `:assets` group from Gemfile](#no-assets-group). Be sure
  to move gems like sass-rails and coffee-rails to the top level.
* Updated information about [encrypted cookies](#encrypted-cookies). Rails 3
  cookies are upgradable to Rails 4 cookies, but the API looks different
  than it did in Rails 4.0.0.beta1.
* Discussed [upgrading RSpec](#rspec) earlier in the book: tests are critical
  to a smooth upgrade process.
* Noted that using memcached in Rails 4 [requires the `dalli`
  gem](#caching-with-memcached).

#### 0.9: March 21, 2013

* Added information about [XML parsing being extracted to a gem](#xml-parsing).
* This handbook assumes that applications being upgraded are [running the latest
  version of Rails 3.2](#rails-32).
* Added information about the [`rails test` command](#new-testing) which is
  recommended over `rake test`.
* Recommended [raising an error](#unpermitted-attributes) when an unpermitted
  parameter is received by **strong_paramters**.
* Added details about [`validates_confirmation_of` now adding error messages to
  the \_confirmation field](#validates-confirmation-of).

#### 0.8: February 25, 2013

* Rails 4.0.0.beta1 has shipped! Updated the [Upgrading Rails
  Itself](#upgrading-rails-itself) section to use beta1.
* Added information about [Ruby 2.0.0](#ruby-193), which Rails 4 recommends.
* Added information about [concerns](#concerns), shared pieces of functionality
  for models and controllers.
* Added information about [breaking changes in the asset
  pipeline](#asset-pipeline).
* Added information about [Auto-EXPLAIN for inefficient queries being
  removed](#auto-explain-queries).
* Added information about [validates\_format\_of and the ^ and $ regular
  expression anchors](#validates-format-of).

#### 0.7: February 13, 2013

* Added information about [the new convention of adding binstubs for `rails`,
  `rake`, and any other commonly used binaries](#binstubs).
* Added information about [performance tests being extracted to a
  gem](#performance-tests).
* Added information about [ActiveRecord::Base#update](#update).

#### 0.6: January 7, 2013

* Added a first cut of the [upgrade checklist](#upgrade-checklist).
* Added information about the [changes to testing in Rails 4](#testing).
* Removed `journey` from the generated `Gemfile` since `journey` has been
  integrated into Rails.
* Added information about [the extraction of the `encode` option for
  `mail_to`](#actionview-encoded_mail_to) to the `actionview-encoded_mail_to`
  gem.
* Noted that [routing concerns can be used in Rails
  3.2](#routing-concerns-in-rails32).

#### 0.5: December 25, 2012

* Added information about [ActionController::Live](#action-controller-live) and
  streaming server-sent events.
* Removed `Rails.queue` chapter since it has been [punted out of Rails
  4](https://twitter.com/dhh/status/281421220417781760). If it returns as a gem
  or plugin, I will consider adding the material back.
* Updated the [Checking for Incompatible Gems](#incompatible-gems) section
  now that Draper 1.0.0 (prerelease) supports Rails 4.
* Added information about [declarative ETags](#etagger).

#### 0.4: December 10, 2012

* Added information about [ActiveResource no longer being bundled with
  Rails](#activeresource).
* Added information about [gotchas when using turbolinks](#turbolinks-gotchas).
* Added information about [ActiveRecord::Relation#not](#relation-not).
* Added information about [JSON serialization in Rails 4](#json-serialization).
* Added basic information about [routing concerns](#routing-concerns). I may
  add some more details later, after I come up with better examples.
* Added information about [asynchronous ActionMailer](#async-actionmailer).

#### 0.3: December 3, 2012

* Added information about [changes in ActiveSupport](#activesupport).
* Added information about [observers being extracted into a gem](#observers).

#### 0.2: November 28, 2012

* Clarified that [`match` has not been removed, but instead cannot be used
  without `:via`](#routing-match).
* Added information about [encrypted cookies](#encrypted-cookies) and the
  encrypted cookie session store.
* Added information about [the addition of the PATCH verb](#patch-verb).
* Added information about [turbolinks](#turbolinks).

#### 0.1: November 26, 2012

* First beta release to customers and technical reviewers.

### Technical Reviewers

Thank you to these folks for reviewing and giving technical feedback:

* [Steve Klabnik](http://steveklabnik.com/)
* [Caius Durling](http://caius.name/)
* [Ashish Dixit](https://twitter.com/tundal45)
* [Ernie Miller](http://erniemiller.org/)
* [Darcy Laycock](http://sutto.net/)
* [Sean Marcia](http://arlingtonruby.org)
* [Giles Bowkett](http://gilesbowkett.blogspot.com/)
