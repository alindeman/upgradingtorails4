## Concerns

The Rails community has embraced the concept of [skinny controllers, fat
models](http://weblog.jamisbuck.org/2006/10/18/skinny-controller-fat-model).
However, while fat models may be better than fat controllers, fat *anything* is
problematic as systems grow.

Rails 4 formalizes the concept of a **concern**: a cross-cutting feature that
encapsulates a behavior in the application. DHH [writes that Basecamp has over
40 concerns](http://37signals.com/svn/posts/3372-put-chubby-models-on-a-diet-with-concerns)
such as "Trashable, Searchable, Visible, Movable, [and] Taggable."

Rails 4 concerns are implemented as Ruby modules and reside in either
`app/models/concerns` (for models) or `app/controllers/concerns` (for
controllers).

### Model Concerns

Imagine a system where records are rarely deleted. Instead, they are merely
archived. Archived records are normally hidden from view, but if the evil
auditing department ever needs access, these archived records can be dug up.

At first, you might add methods and scopes related to archiving directly to
models:

@@@ ruby
# app/models/note.rb
class Note < ActiveRecord::Base
  scope :archived, -> { where(archived: true) }
  scope :unarchived, -> { where(archived: false) }

  def archive
    update_attributes(archived: true)
  end

  # ...
end
@@@

After many models accumulate this same archiving code, you might extract it
into a shared place: a model concern. Concerns are just Ruby modules, though
they normally extend `ActiveSupport::Concern` for some added convenience.

The previous code could be extracted into `Archivable`, stored in
`app/models/concerns/archivable.rb`:

@@@ ruby
# app/models/concerns/archivable.rb
module Archivable
  extend ActiveSupport::Concern

  included do
    scope :archived, -> { where(archived: true) }
    scope :unarchived, -> { where(archived: false) }
  end

  def archive
    update_attributes(archived: true)
  end
end
@@@

Finally, models that can be archived simply include the `Archivable` module:

@@@ ruby
# app/models/note.rb
class Note < ActiveRecord::Base
  include Archivable
end
@@@

<p></p>

@@@ ruby
# app/models/project.rb
class Project < ActiveRecord::Base
  include Archivable
end
@@@

### Controller Concerns

### Use in Rails 3.2
