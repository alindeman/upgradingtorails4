## <a id="testing"></a>Testing

### Directory Structure

Rails 4 has adopted an RSpec-like directory structure by default.

<table>
  <tr>
    <th>Rails 2 and 3 Directory</th>
    <th>Rails 4 Directory</th>
    <th><code>rake</code> task</th>
  </tr>
  <tr>
    <td><code>test/unit</code></td>
    <td><code>test/models</code></td>
    <td><code>rake test:models</code></td>
  </tr>
  <tr>
    <td><code>test/unit/helpers</code></td>
    <td><code>test/helpers</code></td>
    <td><code>rake test:helpers</code></td>
  </tr>
  <tr>
    <td><code>test/functional</code> (for controllers)</td>
    <td><code>test/controllers</code></td>
    <td><code>rake test:controllers</code></td>
  </tr>
  <tr>
    <td><code>test/functional</code> (for mailers)</td>
    <td><code>test/mailers</code></td>
    <td><code>rake test:mailers</code></td>
  </tr>
</table>

For backwards compatibility, running the Rails 3 rake tasks (e.g., `rake
test:units`) will run tests in both the new directories (`test/models` and
`test/helpers`) *and* the old directories (`test/unit`). The new rake tasks
shown in the table above only run tests in the new directories.

I think this is a great improvement, as the terms "unit" and "functional" were
at best opaque and at worst inaccurate.

I think you should take the proactive step of moving all tests to the new
locations after upgrading an application (and before you start writing any new
code). You can use the few commands below to move existing tests into the new
locations.

<!-- TODO: Try this! -->
@@@ text
$ git mv test/unit/helpers test/helpers
$ git mv test/unit test/models

$ mkdir test/mailers
$ git mv test/functional/*mailer_test.rb test/mailers

$ git mv test/functional test/controllers
@@@

If an application has mailers that are not postfixed with `Mailer` (e.g.,
`Notifier` instead of `NotificationsMailer`), you will need to move them
one-by-one from `test/controllers` into `test/mailers`:

@@@ text
$ git mv test/controllers/notifier_test.rb test/mailers
@@@

Finally, commit the result:

@@@ text
$ git commit -m 'Moved tests to Rails 4 conventional locations'
@@@

<!-- TODO: This is a good thing to automate with the rails4_upgrade gem? -->

### RSpec

Speaking of RSpec, if you are using
[**rspec-rails**](https://github.com/rspec/rspec-rails) as your testing
library, upgrade both **rspec-rails** and **rspec-expectations** to version
2.12.1 or greater to get Rails 4 support.

The easiest way is to specify the constraint in your `Gemfile`:

<!-- TODO: Release rspec-rails 2.12.1 -->
@@@ ruby
# Gemfile
group :development, :test do
  gem 'rspec-rails', '>= 2.12.1'
  gem 'rspec-expectations', '>= 2.12.1'
end
@@@

Finally, run `bundle update rspec-rails rspec-expectations` to finish the
upgrade.
