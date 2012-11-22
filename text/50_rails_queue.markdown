## Rails.queue

`Rails.queue` adds a layer between your code and specific background job
processors.

If an existing application uses a gem like `queue_classic`, `delayed_job`,
`resque` or `sidekiq` to manage background tasks, it may benefit from hiding
the specifics of those gems behind the generic facade of `Rails.queue`.

---

It is common to move operations that are computationally expensive or that
communicate with external services to background jobs. This is such a common
need in Rails applications that Rails 4 introduces a construct baked right into
the framework to help.

To use `Rails.queue`, simply wrap expensive operations in  objects that respond
to the `run` method and add instances of them to `Rails.queue` via the shovel
operator (`<<`).

For instance, imagine that publishing a blog post involves communicating with
external services. Therefore, you want to do the publication in a background
worker.

First, wrap the expensive operation in an object:

@@@ ruby
class PostPublisher
  attr_reader :post_id

  def initialize(post_id)
    @post_id = post_id
  end

  def run
    post = Post.find(@post_id)
    # ... publish post ...
  end
end
@@@

Then, add an instance of `PostPublisher` to `Rails.queue`. A likely place for
this code is in a controller:

@@@ ruby
class PostsController < ApplicationController
  def publish
    Rails.queue << PostPublisher.new(params[:id])

    flash[:notice] = "Post is queued for publication!"
    redirect_to posts_url
  end
end
@@@

### Production Setup

By default, `Rails.queue` is a `SynchronousQueue`. `SynchronousQueue` executes
jobs immediately. There is no background processing at all.

This is convenient in development mode because jobs run immediately and
no extra setup is required.

In production, however, it is recommended that you set up a background job
worker backed by a durable queue and run in a separate process. A few options
(and how to configure them with Rails 4) are described next.

#### delayed\_job

[delayed_job](https://github.com/collectiveidea/delayed_job) does not yet
support `Rails.queue`.

#### Resque

[Resque](https://github.com/defunkt/resque) supports `Rails.queue` via the
[resque-rails](https://github.com/jeremy/resque-rails) gem.

Add `resque-rails` to `Gemfile`:

@@@ ruby
gem 'resque-rails', github: 'jeremy/resque-rails', group: [:production]
@@@

Create a configuration file, `config/resque-redis.yml`. Replace the
`production` values with the information for your Redis server:

@@@ text
production:
  host: 10.0.0.1
  port: 6379
@@@

Once configured, jobs added to `Rails.queue` will be run by Resque workers.
Read more about [resque-rails on
GitHub](https://github.com/jeremy/resque-rails).

#### Sidekiq

A prerelease version of [Sidekiq](https://github.com/mperham/sidekiq) supports
`Rails.queue` out of the box.

To try it out today, update `Gemfile` to point to the `rails4` branch:

@@@ ruby
gem 'sidekiq', github: 'mperham/sidekiq', branch: 'rails4'
@@@

And update it via `bundler`:

@@@ text
$ bundle update sidekiq
@@@

Sidekiq automatically configures `Rails.queue`: any jobs added to it will be
run with Sidekiq workers.

### Test Setup

In test mode, `Rails.queue` also defaults to a `SynchronousQueue`. Jobs run
immediately as they do in development.

Rails also gives you the option to simply keep a list of jobs that were queued
without running them. Using a `TestQueue`, tests can verify only that the job
was added to the queue, instead of asserting about its outcome. Assuming the
job is tested in isolation itself, this can speed up the suite without much
loss of confidence in the tests.

To enable this behavior, change the default queue to `ActiveSupport::TestQueue`
for the test environment:

@@@ ruby
# config/environments/test.rb
Widgets::Application.configure do
  # ...

  config.queue = ActiveSupport::TestQueue.new
end
@@@

`Rails.queue` now responds to the `jobs` message, returning a list of queued
jobs:

@@@ ruby
describe PostsController do
  describe "POST #publish" do
    it "queues a job to publish the post" do
      post :publish, id: 1

      job = Rails.queue.jobs.last
      expect(job.post_id).to eq(1)
    end
  end
end
@@@

`TestQueue` will also not allow any item to be enqueued that cannot be
marshalled. More information on marshalling is in the [Gotchas section later on
in the chapter](#queue-gotchas).

### <a id="queue-gotchas"></a>Gotchas

Most background workers that use a separate process will *marshal* the worker
object into a string.

It is therefore important to store only objects that marshal properly. Notably,
it is not a good idea to marshal an ActiveRecord model.

For example, store a Post object's database ID and fetch it again in the
context of the background processor:

@@@ ruby
class PostPublisher
  def initialize(post_id)
    @post_id = post_id
  end

  def run
    post = Post.find(@post_id)
    # ...
  end
end
@@@

Specifically, avoid storing an instance of `Post`, an ActiveRecord model that
is not suitable for marshaling:

@@@ ruby
class PostPublisher
  def initialize(post)
    @post = post ## AVOID
  end

  def run
    # ...
  end
end
@@@

### Asynchronous ActionMailer
