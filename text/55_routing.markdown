## Routing

### <a id="patch-verb"></a>PATCH Verb

Rails 4 introduces support for the HTTP PATCH verb. According to [RFC
5789](http://tools.ietf.org/html/rfc5789), the PATCH verb is appropriate
for applying "partial modifications to a resource."

Traditionally, Rails has used the PUT verb for updates of RESTful resources.
However, the RFC states that PUT should be used only to "overwrite a resource
with a complete new body," and *not* to simply update parts of it.

So, the Rails conventions through Rails 3 have encouraged an incorrect use of
PUT. For partial updates, PATCH is the preferred verb.

Rails 4 retains support for update requests coming in via the PUT verb, yet
adds support for those same types of requests coming in via the PATCH verb too.

Consider a typical RESTful controller with an `update` action:

@@@ ruby
# app/controllers/widgets_controller.rb
class WidgetsController < ApplicationController
  # ...

  def update
    @widget = Widget.find(params[:id])
    @widget.update_attributes(params[:widget])

    redirect_to @widget
  end
end
@@@

In Rails 4, requests for `PUT /widgets/1` and `PATCH /widgets/1` will both
route to the `WidgetsController#update` action.

You only need to be aware of the PATCH verb; the addition should not cause any
upgrade pains as PUT requests continue operating as they always have.

That said, it is recommended that you consider transitioning to using PATCH
instead of PUT when your Rails application is an HTTP/RESTful API.

### <a id="routing-concerns"></a>Routing Concerns

Rails 4 introduces **routing concerns**: additions to the routing
domain-specific language (DSL) that promise to reduce duplication in certain
situations.

Imagine an application that manages many different resources, all of which
can be commented on by users. In Rails 3, a route file might look like:

@@@ ruby
# config/routes.rb
Animals::Application.routes.draw do
  resources :dogs do
    resources :comments
  end

  resources :cats do
    resources :comments
  end

  resources :ferrets do
    resources :comments
  end

  # ... etc ...
end
@@@

Dogs, cats and ferrets have nested comment resources. In Rails 4, this comment
"concern" can be extracted and reused:

@@@ ruby
# config/routes.rb
Animals::Application.routes.draw do
  concern :commentable do
    resources :comments
  end

  resources :dogs,    concerns: [:commentable]
  resources :cats,    concerns: [:commentable]
  resources :ferrets, concerns: [:commentable]
  # ... etc ...
end
@@@

<!-- TODO: Talk about #call-able concerns -->
