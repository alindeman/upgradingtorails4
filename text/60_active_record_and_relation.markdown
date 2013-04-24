## ActiveRecord

Rails 4 did not just deprecate features of ActiveRecord; it also added some
compelling new ones.

### Relation#none

Consider an authorization scheme where only approved users can view posts.

Before Rails 4, writing an `authorized` method on `Post` was difficult because
there is no reliable way to create a scope that will never return anything. An
option that was often used was to return an empty array:

@@@ ruby
class Post < ActiveRecord::Base
  def self.authorized(user)
    user.approved? ? all : []
  end
end
@@@

However, this is problematic because any code that uses the `authorized` method
must know that it could potentially return an array, and in that case, no
more scopes can be chained. Consider a controller that uses the method:

@@@ ruby
class PostsController < ApplicationController
  def index
    @posts = Post.authorized(current_user)
    unless @posts.is_a?(Array)
      @posts = @posts.limit(10)
    end
  end
end
@@@

This is a contrived example: the astute reader will recognize that the `unless`
conditional can be removed if `authorized` is the last method in the chain.
However, a more complicated chain of methods and scopes might not be able to be
simplified in this way.

In any case, the code is too surprising: the `authorized` method has two very
different return values: one where more scopes *can* be added, and the other
(an `Array`) where more scopes *cannot* be added.

All is not lost! Rails 4 introduces the `none` scope. The `authorized` methods
can now be written as:

@@@ ruby
class Post < ActiveRecord::Base
  def self.authorized(user)
    user.approved? ? all : none
  end
end
@@@

A controller can now query for posts that the current user is authorized to
see, but not worry that it may be "none":

@@@ ruby
class PostsController < ApplicationController
  def index
    @posts = Post.authorized(current_user).limit(10)
  end
end
@@@

The `none` scope is returned by `Post.authorized` if the user is not yet
approved. In that case the query will never return any posts, but the
controller can tack on more scopes (i.e., `limit(10)`) even so.

The `none` scope is implemented by returning an `ActiveRecord::NullRelation`,
named for the [null object
pattern](http://en.wikipedia.org/wiki/Null_Object_pattern). No database query
will be used when a `none` scope is used in the chain.

### <a id="relation-not"></a>Relation#not

Imagine an application has a `Comment` model, and you wish to find all of the
comments authored by users *other* than the current user.

In Rails 3, writing a query that needs the not-equal-to operator (`!=`)
requires using `where` with a string condition:

@@@ ruby
Comment.where("user_id != ?", current_user.id)
@@@

Rails 4 introduces the `not` scope. In Rails 4, the same query could be
rewritten as:

@@@ ruby
Comment.where.not(user_id: current_user.id)
@@@

Notably, no string clause is required. The result is a cleaner expression that
is guaranteed to operate correctly regardless of the underlying database.

### <a id="update"></a>ActiveRecord::Base#update

Rails 4 introduces `ActiveRecord::Base#update`, allowing you to use `update`
instead of `update_attributes` when updating the attributes of an ActiveRecord
model.

`update` is more terse and matches the name of the controller action where it
is normally used, a property that `new`, `create`, and `destroy` have shared
for a while now.

@@@ ruby
comment = Comment.find(1)

# Rails 3
comment.update_attributes(body: "Updated content")

# Rails 4 (same effect)
comment.update(body: "Updated content")
@@@

`update` and `update_attributes` can be used interchangeably in Rails 4. In
fact, `update_attributes` is only "soft deprecated": unlike other deprecations,
it will not raise any visible warnings. While `update_attributes` may be
removed in future versions of Rails, it will probably not be anytime soon.
