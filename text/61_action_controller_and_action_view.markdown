## ActionController & ActionView

### <a id="action-controller-live"></a>ActionController::Live

<!-- LIVE -->

### <a id="cache-digests"></a>Cache Digests

Nesting fragment caches--often called *russian doll caching*--is an effective
way to achieve a speed up while keeping view code relatively simply.

Consider an application that manages shopping wishlists. A customer creates
a wishlist with a title and adds items to it.

The application contains a view partial that renders the entire wishlist
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
any time the markup is changed. If the `"v1"` were not present or not
incremented, the application could display a mishmash of cached content with
the older markup alongside updated content with newer markup.

For instance, a designer adding a CSS class to the wishlist title header
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

The partial that renders each item looks similar to the wishlist view:

@@@ erb
<%# app/views/wishlists/_item.html.erb %>
<%- cache ["v1", @item] do %>
  <li><%= @item.name %> (<%= @item.price %>)</li>
<%- end %>
@@@

This strategy is called russian doll caching because of its use of nested
templates, each of which is cached. The wishlist is an outer "doll", and each
item is an inner "doll." Caching occurs in both places.

Russian doll caching works well when the outer cache is automatically busted
by Rails when a record is updated. For instance, if a wishlist's title is
changed, the outer wishlist cache is rerendered on the next request. However,
each inner item cache remains intact; therefore, the cached content for each
item can be used as the wishlist rerenders, speeding up the process.

As shown, however, russian doll caching breaks down when the markup in the
*item* partial (the inner "doll") is changed. In that case, a developer must
manually walk up the graph of dolls, bumping the `"v1"` along the way.

In this example, changing the markup in the `_item.html.erb` partial requires
bumping `"v1"` to `"v2"` in that file, but *also in `wishlists/show.html.erb`*
(the view that renders the entire wishlist). Otherwise, the outer wishlist
cache will continue to display the old item markup.

The [**cache_digests**](https://github.com/rails/cache_digests) gem solves the
problem.

<!-- TODO: Make sure this is definitely true -->
Unlike some other Rails 4 features, **cache_digests** must be added to
`Gemfile`:

@@@ ruby
# Gemfile
gem 'cache_digests'
@@@

In the presence of **cache_digests**, the wishlist and item views can be
written as:

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

Notably, there is no `"v1"`! **cache_digests** has added a much more robust
version automatically. Any time the view where the `cache` call resides is
updated, the cache is busted, *including caches in dependent views*.

This solves the problem we had with wishlists and items. With **cache_digests**
installed, changing the markup in `_item.html.erb` automatically busts
the cache there, as well as in `wishlists/show.html.erb`.

**cache_digests** operates by embedding an MD5 hash of the template content in
the cache's key, including template content for dependent views. When the
template content changes, the hash value changes. Read more about
**cache_digests** via [its README on
GitHub](https://github.com/rails/cache_digests).

Like certain other Rails 4 features, **cache_digests** only requires Rails 3.2
so it can be used today by simply adding the gem to `Gemfile`. I recommend
using **cache_digests** in existing applications if you already use this style
of caching or want to use it before Rails 4 is released.

### <a id="encrypted-cookies"></a>Encrypted Cookies

<!-- https://github.com/rails/rails/pull/8112 -->

### <a id="etagger"></a>Declarative ETags
