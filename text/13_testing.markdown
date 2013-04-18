## <a id="testing"></a>Testing

### <a id="performance-tests"></a>Performance Tests

Rails 3 included a framework for writing performance tests. For example, you
might have written a performance test to measure various metrics (wall clock
time, memory usage, etc...) of a controller action. Plotting these metrics over
time could help pinpoint where a performance degredation was introduced.

Rails 4 no longer includes this framework by default. If your existing Rails
application has performance tests, pull in the `rails-perftest` gem:

@@@ ruby
# Gemfile

gem 'rails-perftest'
@@@

More information about Rails performance tests, including examples, can be
found in [the README for the rails-perftest
project](https://github.com/rails/rails-perftest).

### RSpec

If you are using [**rspec-rails**](https://github.com/rspec/rspec-rails) as
your testing library, upgrade **rspec-rails** to version 2.13.1 or greater to
get Rails 4 support.

The easiest way is to specify the constraint in your `Gemfile`:

@@@ ruby
# Gemfile
group :development, :test do
  gem 'rspec-rails', '>= 2.13.1'
end
@@@

Finally, run `bundle update rspec-rails` to finish the upgrade.
