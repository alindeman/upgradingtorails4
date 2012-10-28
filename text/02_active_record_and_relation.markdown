## ActiveRecord & ActiveRecord::Relation

Querying the database looked drastically different between Rails 2 and Rails 3.  Thankfully Rails 4 does not change things up nearly as much but there are some improvements and gotchas you need to be aware of.

### Pay attention if ...

* You see deprecation warning when making database queries
* TODO

---

### Rails 2 Finder Syntax

Speaking of Rails 2, Rails 4 deprecates the older style of finding records. If you have an application that was upgraded to Rails 3 without using the new chained syntax, you will receive lots of deprecation warnings in Rails 4:

@@@ ruby
Post.find(:all, conditions: ["created_at > ?", 2.days.ago])
# DEPRECATION WARNING: Calling #find(:all) is deprecated. Please call #all directly instead.
# You have also used finder options. These are also deprecated.
# Please build a scope instead of using finder options.
@@@

Rails 4 does not remove the Rails 2 finders, but a future version of Rails might. To be ready, you can squash the deprecation warnings by switching to the chained scope syntax introduced in Rails 3:

@@@ ruby
Post.where("created_at > ?", 2.days.ago)
@@@

### Relation#all

In Rails 3, calling `all` at the end of a chain of scopes forced the database query to execute and the results to be returned as an array:

@@@ ruby
Post.where("created_at > ?, 2.days.ago).all
# [Post, Post]

Post.where("created_at > ?", 2.days.ago).all.class
# Array
@@@

In Rails 4, **`all` returns a new Relation**. This new relation simply represents "all" of the records and can be further chained onto.

@@@ ruby
Post.all.where("created_at > ?", 2.days.ago)
# [Post, Post]

Post.all.where("created_at > ?", 2.days.ago).class
# ActiveRecord::Relation
@@@

If you see errors caused by code that previously expected an `Array` and is not handling the change to `ActiveRecord::Relation` properly, you can change the call from `all` to `to_a`:

@@@ ruby
Post.where("created_at > ?", 2.days.ago).to_a.class
# Array
@@@

If you've been thinking that `all` in Rails 4 sounds like `scoped` in Rails 3, you're right! And in fact, `Relation#scope` is deprecated in favor of `all`.

@@@ ruby
Post.scoped # Rails 3
Post.all    # Rails 4
@@@
