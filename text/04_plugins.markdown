## Plugins

<!-- TODO: Was Rails 2.3 the release? -->
The term *Rails plugin* can mean multiple things. In this chapter, I mean Rails
plugin means a piece of shared code that was installed via the `script/plugin
install` command and resides in the `vendor/plugins` directory. These types of
plugins were in use prior to Rails 2.3, but they fell out of popularity
after that release.

If your application uses gems to manage dependencies, you're in the clear.
Simply list the contents of the `vendor/plugins` directory in a Rails
application to verify. If that directory is empty or nonexistant, Rails plugins
won't hinder your upgrade path and you can safely skip the content in this
chapter.

However, if you find that an application is using plugins, you'll need to do a
bit of work to successfully upgrade.

* http://opensoul.org/blog/archives/2009/10/05/how-to-gemify-your-rails-plugins/

TODO
