## Introduction

Even though Rails 4 is not yet released, it is important to start thinking
about the how easy or difficult it will be to upgrade your applications. It may
be possible to nip certain problems in the bud now to save pain later.

My goal with this book is to provide a clear path for upgrading Rails 3
applications to Rails 4.

The book is split into three major parts:

1. Deprecations: features removed in Rails 4 or marked for removal in a future
   release
2. New Features: novel features in Rails 4 to be aware of, especially when
   upgrading
3. Common Upgrading Scenarios: specific issues you might encounter when
   upgrading a Rails application and how to fix them

Also included is an [Upgrade Checklist](#upgrade-checklist) which links to
important sections that are relevant when upgrading an application. I recommend
you read through the Deprecations section entirely first, then use the
checklist as a guide when upgrading your applications.

### Beta

This book is an evolving work in process, so please excuse any typos or errors.

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

#### 0.1: November 26, 2012

* First beta release to customers and technical reviewers.

#### 0.2: November 28, 2012

* Clarified that [`match` has not been removed, but instead cannot be used
  without `:via`](#routing-match).
* Added information about [encrypted cookies](#encrypted-cookies) and the
  encrypted cookie session store.
* Added information about [the addition of the PATCH verb](#patch-verb).
* Added information about [turbolinks](#turbolinks).

#### 0.3: December 3, 2012

* Added information about [changes in ActiveSupport](#activesupport).
* Added information about [observers being extracted into a gem](#observers).

#### 0.4: December 10, 2012

* Added information about [ActiveResource no longer being bundled with
  Rails](#activeresource).
* Added information about [gotchas when using turbolinks](#turbolinks-gotchas).
* Added information about [ActiveRecord::Relation#not](#relation-not).
* Added information about [JSON serialization in Rails 4](#json-serialization).
* Added basic information about [routing concerns](#routing-concerns). I may
  add some more details later, after I come up with better examples.
* Added information about [asynchronous ActionMailer](#async-actionmailer).

#### 0.5: December 25, 2012

* Added information about [ActionController::Live](#action-controller-live) and
  streaming server-sent events.
* Removed `Rails.queue` chapter since it has been [punted out of Rails
  4](https://twitter.com/dhh/status/281421220417781760). If it returns as a gem
  or plugin, I will consider adding the material back.
* Updated the [Checking for Incompatible Gems](#incompatible-gems) section
  now that Draper 1.0.0 (prerelease) supports Rails 4.
* Added information about [declarative ETags](#etagger).

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

#### 0.7: February 13, 2013

* Added information about [the new convention of adding binstubs for `rails`,
  `rake`, and any other commonly used binaries](#binstubs).
* Added information about [performance tests being extracted to a
  gem](#performance-tests).
* Added information about [ActiveRecord::Base#update](#update).

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
