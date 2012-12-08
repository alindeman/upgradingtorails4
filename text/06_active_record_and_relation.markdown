## <a id="activerecord"></a>ActiveRecord

Querying the database changed dramatically from Rails 2 to Rails 3.
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

Notably, though, the `find_by_...` dynamic finder is *not* deprecated. Code
such as `User.find_by_email("andy@andylindeman.com")` will continue functioning
without deprecation warnings.

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

Rails 4 changed the way `order` operates when there are multiple calls to
`order` in a chain. You might say that, well, Rails 4 changed the order
of `order`.

Consider this query that attempts to show comments with many replies first.
If comments have the same number of replies, the tie is broken by the creation
time of the comment, earlier comments first:

@@@ ruby
Comment.order("replies_count DESC").order(:created_at)
@@@

This query works correctly in Rails 3. The query generated will be something
like:

@@@ sql
-- Rails 3
SELECT * FROM comments ORDER BY replies_count DESC, created_at
@@@

In Rails 4, however, the order specifications are flipped: the sort order
is `created_at` first and `replies_count` second:

@@@ sql
-- Rails4
SELECT * FROM comments ORDER BY created_at, replies_count DESC
@@@

In this case, the query is broken in Rails 4.

One obvious fix is to simply switch the sequence of the `order` calls:

@@@ ruby
# Rails 4
Comment.order(:created_at).order("replies_count DESC")
@@@

However, a more readable way uses `order` with multiple arguments. This call
works correctly in both Rails 3 and Rails 4:

@@@
# Works in both Rails 3 and Rails 4
Comment.order("replies_count DESC", :created_at)
@@@

### <a id="whiny-nils"></a>whiny nils

Rails 4 removed *whiny nils*, a feature that would raise a warning when code
sent the `id` message to `nil`. Usually this cropped up in applications when
code like `@model.id` was run and `@model` was not yet initialized
(uninitialized instance variables in Ruby evaluate to `nil`).

Before Ruby 1.9.3, any `Object` would respond to the `id` method, and `nil` is
an `Object`. Especially confusing was the fact that `nil.id` returned `4` due
to implementation details of Ruby. Ask a developer who has been using Rails for
many years about `4` sometime.

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

Rails 4 introduces encrypted cookies which may be a good alternative in certain
use cases where the ActiveRecord session store was the only option before. I
discuss more about encrypted cookies when I talk about the [new features in
ActionController](#encrypted-cookies).

### <a id="observers"></a>Observers

Observers have been extracted into a gem. Observers watch ActiveRecord models
and are invoked in the same way that callbacks (e.g., `after_save`) are invoked
on the model itself.

The Rails guides once said about observers: "Whereas callbacks can pollute a
model with code that isn't directly related to its purpose, observers allow you
to add the same functionality without changing the code of the model."

In many applications, however, observers cause more problems than they solve.
It is often mentally taxing to remember that code in observers ("physically"
far away from model code) is run when creating, updating, saving or deleting a
record. [Gems like no-peeping-toms have been
written](https://github.com/patmaddox/no-peeping-toms) to disable observers in
tests because they can slow the suite down or attempt to interface with
external systems.

By extracting observers, Rails 4 is not-so-subtly discouraging their use in
new applications.

To continue using observers in upgraded applications, though, simply bring in
the `rails-observers` gem:

@@@ ruby
# Gemfile
gem 'rails-observers',
  github: 'rails/rails-observers'
@@@

### <a id="json-serialization"></a>JSON Serialization

In Rails 3, HTML entities are output as-is when a model is serialized to JSON.
For example, `Post.find(2).to_json` might return a structure like:

@@@ json
{
  "id": 2,
  "title": "Hello World",
  "body": "<a href='hello.html'>Hello!</a>"
}
@@@

However, because the `body` attribute can contain HTML, the application is
vulnerable to attacks such as cross-site scripting (XSS) in certain
circumstances. If the JSON were embedded directly into a page, for example, an
attacker could embed malicious HTML and JavaScript that could steal the user's
session or redirect them to another site.

Rails 4 enables the `config.active_support.escape_html_entities_in_json`
option by default. The option name is actually a bit confusing: HTML entities
are not really *escaped*, but simply encoded differently so they do not
evaluate to HTML tags when embedded directly in an HTML page.

In Rails 4, the expression `Post.find(2).to_json` will (by default) return a
structure like:

@@@ json
{
  "id": 2,
  "title": "Hello World",
  "body": "\u003Ca href='hello.html'\u003EHello!\u003C/a\u003E"
}
@@@

The angle brackets in `body` have been encoded such that a browser would not
interpret them as HTML tags. However, JSON parsers will decode both the Rails 3
and the Rails 4 versions exactly the same. For this reason, it is unlikely that
this change will cause you any pain when upgrading. Even so, it is important to
understand why the change was made, and why you might see slightly different
JSON serialization output between Rails 3 and Rails 4.
