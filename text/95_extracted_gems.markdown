## <a id="extracted-gems"></a>Extracted Gems

During the development of Rails 4, many features that were present in earlier
versions of Rails were removed from Rails itself and extracted to gems.

During the upgrade process, I recommend pulling in all of these gems to make
the upgrade process as smooth as possible. You may not know upfront if your
application makes use of one of these features, and it's one more thing to
worry about breaking.

After your application has been successfully upgraded, though, consider
spending time removing gems that provide features your application does not
use.

The table below describes each of the extracted gems and what it provides:

<table class="table table-bordered table-condensed">
  <thead>
    <tr>
      <th>Gem</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>
        <a href="https://github.com/rails/protected_attributes">protected_attributes</a>
      </th>
      <td>
        Provides <strong>attr_accessible</strong> and
        <strong>attr_protected</strong> for mass-assignment protection. I
        expect that most applications will keep this gem, unless you spend time
        to switch to <a
        href="#strong_parameters"><strong>strong_parameters</strong></a>.
      </td>
    </tr>
    <tr>
      <th>
        <a href="https://github.com/rails/activeresource">activeresource</a>
      </th>
      <td>
        Provides an ActiveRecord-like abstraction over a RESTful API. You can
        quickly figure out if your application uses ActiveResource by searching
        for classes that inherit from <pre>ActiveResource::Base</pre>. More
        details about ActiveResource are available in the <a
        href="#activeresource">ActiveResource</a> chapter.
      </td>
    </tr>
    <tr>
      <th>
        <a href="https://github.com/rails/actionpack-action_caching">actionpack-action_caching</a><br/>
        <a href="https://github.com/rails/actionpack-page_caching">actionpack-page_caching</a>
      </th>
      <td>
        Rails 4 includes many improvements to fragment caching, but action and
        page caching have been extracted. If your application uses the
        <pre>caches_page</pre> or <pre>caches_action</pre> directives in
        any controller, you will need these gems. Otherwise, you can remove
        them.
      </td>
    </tr>
    <tr>
      <th>
        <a href="https://github.com/rails/activerecord-session_store">activerecord-session_store</a>
      </th>
      <td>
        The ability to store session data in a database table has been
        extracted in Rails 4. You need this gem if you store session data in
        the database by setting <pre>config.session_store
        :active_record_store</pre> in
        <pre>config/initializers/session_store.rb</pre>. Otherwise, you do not
        need this gem.
      </td>
    </tr>
    <tr>
      <th>
        <a href="https://github.com/rails/rails-observers">rails-observers</a>
      </th>
      <td>
        Rails no longer encourages the use of observers, separate objects that
        can react to lifecycle events of ActiveRecord models. If you use
        observers--classes that inherit from <pre>ActiveRecord::Observer</pre>,
        you need this gem; otherwise, you can remove it. More information is
        available in the <a href="#observers">Observers</a> chapter.
      </td>
    </tr>
    <tr>
      <th>
        <a href="https://github.com/rails/actionpack-xml_parser">actionpack-xml_parser</a>
      </th>
      <td>
        Following security vulnerabilities involving inbound XML, Rails
        extracts the ability to accept XML input to a gem. If your application
        accepts XML in a request body (note: this is distinct from
        <em>responding with</em> XML), you will need to pull in this gem.
        Otherwise, it can be removed.
      </td>
    </tr>
    <tr>
      <th>
        <a href="https://github.com/rails/rails-perftest">rails-perftest</a>
      </th>
      <td>
        Rails 4 extracts <pre>ActionDispatch::PerformanceTest</pre>. If your
        application includes performance tests (usually located in
        <pre>test/performance</pre>), you need to bundle this gem. Otherwise,
        you can remove it. More information is available in the <a
        href="#performance-tests">Performance Tests</a> chapter.
      </td>
    </tr>
    <tr>
      <th>
        <a href="https://github.com/reed/actionview-encoded_mail_to">actionview-encoded_mail_to</a>
      </th>
      <td>
        Rails previously included a little-known feature to obfuscate email
        addresses in hyperlinks, either with HTML entities or JavaScript code.
        If your application uses the <pre>encode</pre> option with
        <pre>mail_to</pre> in any views, keep this gem. Otherwise, it can be
        removed. More information is available in the <a
        href="#actionview-encoded_mail_to">activerecord-encoded_mail_to</a>
        chapter.
      </td>
    </tr>
  </tbody>
</table>
