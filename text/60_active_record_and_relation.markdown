## ActiveRecord

Rails 4 did not just deprecate features of ActiveRecord, it also added some
compelling new ones.

### Relation#none

It was previously not possible to reliably return a scope that would select no
records but still allow other scopes to be chained onto it. However, Rails 4
introduces the `none` scope to faciliate this.

Consider an authorization scheme where an unapproved user cannot view any
posts:

@@@ ruby
class Post < ActiveRecord::Base
  def self.authorized(user)
    if user.unapproved?
      none
    else
      all
    end
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

The `none` scope is returned by `Post.authorized` if the user is unapproved.
The controller can tack on more scopes (e.g., `limit(10)`) even though the
query has been stunted.

The `none` scope is implemented by returning an `ActiveRecord::NullRelation`.
No database query will be used when a `none` scope is chained on.
