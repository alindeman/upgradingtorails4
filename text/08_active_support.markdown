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
