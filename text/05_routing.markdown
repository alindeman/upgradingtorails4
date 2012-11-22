## Routing

The Rails router is one of the first pieces of code to run when an application
receives an HTTP request. Routes determine which controller will handle the
request.

Routes are specified in `config/routes.rb`.

### match

Rails 4 removes the `match` directive. `match` was often used incorrectly,
and to dangerous consequences.

Consider this route that submits an order to purchase a widget:

@@@ ruby
# config/routes.rb
Widgets::Application.routes.draw do
  match "/widgets/:id/purchase" => "widgets#purchase"
end
@@@

If this is the route that actually submits an order and charges a user's
credit card, the application is vulnerable to the cross-site request
forgery (CSRF) attack.

Because `match` without any other qualifiers routes a request with any HTTP
verb, including GET, an attacker could lure a user to click a link on a
different site that fires a request to purchase a widget:

@@@ html
<a href="http://example.com/widgets/1234/purchase">
  Click here! It's awesome! Seriously!
</a>
@@@

A naive user who clicks the link will have immediately purchased the widget
with ID 1234.

Rails has included protection against CSRF attacks for quite some time, but not
for requests that use the GET verb. Requests that come in via GET are supposed
to be *safe*, meaning they should only be used for retriveval of information,
not causing any appreciable server-side state changes. In this case, the
application violates that constraint by allowing a GET request to submit a
purchase order.

The solution is to update the routes to use a more appropriate HTTP verb, such
as POST:

@@@ ruby
# config/routes.rb
Widgets::Application.routes.draw do
  post "/widgets/:id/purchase" => "widgets#purchase"
end
@@@

In other cases, an application might have been using `match` for *safe*
requests. In that case, the directive can simply be updated to `get` to be
compatible with Rails 4:

@@@ ruby
Widgets::Application.routes.draw do
  get "/widgets" => "widgets#index"
end
@@@

You will need to change any instance of `match` in `config/routes.rb` before
an application will even boot in Rails 4. If you forget, you will be confronted
with an error:

@@@ text
You should not use the `match` method in your router without specifying an HTTP
method. (RuntimeError)
@@@

If you truly need to route a request that comes in via any HTTP verb, modify
the `match` route to specify `:via => :any`

@@@ ruby
Widgets::Application.routes.draw do
  match "/something" => "something#index", :via => :any
end
@@@
