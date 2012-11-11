## <a id="forking-and-loosening-constraints"></a>Forking Gems to Loosen Version Constraints

If in the course of upgrading an application to Rails 4, `bundler` detects a
conflict between a gem and `rails` (or one of its supporting gems like
`activerecord`), it is *possible* that the gem is actually compatible and
simply needs its dependency constraints loosened.

If you run into a gem that is not yet compatible with Rails 4, running `bundle`
commands will result in an error message similar to:

@@@ text
Bundler could not find compatible versions for gem "actionpack":
  In Gemfile:
    simple_form (~> 2.0.4) ruby depends on
      actionpack (~> 3.0) ruby

    rails (>= 0) ruby depends on
      actionpack (4.0.0.beta)
@@@

The examples in this chapter use the [simple_form
gem](https://github.com/plataformatec/simple_form), a gem that makes producing
forms easier. However, you can attempt this process for any gem that is
currently constrainted to Rails 3 only.

**As a cautionary note,** the process described here only works as written if
the gem is already compatible with Rails 4, yet its dependency constraints are
too narrowly focused on Rails 3. If the gem's implementation is not compatible
with Rails 4, more work will be needed before it can be used in an upgraded
application.

### Forking the Source

Most Ruby gems' source code is hosted on GitHub. GitHub makes it easy to create
a copy of the source code repository for a project in order to modify it.
GitHub calls this process *forking*.

To fork a repository, you will need a GitHub account. A free GitHub account is
easy to create via [sign up page](https://github.com/signup/free).

Next, find the source code repository for a gem by searching for it on
[rubygems.org](http://rubygems.org):

![searching for simple_form on rubygems.org](../images/searching_for_simple_form.png)

Click on the first result and find the "Source Code" link:

![simple_form source code link](../images/simple_form_source_code.png)

You will likely be directed to the source code on GitHub. Click the "Fork"
button in the top right. If there is no "Fork" button, make sure to login
to GitHub first.

![fork simple_form](../images/fork_simple_form.png)

After a few seconds, the fork will be created:

![simple_form fork successfully created](../images/simple_form_fork_created.png)

Using the SSH or HTTP URL, clone the repository to your local machine and
navigate into the directory for the gem:

@@@ text
$ git clone git@github.com:alindeman/simple_form.git
$ cd simple_form
@@@

Look for a file with the extension `.gemspec` corresponding to the gem name.
In the case of `simple_form`, there is a `simple_form.gemspec`. Open it in
your favorite text editor.

Find lines that call the `add_dependency` method for the `rails` gem or any
subproject of Rails such as `activerecord`, `activemodel`, or `actionpack`.
`simple_form` currently depends on two such gems:

@@@ ruby
# simple_form.gemspec
Gem::Specification.new do |s|
  # ... other directives ...

  s.add_dependency('activemodel', '~> 3.0')
  s.add_dependency('actionpack', '~> 3.0')
end
@@@

The '~> 3.0' constraint allows the last digit of the version to vary. For
instance, '~> 3.0' allows `activemodel` and `actionpack` version 3.0.x, 3.1.x,
and 3.2.x, but not 4.0.x. This is our problem!

Change the `simple_form.gemspec` so it is more forgiving. The easiest way is
to replace `~>` with `>=`. `>=` specifies that any version greater than or
equal to 3.0 is allowed, including 4.0.

@@@ ruby
Gem::Specification.new do |s|
  # ... other directives ...

  s.add_dependency('activemodel', '>= 3.0')
  s.add_dependency('actionpack', '>= 3.0')
end
@@@

Save the file, commit it via git, and push it back to GitHub:

@@@ text
$ git add simple_form.gemspec
$ git commit -m 'Loosens constraints to support Rails 4.0'
$ git push origin master
@@@

Next, navigate back to the Rails application that depends on `simple_form` (or
whichever gem you're upgrading) and open up its `Gemfile`. Replace the line
that currently specifies `simple_form` as a dependency with a line that
points to the newly forked version.

@@@ ruby
# Gemfile

# Replaces gem 'simple_form', 'X.Y.Z'
# Use your GitHub username instead of 'alindeman'
gem 'simple_form', github: 'alindeman/simple_form'
@@@

Finally, ask Bundler to install the newly updated version of `simple_form` by
running `bundle install`:

@@@ text
$ bundle install
@@@

If all went well, output similar to below will be shown:

@@@ text
...
Using simple_form (2.1.0.dev) from git://github.com/alindeman/simple_form.git (at master)
...
@@@

This means that Bundler is using your version of `simple_form` instead of the
official one which (in this example) does not support Rails 4.

It may be necessary to perform these steps for many gems in order to upgrade an
application. Again, it may feel a bit like the whack-a-mole game.

Because it can be a significant amount of work to make a gem compatible with
Rails 4, consider [creating a pull
request](https://help.github.com/articles/creating-a-pull-request) so that the
gem maintainer can bring in the code changes in the official gem repository.
