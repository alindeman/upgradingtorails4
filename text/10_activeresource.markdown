## <a id="activeresource"></a>ActiveResource

Rails 4 no longer bundles ActiveResource by default. ActiveResource allows
applications to manipulate remote resources with a syntax similar to
ActiveRecord. Usually these resources are exposed by another Rails application
using RESTful HTTP API conventions.

ActiveResource has not been well-maintained. Before it was extracted out of the
main Rails repository, it was the source of many bugs in the issue tracker.
Even so, it did not get much attention from Rails core maintainers.

ActiveResource [stills exists in the Rails organization on
GitHub](http://github.com/rails/activeresource), but as of Rails 4, it is
independent from Rails' release cycle. It is possible that ActiveResource 4.0.1
could be released independently of Rails 4.0.1, for instance.

To continue using ActiveResource, add it explicitly in `Gemfile`:

@@@ ruby
# Gemfile
gem 'activeresource', github: 'rails/activeresource'
@@@

Cutting the ties a bit from Rails has already been beneficial for
ActiveResource. A new group of maintainers, led by Jeremy Kemper, is
working to fix issues and add new features.

ActiveResource 4 adds support for associations (`has_many`, `has_one`, and
`belongs_to`). Callbacks (e.g., `after_create` and `after_save`) have also been
added. ActiveResource 4 can now be used with APIs that do not follow Rails
RESTful conventions. Finally, many bugs have been fixed.

More information about ActiveResource's new direction and plans for the future
are documented in Guillermo Iguaran's blog post [ActiveResource is dead, long
live ActiveResource](http://yetimedia.tumblr.com/post/35233051627/activeresource-is-dead-long-live-activeresource).
