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
