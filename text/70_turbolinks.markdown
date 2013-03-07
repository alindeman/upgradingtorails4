## <a id="turbolinks"></a>Turbolinks

<!-- TODO: Make sure this stays true -->
New Rails 4 applications include the **turbolinks** gem, which can sometimes
make your application faster by avoiding full page refreshes as a user
navigates through an application by clicking links.

When **turbolinks** is enabled and a user clicks on a link, the request is sent
in the background using AJAX. The response still contains a fully rendered HTML
page, so **turbolinks** does not necessarily save much bandwidth.  However, it
does achieve a significant speedup by splicing out both the `<title>` of the
new page and the content in the `<body>` section, and replacing that content in
the current document.

Because the page did not refresh completely, the stylesheets (CSS) and
JavaScript assets do not have to be parsed again by the browser. Because modern
web applications tend to have a lot of CSS and JavaScript, **turbolinks** can
make your application feel a lot snappier.

**turbolinks** is supported in Safari 6.0+ (but *not* Safari 5), IE10, and
recent versions of Google Chrome and Mozilla Firefox.

**turbolinks** is similar to [pjax](http://pjax.heroku.com/), but ideally
operates transparently so that no changes are needed beyond enabling it in your
application. In practice, of course, there are [gotchas](#turbolinks-gotchas)
that I touch on later in the chapter.

### Adding turbolinks to existing applications

**turbolinks** is packaged as a gem, so existing applications will need to
explicitly install it. New Rails 4 applications are generated with **turbolinks**
in the generated `Gemfile`.

Notably, the **turbolinks** gem is compatible with Rails 3 so long as the
application uses the asset pipeline.

Add **turbolinks** to `Gemfile` and use bundler to install it:

@@@ ruby
# Gemfile
gem 'turbolinks'
@@@

<p></p>

@@@ text
$ bundle install
@@@

Finally, include `turbolinks` in `app/assets/javascripts/application.js`:

@@@ javascript
// app/assets/javascripts/application.js

// ...
//= require turbolinks
@@@

After restarting `rails server`, you can verify that **turbolinks** is working
correctly by using the web developer tools. In Google Chrome, for instance,
developer tools can be enabled via the "Tools -> Developer Tools" menu item.
Navigate to a page in the turbolinkified application and select the "Network"
tab:

![turbolinks in action](../images/turbolinks_in_action.png)

In this example, the last three requests were handled by **turbolinks.js**. You
can tell by looking at the "Initiator" column.

From a user's perspective, everything worked as if the entire page *had*
refreshed, except it did not; the page was speedier for having avoided
loading stylesheets and JavaScript again.

### Graceful Degredation

**turbolinks** gracefully degrades when a browser does not support its features.
For instance, in Safari 5 links will simply work as if **turbolinks** were
not present at all.

Furthermore, **turbolinks** detects when stylesheets or JavaScript assets have
changed on the page being requested. If the assets are different than on the
current page, **turbolinks** initiates a full page refresh so that those
new assets are correctly loaded.

In that degenerate case, the page will actually be loaded *twice*: one time to
check if the assets have changed, and a second time to load in the user's
browser.

That degenerate case will perform even worse than if **turbolinks** were not
present at all. To avoid this performance nightmare, make sure that the
**turbolinks** JavaScript is the last asset loaded in the `<head>` of your
application's pages.

In practice this often means making sure that `<%= javascript_include_tag
"application" %>` is the *last* piece of JavaScript to be loaded in a layout
(the default layout is located in `app/views/layouts/application.html.erb`).
If there are additional `javascript_include_tag` or `stylesheet_link_tag` calls
after turbolinks, you may suffer performance issues; in that case, consider
reordering the `<head>` content so **turbolinks** is last in line.

### <a id="turbolinks-gotchas"></a>Gotchas

**turbolinks** may negatively affect JavaScript that runs code triggered by the
jQuery `$(document).ready` event. With **turbolinks** enabled,
`$(document).ready` runs only when the first page is ready, not each time a new
page is loaded through **turbolinks**.

Consider a view that contains a button:

@@@ erb
<%# hello.html.erb %>
<button id="hello-button">Hello!</button>
@@@

Next, consider the corresponding JavaScript code that attachs to the button's
`click` event to display an alert when the button is clicked:

@@@ javascript
$(document).ready(function() {
  $("#hello-button").on("click", function() {
    alert("Hello!");
  });
});
@@@

However, if `hello.html.erb` is displayed through **turbolinks** (i.e., it is
not the first page loaded in the web application), the `$(document).ready`
event will not be triggered, and the event handler will not be attached to the
button.

To fix the problem, it is necessary to attach the event handler both in
`$(document).ready` (which will handle the case where the page is the first to
load) and `$(document).on("page:load")` (which will handle the case where the
page is loaded through **turbolinks**).

@@@ javascript
var attachClickHandler = function() {
  $("#hello-button").on("click", function() {
    alert("Hello!");
  });
};

$(document).ready(attachClickHandler);
$(document).on("page:load", attachClickHandler);
@@@

The `"page:load"` event is triggered when **turbolinks** loads a new page.

While it is good to know how to manually fix the issue, a library called
[jquery.turbolinks](https://github.com/kossnocorp/jquery.turbolinks) has also been
created to fix the issue described here, ideally without any code changes.

That is, in the presence of **jquery.turbolinks**, the original code will work
correctly as `$(document).ready` will be fired in both cases.

### Additional Resources

* [turbolinks on GitHub](https://github.com/rails/turbolinks)
* [Railscasts #390: Turbolinks](http://railscasts.com/episodes/390-turbolinks)
* [Turbolinks Compatibility](http://reed.github.com/turbolinks-compatibility/)
