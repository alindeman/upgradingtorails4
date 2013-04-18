## <a id="thread-safety"></a>Thread Safety

Traditionally, Rails applications have been deployed on web application servers
that only run code on a single thread. In that case, the number of requests an
application server can process concurrently is the number of web application
server processes that are running.

More and more developers are deploying to threaded web application servers like
[Rainbows!](http://rainbows.rubyforge.org/) and [puma](http://puma.io/) where a
single application server can process concurrent requests without booting the
application multiple times in separate processes. Java application servers have
always taken advantage of threads, and JRuby can piggyback on their success
with servers like [trinidad](https://github.com/trinidad/trinidad) and
[Torquebox](http://torquebox.org/).

---

### config.threadsafe! and config.eager_load

Rails 3 required a configuration flag, `config.threadsafe!`, to enable
applications to work properly with threaded web application servers. This flag
was usually set in `config/environments/production.rb`.

Rails 4 deprecates the `config.threadsafe!` option because Rails applications
are now threadsafe by default as long as both `config.cache_classes` and
`config.eager_load` are set to `true`.

`config.cache_classes` might seem familiar to you because it exists in Rails 3.
It is usually set to `false` in the development environment so that code is
reloaded every request as you develop new features. It is normally set to
`true` in the test and production environments because reloading code in
those environments is not necessary and would only hinder performance.

`config.eager_load` is introduced in Rails 4. When set to `false`, certain
application code is not loaded until it is needed. When set to `true`, on
the other hand, the entire application is loaded when the application boots.

When upgrading to Rails 4, you will want to add a setting for
`config.eager_load` to each of `config/environments/development.rb`,
`config/environments/test.rb`, and `config/environments/production.rb`.

Loading the entire application often takes many seconds. For this reason, it
makes sense to avoid doing so in the development and test environments. With
`config.eager_load` *disabled* in development and test, the entire application
does not have to boot if a request or test only needs a subset of the code.

Set `config.eager_load` to `false` in `config/environments/development.rb`
and `config/environments/test.rb`:

@@@ ruby
# config/environments/development.rb AND
# config/environments/test.rb
Widgets::Application.configure do
  # ...

  config.eager_load = false
end
@@@

Unfortunately, loading the entire application upfront is the only way for the
application to be thread-safe. Having multiple threads vying to load the same
piece of code at the same time is a recipe for disaster.

So, in production, it makes sense to pay the price of eager loading the
application in order to be able to run it on a threaded web application server.

Set `config.eager_load` to `true` in `config/environments/production.rb`:

@@@ ruby
# config/environments/production.rb
Widgets::Application.configure do
  # ...

  config.eager_load = true
end
@@@

With `config.cache_classes` and `config.eager_load` enabled in production,
the application is thread-safe and can run on a web application server that
uses threads.

This is a huge win: it may be possible for you to handle many more requests
concurrently without needing more memory. Check out the [puma](http://puma.io/)
landing page for more information.

A threaded web application server also makes it feasible to keep long-running
connections open to web clients in order to stream data to them over many
minutes or even hours. The section on
[ActionController::Live](#action-controller-live) scratches the surface about
what is possible in that realm.
