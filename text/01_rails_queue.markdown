## Rails.queue

`Rails.queue` adds an abstraction layer between your code and background job
processors

### Pay attention if ...

<!-- TODO: add links to each of these gems -->
* You use a gem like `queue_classic`, `delayed_job`, `resque` or `sidekiq` to
manage background tasks

---

It is common to move operations that are computationally expense or communicate
with external services to background jobs. This is such a common need in Rails
applications that Rails 4 introduces a construct baked right into the
framework to help.

To use `Rails.queue`, simply wrap expensive operations in  objects that respond
to the `run` method and add instances of them to `Rails.queue` via the shovel
operator (`<<`).

For instance, imagine that publishing a blog post involving communicating with
external services. Therefore, you want to do the publication in a background
worker.

First, wrap up the expensive operation in an object:

@@@ ruby
class PostPublisher
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
