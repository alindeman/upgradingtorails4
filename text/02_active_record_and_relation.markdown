## ActiveRecord & ActiveRecord::Relation

Querying the database looked drastically different between Rails 2 and Rails 3.
Thankfully Rails 4 does not change things up nearly as much but there are some
improvements and gotchas you need to be aware of.

---

### Rails 2 Finder Syntax

Rails 2 finder syntax is deprecated. If you have an application that uses
`find(:all)` and `find(:first)`, you'll need to transition it to the new
chained syntax.

@@@ ruby
Post.find(:all, conditions: ["created_at > ?", 2.days.ago])
# DEPRECATION WARNING: Calling #find(:all) is deprecated. Please call #all directly instead.
# You have also used finder options. These are also deprecated.
# Please build a scope instead of using finder options.
@@@

Rails 4 does not remove the Rails 2 finders, but a future version of Rails
might. To be ready, you can squash the deprecation warnings by switching to the
chained scope syntax introduced in Rails 3:

@@@ ruby
Post.where("created_at > ?", 2.days.ago)
@@@

### Relation#all

Calling `all` on a relation in Rails 4 will return a new relation instead of
an `Array`.

Consider this code snippet and the return values in Rails 3:

@@@ ruby
Post.where("created_at > ?, 2.days.ago).all
# [Post, Post]

Post.where("created_at > ?", 2.days.ago).all.class
# Array
@@@

In Rails 4, however, `all` does not force the query to an `Array`. It has
similar behavior to `scoped` in Rails 3:

@@@ ruby
Post.all.where("created_at > ?", 2.days.ago)
# [Post, Post]

Post.all.where("created_at > ?", 2.days.ago).class
# ActiveRecord::Relation
@@@

This change should mostly be transparent. Both `ActiveRecord::Relation` and
`Array` are `Enumerable`; furthermore, if there is a method that
`ActiveRecord::Relation` does not respond to, but `Array` does`, the method
will be proxied through to a loaded version of the query results transparently.

However, if you see errors caused by code that previously expected an `Array`
and is not handling the change to `ActiveRecord::Relation` properly, you can
force the scope to execute and return an `Array` with `to_a`:

@@@ ruby
Post.where("created_at > ?", 2.days.ago).to_a.class
# Array
@@@

This is also a change you can start making today, as `ActiveRecord::Relation`
supports `to_a` in Rails 3.

### Relation#none

It was previously not possible to reliably return a scope that would select no
records, but still allow other scopes to be chained onto it. However, Rails 4
introduces the `none` scope to faciliate this!

For example, consider an authorization scheme where an unapproved user cannot
view any posts:

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
The controller can still tack on more scopes (e.g., `limit(10)`) even though
the query has been stunted.

The `none` scope is implemented by returning an `ActiveRecord::NullRelation`.
No database query will be used when a `none` scope is chained on.

### Relation#includes

The `includes` scope is most often used to eager-load associated records, to
avoid the [N+1 query
problem](http://guides.rubyonrails.org/active_record_querying.html#eager-loading-associations):

@@@ ruby
# OUCH: Executes a query to find all posts, then N queries to find each posts'
# comments!
Post.find_each do |post|
  post.comments.each do |comment|
    # ...
  end
end
@@@

A solution that `includes` comments allows Rails to run 1 or 2 queries at most:
@@@ ruby
Post.includes(:comments).find_each do |post|
  post.comments.each do |comment|
    # ...
  end
end
@@@

Rails usually uses an `OUTER JOIN` to perform the eager loading. However, while
it was never *guaranteed* to perform an `OUTER JOIN`, many developers have
written code that takes advantage of this hidden implementation detail.

For instance, a query that selects a post and its *visible* comments only.

@@@ ruby
Post.includes(:comments).where("comments.visible = ?", true).find_each do |post|
  # ...
end
@@@

**This will cause a deprecation warning in Rails 4.**

@@@ ruby
Post.includes(:comments).where("comments.visible = ?", true)
# DEPRECATION WARNING: It looks like you are eager loading table(s) (one of:
# posts, comments) that are referenced in a string SQL snippet.
@@@

Rails was required to parse the string in the `where` clause to figure out that
the `comments` table was referenced.

There are a few ways to fix it; first, you can explicitly tell Rails that the
query `references` a joined table:

@@@ ruby
Post.includes(:comments).references(:comments).where("comments.visible = ?", true)
@@@

Or you can use a hash of conditions, which does not require the `references`:

@@@ ruby
Post.includes(:comments).where(comments: { visible: true })
@@@

This deprecation will only bite you if you pair `includes` with a `where`
condition on the joined table. If you only use `includes` only to eager load
associations, this will not affect your code.
If you only use `includes` for eager loading associations, you should have no
problems upgrading.
