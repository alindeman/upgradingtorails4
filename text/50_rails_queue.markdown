## Rails.queue

`Rails.queue` adds an abstraction layer between your code and background job
processors.

If an existing application uses a gem like `queue_classic`, `delayed_job`,
`resque` or `sidekiq` to manage background tasks, it may benefit from hiding
the specifics of those gems behind the generic facade of `Rails.queue`.

---

It is common to move operations that are computationally expense or communicate
with external services to background jobs. This is such a common need in Rails
applications that Rails 4 introduces a construct baked right into the framework
to help.

To use `Rails.queue`, simply wrap expensive operations in  objects that respond
to the `run` method and add instances of them to `Rails.queue` via the shovel
operator (`<<`).

For instance, imagine that publishing a blog post involving communicating with
external services. Therefore, you want to do the publication in a background
worker.

First, wrap up the expensive operation in an object:

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

### Development

By default in development mode, `Rails.queue` is a `SynchronousQueue`.
`SynchronousQueue` executes jobs added to it immediately without you having to
setup a production-like backgroud job worker.

### Test

By default in test mode, `Rails.queue` is a `TestQueue`. `TestQueue` does not
execute jobs at all, but instead provides a way to access the list of jobs that
are in the queue in your tests:

@@@ ruby
# TODO: This has not actually be tested to work properly
describe PostsController do
  describe "POST #publish" do
    it "queues a job to publish the post" do
      post :publish, id: 1

      job = Rails.queue.jobs.first
      expect(job).not_to be_nil
      expect(job.post_id).to eq(1)
    end
  end
end
@@@

<!-- TODO: Link to Gotchas section? --> `TestQueue` will also not allow any
item to be enqueued that cannot be marshalled.  More information on marshalling
is in the Gotchas section later on in the chapter

### Production

By default in production, `Rails.queue` is an `ActiveSupport::Queue`. Rails
also spawns a consumer, by default a `ThreadedQueueConsumer`, which spawns a
background thread and runs jobs in that thread.

While jobs do run in the background and there is no additional setup required,
it is not a recommend setup because the queue is entirely in memory and jobs
would be lost if the web server were restarted. Furthermore, jobs cannot be
distributed to other servers because they run in the web server process.  

In production, it is recommended that you setup a background job worker backed
by a durable queue and run in a separate process. A few options (and how to
configure them with Rails 4) are described next.

#### delayed\_job

[delayed_job](https://github.com/collective_idea/delayed_job) supports
`Rails.queue` via `TODO`

#### Resque

[Resque](https://github.com/defunkt/resque) supports `Rails.queue` via `TODO`

#### Sidekiq

[Sidekiq](https://github.com/mperham/sidekiq) supports `Rails.queue` via
`Sidekiq::Client::Queue`

@@@ ruby
# config/environments/production.rb
Blog::Application.configure do
  config.queue = Sidekiq::Client::Queue
end
@@@

### Multiple Queues

TODO: If I'm able to get to it. Lower priority

### Gotchas

Most background processors that use a separate process will *marshal* the
worker object into a string.

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

class PostPublisher
  def initialize(post)
    @post = post ## AVOID
  end

  def run
    # ...
  end
end
@@@
