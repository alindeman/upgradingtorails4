## <a id="activesupport"></a>ActiveSupport

ActiveSupport adds many additional methods on core Ruby objects, and introduces
a few new objects of its own. For instance, `"octopus".pluralize` returning
`"octopi"` comes from ActiveSupport (specifically, the inflector). The "magic"
`params` hash in Rails controllers that allows access to its values via either
string *or* symbol keys is `HashWithIndifferentAccess` from ActiveSupport.

---

### Object#try

ActiveSupport adds the `try` method to all `Object` instances. `try` sends
a message to an object only if it is not `nil`.

For example, `try` could be used in a view to print the name of a customer's
spouse, only if the customer *has* a spouse:

@@@ erb
<%= @customer.spouse.try(:name) %>
@@@

If `@customer.spouse` is present (not `nil`), the view will include his or her
name. On the other hand, if `customer.spouse` *is* `nil`, the expression itself
will return `nil`, and the view simply will not output anything.

Both Rails 3 and Rails 4 react the same way when `try` is called on `nil`:

@@@ ruby
# Both Rails 3 and Rails 4 return `nil`
nil.try(:name)
@@@

However, Rails 4 changes the behavior when the object receiver is not `nil`
yet does not respond to the message being sent:

@@@ ruby
# Rails 4
"abc".try(:bogus_method) # => nil
@@@

In Rails 3, the same expression will raise a `NoMethodError`:

@@@ ruby
# Rails 3
"abc".try(:bogus_method) # => NoMethodError
@@@

If you have used `Object#try` and want to keep the Rails 3 behavior when
upgrading to Rails 4, switch to the newly-introduced `Object#try!`.

@@@ ruby
# Rails 4
"abc".try!(:bogus_method) # => NoMethodError
@@@

Interestingly, this same change was made to `Object#try` in Rails 3.1.0 but
was reverted in Rails 3.1.1. It seems like it might finally be sticking around
for good in Rails 4.

Finally, `Object#try` can no longer be used to invoke private methods in Rails
4. Attempting to call a private method using `try` will simply return `nil`
and the method will not be invoked.

## <a id="caching-with-memcached"></a>Caching with memcached

Caching data in memcached is a popular way to speed up Rails applications.
Rails 4 requires the `dalli` gem, whereas Rails 3 used the now-outdated
`memcache-client` gem.

If your application caches data in memcached, you may receive an error tracing
to a line in `config/application.rb` or `config/environments/production.rb`:

@@@ ruby
# config/environments/production.rb
Widgets::Application.configure do
  # ...

  config.cache_store = :mem_cache_store, "cache1.example.com"
end
# You don't have dalli installed in your application. Please add it
# to your Gemfile and run bundle install
# 
# `rescue in lookup_store': Could not find cache store adapter for
# mem_cache_store (cannot load such file -- dalli) (RuntimeError)
@@@

The error message sums it up: you need `dalli` instead of `memcache-client`.
Adjust Gemfile:

@@@ ruby
# Gemfile

# Remove "gem 'memcache-client'"
gem 'dalli', '~> 2.6.2'
@@@

Finish up with:

@@@ text
$ bundle install
@@@
