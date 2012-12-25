## ActionController and ActionView

### <a id="action-controller-live"></a>ActionController::Live

By default, Rails renders the entire view template and layout before sending
any data back to a browser. While simple, this approach adds to the
critical path of time it takes before the resulting page appears to the
user.

Rails 3.1 introduced optional streaming templates: when enabled, parts of the
view (e.g., the layout) are rendered and sent back to users's browser before
the *entire* view is rendered. Importantly, the browser can start downloading
and parsing assets like JavaScript, CSS stylesheets, and images while the rest
of the view is being generated. More information about this kind of streaming
is available in the [`ActionController::Streaming` API
documentation](http://api.rubyonrails.org/classes/ActionController/Streaming.html).

However, `ActionController::Streaming` does not give developers full control
over the streaming process, and is geared mostly toward speeding up short-lived
requests. Recent developments in HTML5 and JavaScript make it more
appealing to keep long-lived connections open between servers and browsers. To
achieve that alongside Rails, though, many developers have been forced to use a
separate [Sinatra](http://www.sinatrarb.com/) application or even reach for
technologies like [node.js](http://nodejs.org/).

Rails 4 introduces `ActionController::Live` which gives developers full control
over sending arbitrary data to browsers, including over long-held connections.

An example is the easiest way to explain this new feature in detail.

As a prerequisite, a Rails application using `ActionController::Live` should be
configured for thread safety. Rails 4 makes this straightforward, as I explain
in the [Thread Safety](#thread-safety) section earlier. Thread safety is
already the default in production, but the development configuration needs to
be adjusted by editing `config/environments/development.rb`:

@@@ ruby
# config/environments/development.rb
Widgets::Application.configure do
  # ...

  # Add this line: disables mutex around each request
  config.middleware.delete ::Rack::Lock

  # Change from false to true!
  config.eager_load = true
end
@@@

Next, make sure to use a thread-safe web application server like
[puma](http://puma.io/); avoid process-based servers like
[unicorn](http://unicorn.bogomips.org/) which can only handle as many
concurrent requests as there are processes. To install puma, simply add it to
`Gemfile` and run `bundle exec puma -p 3000` instead of `rails server`:

@@@ ruby
# Gemfile
gem 'puma'
@@@

Now, consider a controller with `ActionController::Live` enabled. It will
send [server-sent events](http://en.wikipedia.org/wiki/Server-sent_events)
every second, incrementing a counter each time.

@@@ ruby
# app/controllers/timer_controller.rb
class TimerController < ApplicationController
  include ActionController::Live

  def tick
    response.headers["Content-Type"] = "text/event-stream"

    begin
      seconds = 0
      loop do
        response.stream.write("data: #{seconds}\n\n")

        seconds += 1
        sleep 1
      end
    rescue IOError
      # client disconnected
    ensure
      response.stream.close # cleanup
    end
  end
end
@@@

As shown in the `TimerController` example above, live streaming is enabled by
including the `ActionController::Live` module. When enabled, the controller
action can write directly to `response.stream`.

The consumer of the timer API can simply be a static page in `public/`:

@@@ 
<!DOCTYPE html>
<html>
  <!-- public/timer.html -->
  <head>
    <title>Timer Example</title>

    <script type="text/javascript" src="http://code.jquery.com/jquery.js"></script>
    <script type="text/javascript">
      $(function() {
        var timerSource = new EventSource("/tick");
        timerSource.onmessage = function(e) {
          $("#timer").text(e.data);
        };
      });
    </script>
  </head>
  <body>
    Timer: <span id="timer"></span> seconds.
  </body>
</html>
@@@

Any time a server-sent event is received, the `#timer` text is updated.

Finally, wire everything up by adding a route for the `tick` controller action:

@@@ ruby
# config/routes.rb
Widgets::Application.routes.draw do
  # ...

  get "/tick" => "timer#tick"
end
@@@

Start or restart `puma` (`bundle exec puma -p 3000`), and navigate to
<http://localhost:3000/>. The timer should start counting up every second:

![Timer at 7 seconds](../images/timer_7.png)

![Timer at 22 seconds](../images/timer_22.png)

Voila! Consider using `ActionController::Live` to implement desktop-like
applications that react rapidly (avoiding AJAX polling, for instance), or to
stream JSON to users on connections with lower bandwidth or higher latency
(e.g., cellular data).

### <a id="cache-digests"></a>Cache Digests

Nesting fragment caches--often called *russian doll caching*--are an effective
way to achieve a speed up while keeping view code relatively simple.

Consider an application that manages shopping wishlists: a customer creates
a wishlist with a title and adds items he or she wishes to buy to it.

The application might contain a view that renders the entire wishlist
and caches the fragment of HTML:

@@@ erb
<%# app/views/wishlists/show.html.erb %>
<%- cache ["v1", @wishlist] do %>
  <h1><%= @wishlist.title %></h1>

  <ul>
    <%= render @wishlist.items %>
  </ul>
<%- end %>
@@@

`"v1"` is used to allow developers to easily *bust* the cache for every
wishlist by incrementing the string to `"v2"` (and later to "`v3`" and so on)
any time the markup inside the `cache` block is changed. If the `"v1"` were not
present or not incremented, the application might display a mishmash of cached
content with the older markup alongside updated content with newer markup.

For instance, a designer adding a CSS class to the wishlist title header tag
would also need to increment the string to make sure every wishlist title
is rerendered with the class after the new application code was deployed:

@@@ erb
<%# app/views/wishlists/show.html.erb %>
<%- cache ["v2", @wishlist] do %>
  <h1 class="wishlist-title"><%= @wishlist.title %></h1>

  <ul>
    <%= render @wishlist.items %>
  </ul>
<%- end %>
@@@

Next, consider a partial that renders each item. It might look very similar
to the wishlist view:

@@@ erb
<%# app/views/wishlists/_item.html.erb %>
<%- cache ["v1", @item] do %>
  <li><%= @item.name %> (<%= @item.price %>)</li>
<%- end %>
@@@

This strategy is called russian doll caching because of its use of nested
templates, each of which is cached. The wishlist is an outer "doll," and each
item is an inner doll. Caching occurs in both places.

Russian doll caching works well when the outer cache is automatically busted
by Rails when a record is updated. For instance, if a wishlist's title is
changed, the outer wishlist cache is rerendered on the next request. However,
each inner item cache remains intact; therefore, the cached content for each
item can be used as the wishlist rerenders, speeding up the process.

As shown, however, russian doll caching breaks down a bit when the markup in
the *item* partial (the inner "doll") is changed. In that case, a developer
must manually walk up the graph of dolls, incrementing `"v1"` along the way.

In this example, changing the markup in the `_item.html.erb` partial requires
bumping `"v1"` to `"v2"` in that file, but *also in `wishlists/show.html.erb`*
(the view that renders the entire wishlist). Otherwise, the outer wishlist
cache will continue to display the old item markup.

The [**cache_digests**](https://github.com/rails/cache_digests) plugin that
is included with Rails 4 solves the problem.

In Rails 4, the wishlist and item views can be written as:

@@@ erb
<%# app/views/wishlists/show.html.erb %>
<%- cache @wishlist do %>
  <h1 class="wishlist-title"><%= @wishlist.title %></h1>

  <ul>
    <%= render @wishlist.items %>
  </ul>
<%- end %>
@@@

<p></p>

@@@ erb
<%# app/views/wishlists/_item.html.erb %>
<%- cache @item do %>
  <li><%= @item.name %> (<%= @item.price %>)</li>
<%- end %>
@@@

With **cache_digests**, any time the view where the `cache` call resides is
modified, the cache is automatically busted, *including caches in dependent
views*. There is no longer a need for `"v1"`.

**cache_digests** solves the problem we had with wishlists and items: changing
the markup in `_item.html.erb` automatically busts the cache there, as well as
in `wishlists/show.html.erb`.

**cache_digests** operates by embedding an MD5 hash of the template content in
the cache's key, including template content for dependent views. When the
template content changes, the hash value changes. Read more about
**cache_digests** via [its README on
GitHub](https://github.com/rails/cache_digests).

Like certain other Rails 4 features, **cache_digests** can be included as a gem
in Rails 3.2, so it can be used today by simply adding the gem to `Gemfile`. I
recommend using **cache_digests** in existing applications if you already use
this style of caching or want to use it before Rails 4 is released.

### <a id="encrypted-cookies"></a>Encrypted Cookies

Rails 4 introduces encrypted cookies, and uses them by default (in newly
generated apps) as the store for session data. Encrypted cookies save data in a
form that cannot be easily tampered with by users; furthermore, users cannot
even read the data that is being saved at all.

In contrast, Rails 3 uses digitally signed cookies as the default store for
sessions. Digitally signed cookies cannot be easily tampered with, but users
*can* read the data that is being saved.

When upgrading an application, consider transitioning to the encrypted cookie
store to garner an increase in security.

Rails 4 offers a cookie store that acts as a hybrid between the signed cookie
store (from Rails 3) and the encrypted cookie store. Appropriately enough, it
is called the `UpgradeSignatureToEncryptionCookieStore`. Existing digitally
signed sessions will still be valid, and will be upgraded to be encrypted
automatically. New sessions will simply be encrypted right away.

To use the `UpgradeSignatureToEncryptionCookieStore`, edit
`config/initializers/session_store.rb` and change `:cookie_store` to
`:upgrade_signature_to_encryption_cookie_store`. Leave the value of `:key` as
it is:

@@@ ruby
# config/initializers/session_store.rb
Rails4app::Application.config.session_store :upgrade_signature_to_encryption_cookie_store,
                                            :key => '_widgets_session'
@@@

The encrypted cookie store requires an additional secret key. Generate one
using the `rake secret` task from the command line:

@@@ text
$ rake secret
35137db8a7a7ac525c...
@@@

Copy the long string of numbers and characters that is printed to your
clipboard. Next, open `config/initializers/secret_token.rb`. Duplicate (but do
not remove) the existing line that reads like
`Widgets::Application.config.secret_token = '...'` (where `'...'` is an
existing long string of numbers and characters). On the duplicated line,
replace `secret_token` with `secret_key_base` and replace `'...'` (the existing
secret token) with the string you just generated with `rake secret`.

After duplicating and editing the line, you should have something that looks
like:

@@@ ruby
# config/initializers/secret_key.rb

# Existing line and secret token
Widgets::Application.config.secret_token = 'f547fd1e154c2a9a682a...'

# Newly added secret key for encrypted cookies
Widgets::Application.config.secret_key_base = '35137db8a7a7ac525c...'
@@@

With the changes to `config/initializers/session_store.rb` and
`config/initializers/secret_token.rb`, the application has been updated to use
the new encrypted session store!

<!-- TODO: Talk about what things look like in new apps? -->

### <a id="etagger"></a>Declarative ETags

Client-side caching is one of the best ways to save sending bytes over
the wire, thereby reducing bandwidth costs and load times.

An entity tag (ETag) is one mechanism defined in the HTTP specification that
supports caching. When a browser first requests a resource, the server may send
back an ETag. An ETag is an opaque hash representing a version of the resource.
When a browser requests the same resource in the future, it sends along the
ETag it remembered the last time the resource was requested. If the ETag
matches the current version of the resource (i.e., the resource has not changed
since the last time it was requested), the server can send a "304 Not Modified"
response with an empty body, saving bandwidth and server processing time.

Conversely, if the resource has changed, the ETags will not match and the
server knows it must send a full response.

An ETag can be generated easily from an ActiveRecord model. By default, the
model's class, `id` (primary key), and `updated_at` are combined as the basis
for the ETag hash. With `updated_at` in the mix, this `cache_key` returns a new
value every time the record is updated.

Rails has provided facilities to generate ETags since Rails 3. Consider a
controller that takes advantage of ETags:

@@@ ruby
# app/controllers/widgets_controller.rb
class WidgetsController < ApplicationController
  def show
    @widget = Widget.find(params[:id])
    fresh_when(@widget)
  end
end
@@@

The
[fresh_when](http://api.rubyonrails.org/classes/ActionController/ConditionalGet.html#method-i-fresh_when)
method sets up the response to include an ETag. Furthermore, if the ETag sent
by the browser matches the current ETag, the controller will not render the
view, opting instead for a "304 Not Modified" empty response.

Sometimes it makes sense to use additional information when generating the
ETag, for instance, to scope the ETag to the currently logged in user.
Otherwise, cached content may be reused for a different user (an information
leak and security vulnerability!).

In Rails 3, this scoping can be achieved by passing an array of values to
`fresh_when`:

@@@ ruby
# app/controllers/widgets_controller.rb
class WidgetsController < ApplicationController
  def show
    @widget = Widget.find(params[:id])
    fresh_when([@widget, current_user.id])
  end
end
@@@

In this case, the `current_user`'s ID is used when generating the ETag. As a
result, different ETags will be generated for every user, even for the same
version of `@widget`. Information leak patched!

However, it can be a pain to remember to add this scoping in every controller
action. Rails 4 introduces a new declarative syntax for these controller-wide
ETag concerns:

@@@ ruby
# app/controllers/widgets_controller.rb
class WidgetsController < ApplicationController
  etag { current_user.id }

  def show
    @widget = Widget.find(params[:id])
    fresh_when(@widget)
  end
end
@@@

The code above has the same behavior as the previous example where
`current_user.id` was passed explicitly to `fresh_when`. However, it is more
readable and DRY (Don't Repeat Yourself: an adjective for code or knowledge
that is not duplicated).

Furthermore, you can call `etag` multiple times to continue adding values used
when generating ETags for the controller.

#### Use in Rails 3.2

While declarative ETags will be baked into Rails 4 itself, they can also be
used in Rails 3.2 applications by pulling in the `etagger` gem:

@@@ ruby
# Gemfile
gem 'etagger'
@@@
