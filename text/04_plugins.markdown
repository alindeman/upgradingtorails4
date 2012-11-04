## Plugins

The term *Rails plugin* can mean multiple things. In this chapter, Rails plugin
refers to a piece of shared code that was installed via the `script/plugin
install` command and resides in the `vendor/plugins` directory. Applications
might have these sorts of plugins even though they fell out of popularity soon
after Rails 2.3 was released.

**If an application uses gems to manage dependencies, nothing needs to be
done.** Simply list the contents of the `vendor/plugins` directory in a Rails
application: if the directory is empty or nonexistent, you can safely skip the
content in this chapter.

However, if an application is using plugins, you'll need to do a bit of work to
successfully upgrade. Read on!

### The Easy Option

The easiest way to work around the removal of plugins in Rails 4 is to, well,
remove your plugins!

Many popular libraries that once existed as plugins have already been
"gemified" by the Ruby and Rails communities. If they have, it may be very
easy to replace the plugin with its gem counterpart.

For instance, the [`acts_as_tree`](https://github.com/rails/acts_as_tree)
plugins is a popular way to model tree structures in ActiveRecord models. Many
Rails applications pulled it in as a plugin.

The good news is that `acts_as_tree` has been converted to a gem by open source
contributors. The gem's source code is [available on
GitHub](https://github.com/amerine/acts_as_tree) and released on
[rubygems.org](http://rubygems.org/gems/acts_as_tree).

![acts_as_tree on rubygems.org](../images/acts_as_tree.png)

When dealing with plugins while upgrading to Rails 4, first [search
rubygems.org for a gem of the same name as the plugin](http://rubygems.org/).
If nothing turns up, [perform a similar search on
GitHub](https://github.com/search). Look for repositories with descriptions
like "Gem version of ...".

![searching for acts_as_tree on rubygems.org](../images/acts_as_tree.png)

The process for replacing a plugin with a gem is straightforward. First,
remove the plugin:

@@@ text
$ git rm -rf vendor/plugins/acts_as_tree

# If not using git:
$ rm -rf vendor/plugins/acts_as_tree
@@@

Next, add the gem version of the plugin to `Gemfile`. 

@@@ ruby
# Gemfile
gem 'acts_as_tree'
@@@

If the gem is only available on GitHub, but not rubygems.org, pull the gem in
via git instead:

@@@ ruby
# Gemfile
gem 'acts_as_tree', github: 'amerine/acts_as_tree'
@@@

Install the new gem with Bundler:

@@@ text
$ bundle install
@@@

Finally, wrap your work up in a commit (if using git):

@@@ text
$ git add Gemfile Gemfile.lock
$ git commit -m 'Replaced acts_as_tree plugin with gem'
@@@

Repeat these steps for each plugin. If you run into a situation where you
cannot find a gem counterpart for a plugin, read on to the next section.

### Adapting Rails Plugins

If the Rails community has not created a gem version of a plugin or the plugin
is highly customized for the needs of an application, you must do one of two
things:

1. Create a gemified version yourself.
2. Move the plugin from `vendor/plugins/` to `lib/` and require it manually.

An entire handbook like this one could be written about creating and managing
gems, so #1 will not be covered here. It is, however, the more robust solution,
and you can check out the following posts to find out more about it:

* [How to Gemify your Rails
  Plugins](http://opensoul.org/blog/archives/2009/10/05/how-to-gemify-your-rails-plugins/)
* [How to convert a Rails plugin into a
  gem](http://patshaughnessy.net/2009/12/12/how-to-convert-a-rails-plugin-into-a-gem)

The #2 option of moving the plugin from `vendor/plugins/` to `lib/` is more
straightforward and can serve as a solid temporary solution so that you can
proceed with the upgrade.

First, move the plugin's directory:

@@@ text
$ git mv vendor/plugins/acts_as_tree lib/

# If not using git:
$ mv vendor/plugins/acts_as_tree lib/
@@@

Next, add an *initializer* to load the plugin. Save it to
`config/initializers/acts_as_tree.rb`:

@@@ ruby
# config/initializers/acts_as_tree.rb
ActiveSupport::Dependencies.autoload_paths <<
  "#{Rails.root}/lib/acts_as_tree/lib"

require "#{Rails.root}/lib/acts_as_tree/init"
@@@

The first two lines add the plugin's files to the Rails autoload path. Rails
will automatically load code from that directory as it's needed. The last line
runs the plugin initialization code: since Rails no longer runs it, it must
be required manually.

Finally, commit the changes:

@@@
$ git add config/initializers/acts_as_tree
$ git commit -m 'Loads acts_as_tree in Rails 4'
@@@

Follow these same steps for each plugin that needs to be adapted.

TODO: Find and test a plugin with rake tasks

TODO: Find and test a plugin with generators
