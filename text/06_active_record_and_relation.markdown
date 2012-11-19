## <a id="activerecord"></a>ActiveRecord

Querying the database changed dramactially from Rails 2 and Rails 3.
Thankfully Rails 4 does not change things up nearly as much, but there are some
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

### Dynamic Finders

Many of the dynamic finders have been deprecated in favor of alternate syntax.
Most of the alternate syntax is already supported in Rails 3.2.

<table>
  <tr>
    <th>Deprecated Syntax</th>
    <th>Preferred Syntax</th>
  </tr>
  <tr>
    <td>
      <code>find_all_by_...</code><br/>
      <code>scoped_by_...</code>
    </td>
    <td>
      <code>where(...)</code>
    </th>
  </tr>
  <tr>
    <td>
      <code>find_last_by_...</code>
    </td>
    <td>
      <code>where(...).last</code>
    </td>
  </tr>
  <tr>
    <td>
      <code>first_or_initialize_by_...</code>
    </td>
    <td>
      <code>first_or_initialize_by(...)</code>
    </td>
  </tr>
  <tr>
    <td>
      <code>find_or_create_by_...</code><br/>
      <code>first_or_create_by_...!</code>
    </td>
    <td>
      <code>find_or_create_by(...)</code><br/>
      <code>find_or_create_by!(...)</code>
    </td>
  </tr>
</table>

An example of each and the newly preferred alternative:

@@@ ruby
# User.find_all_by_last_name("Lindeman")
User.where(last_name: "Lindeman")

# User.find_last_by_email("andy@andylindeman.com")
User.where(email: "andy@andylindeman.com").last

# User.first_or_initialize_by_github_id(395621)
User.first_or_initialize_by(github_id: 395621)

# User.find_or_create_by_twitter_handle("alindeman")
User.first_or_create_by(twitter_handle: "alindeman")
@@@

Finally, the `find_by_...` dynamic finder is *not* deprecated. Code such as
`User.find_by_email("andy@andylindeman.com")` will continue functioning without
deprecation warnings.

### Eager-Evaluated Scopes

Creating a scope without a callable object is deprecated in Rails 4:

@@@ ruby
class Comment < ActiveRecord::Base
  scope :visible, where(visible: true)
end
# DEPRECATION WARNING: Using #scope without passing a callable object is
# deprecated.
@@@

The preferred syntax in Rails 4 is to create scopes by passing a proc or lambda
(though any object that responds to `call` will do).

@@@ ruby
class Comment < ActiveRecord::Base
  # Ruby 1.9 lambda syntax
  scope :visible, -> { where(visible: true) }

  # Also acceptable
  scope :visible, proc { where(visible: true) }
  scope :visible, lambda { where(visible: true) }
end
@@@

### Relation#all

Calling `all` on a relation in Rails 4 will return a new relation instead of
an `Array`.

Consider this code snippet and the return values in Rails 3:

@@@ ruby
Post.where("created_at > ?", 2.days.ago).all
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

In most cases, the change should not negatively affect your code. Both
`ActiveRecord::Relation` and `Array` act as  `Enumerable`; furthermore, if
there is a method that `ActiveRecord::Relation` does not respond to but `Array`
does, the method will be proxied through to a version of the results that has
been loaded into an `Array`.

However, if you see errors caused by code that previously expected an `Array`
and is not handling the change to `ActiveRecord::Relation` properly, you can
force the scope to execute and return an `Array` with `to_a`:

@@@ ruby
Post.where("created_at > ?", 2.days.ago).to_a.class
# Array
@@@

This is also a change you can start making today, as `ActiveRecord::Relation`
supports `to_a` in Rails 3.

### Relation#includes

The `includes` scope is most often used to eager load associated records, to
avoid the [N+1 query
problem](http://guides.rubyonrails.org/active_record_querying.html#eager-loading-associations):

@@@ ruby
# OUCH: Executes a query to find all posts, then
# N queries to find each posts' comments!
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
Rails never *guaranteed* that it would use an `OUTER JOIN`, many developers
have written code that takes advantage of this implementation detail.

Consider a query that selects a post and its visible comments:

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

In Rails 4, you must explicitly tell Rails that the query `references` a joined
table:

@@@ ruby
Post.includes(:comments).references(:comments).where("comments.visible = ?", true)
@@@

Alternatively you can use a hash of conditions, which does not require the
`references`:

@@@ ruby
Post.includes(:comments).where(comments: { visible: true })
@@@

This deprecation will only bite you if you pair `includes` with a `where`
condition on the joined table. If you use `includes` only to eager load
associations, this deprecation will not affect your code.

### Relation#order

<!-- TODO: `order` prepends rather than appends -->
<!-- http://edgeguides.rubyonrails.org/active_record_querying.html#ordering -->

### <a id="whiny-nils"></a>whiny nils

Rails 4 removed *whiny nils*, a feature that would raise a warning when code
sent the `id` message to `nil`. Usually this cropped up in applications when
code like `@model.id` was run and `@model` was not yet initialized
(uninitialized instance variables in Ruby evaluate to `nil`).

Because before Ruby 1.9.3, any `Object` would respond to the `id` method, and
`nil` is an `Object`. Especially confusing was the fact that `nil.id` returned
`4` due to implementation details of Ruby. Ask a developer who has been using
Rails for many years about `4` sometime.

Thankfully `Object` instances in Ruby 1.9.3 no longer respond to `id`.
Therefore, whiny nils are no longer needed. Attempting to run `nil.id` will
simply raise a `NoMethodError`, instead of a confusing warning.

Rails will raise a deprecation warning if an application's configuration
attempts to enable whiny nils. To squash the warning, remove any lines that
refer to `config.whiny_nils` in `config/environments/development.rb` and
`config/environments/test.rb`.

### ActiveRecord Session Store

Storing session data in the database using the ActiveRecord session store has
been extracted into a gem.

Applications often used the ActiveRecord session store when the data being
added to the sessions was sensitive (e.g., users should not be able to decode
the contents of the session) or unusually large (over 4KB, the maximum size
of a cookie).

To continue using the ActiveRecord session store, bring in the
`activerecord-session_store` gem:

@@@ ruby
# Gemfile
gem 'activerecord-session_store',
  github: 'rails/activerecord-session_store'
@@@

Rails 4 introduces [encrypted cookies](#encrypted-cookies) which may be a good
alternative in certain use cases where the ActiveRecord session store was
the only option before.
