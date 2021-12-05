# Morphs

By default, StimulusReflex updates your entire page. After re-processing your controller action, rendering your view template, and sending the raw HTML to your browser, StimulusReflex uses the amazing [morphdom](https://github.com/patrick-steele-idem/morphdom) library to do the smallest number of DOM modifications necessary to refresh your UI in just a few milliseconds. For many developers, this will be a perfect solution forever. They can stop reading here.

Most real-world applications are more sophisticated, though. You think of your site in terms of sections, components and content areas. We reason about our functionality with abstractions like _"sidebar"_ but when it's time to work, we shift back to contemplating a giant tree of nested containers. Sometimes we need to surgically swap out one of those containers with something new. Sending the entire page to the client seems like massive overkill. We need to update just part of the DOM without disturbing the rest of the tree... _and we need it to happen in \~10ms_.

Other times... we just need to hit a button which feeds a cat which may or may not still be alive in a steel box. 🐈

It's _almost_ as if complex, real-world scenarios don't always fit the one-size-fits-all default full page Reflex.

### &#x20;<a href="#introducing-morphs" id="introducing-morphs"></a>

Behind the scenes, there are actually three different _modes_ in which StimulusReflex can process your requests. We refer to them by _what they will replace_ on your page: **Page**, **Selector** and **Nothing**. All three benefit from the same logging, callbacks, events and promises.

Changing the Morph mode happens in your server-side Reflex class, either in the action method or the callbacks. Both markup e.g. `data-reflex` and programmatic e.g. `stimulate()` mechanisms for initiating a Reflex on the client work without modification.

`morph` is only available in Reflex classes, not controller actions. Once you change modes, you cannot change between them.

![](../.gitbook/assets/3e604acbd337e49c99714991944573ab99fcbccf)

Each Morph is useful in different scenarios.

Process Controller Action?

All children of a CSS DOM **selector**

### &#x20;<a href="#page-morphs" id="page-morphs"></a>

Page morphs are the default behavior of StimulusReflex and they are what will occur if you don't call `morph` in your Reflex.

All Reflexes are, in fact, Page morphs - until they are not. 👴⚗️

What makes Page Morphs interesting and distinct from other Morph types is that they are the only one that re-runs the page's controller action before rendering the new HTML markup.

Any instance variables that you set in your Reflex action method are available to your controller action. In addition, there is a special `@stimulus_reflex` variable that is set to `true` when a controller action is being run by a Reflex.

StimulusReflex does not support using redirect\_to in a Page Morph. If you try to return an HTTP 302 in your controller during a Reflex action, your page content will become "You are being redirected."

### &#x20;<a href="#scoping-page-morphs" id="scoping-page-morphs"></a>

Instead of updating your entire page, you can specify exactly which parts of the DOM will be updated using the `data-reflex-root` attribute.

`data-reflex-root=".class, #id, [attribute]"`

Simply pass a comma-delimited list of CSS selectors. Each selector will retrieve one DOM element; if there are no elements that match, the selector will be ignored.

StimulusReflex will decide which element's children to replace by evaluating three criteria in order:

1.  1\.

    Is there a `data-reflex-root` on the element with the `data-reflex`?
2.  2\.

    Is there a `data-reflex-root` on an ancestor element above the element in the DOM? It could be the element's immediate parent, but it doesn't have to be.
3.  3\.

    Just use the `body` element.

Here is a simple example: the user is presented with a text box. Anything they type into the text box will be echoed back in two div elements, forwards and backwards.

\<div data-reflex-root="\[forward],\[backward]">

\<input type="text" value="<%= @words %>" data-reflex="keyup->Example#words">

\<div forward><%= @words %>\</div>

\<div backward><%= @words&.reverse %>\</div>

class ExampleReflex < ApplicationReflex

One interesting detail of this example is that by assigning the root to `[forward],[backward]` we are implicitly telling StimulusReflex to **not** update the text input itself. This prevents resetting the input value while the user is typing.

In StimulusReflex, morphdom is called with the **childrenOnly** flag set to _true_.

This means that \<body> or the custom parent selector(s) you specify are not updated. For this reason, it's necessary to wrap anything you need to be updated in a div, span or other bounding tag so that it can be swapped out without confusion.

If you're stuck with an element that just won't update, make sure that you're not attempting to update the attributes on an \<a>.

It's completely valid for an element with a data-reflex-root attribute to reference itself via a CSS class or other mechanism. Just always remember that the parent itself will not be replaced! Only the children of the parent are modified.

### &#x20;<a href="#permanent-elements" id="permanent-elements"></a>

Perhaps you just don't want a section of your DOM to be updated by StimulusReflex. Perhaps you need to integrate 3rd-party elements such as ad tracking scripts, Google Analytics, and any other widget that renders itself such as a React component or legacy jQuery plugin.

Just add `data-reflex-permanent` to any element in your DOM, and it will be left unchanged by full-page Reflex updates and `morph` calls that re-render partials. Note that `morph` calls which insert simple strings or empty values do not respect the `data-reflex-permanent` attribute.

\<div data-reflex-permanent>

\<iframe src="https://ghbtns.com/github-btn.html?user=stimulusreflex\&repo=stimulus\_reflex\&type=star\&count=true" frameborder="0" scrolling="0" class="ghbtn">\</iframe>

\<iframe src="https://ghbtns.com/github-btn.html?user=stimulusreflex\&repo=stimulus\_reflex\&type=fork\&count=true" frameborder="0" scrolling="0" class="ghbtn">\</iframe>

We have encountered scenarios where the `data-reflex-permanent` attribute is ignored unless there is a unique `id` attribute on the element as well. If you are working with the [Trix](https://trix-editor.org) editor ([ActionText](https://guides.rubyonrails.org/action\_text\_overview.html)) you absolutely must use `data-reflex-permanent` and specify an `id` attribute.

Please let us know if you can identify this happening in the wild, as technically it shouldn't be necessary... and yet, it works.

Beware of Ruby gems that implicitly inject HTML into the body as it might be removed from the DOM when a Reflex is invoked. For example, consider the [intercom-rails gem](https://github.com/intercom/intercom-rails) which automatically injects the Intercom chat into the body. Gems like this often provide [instructions](https://github.com/intercom/intercom-rails#manually-inserting-the-intercom-javascript) for explicitly including their markup. We recommend using the explicit option whenever possible, so that you can wrap the content with `data-reflex-permanent`.

### &#x20;<a href="#selector-morphs" id="selector-morphs"></a>

This is the perfect option if you want to re-render a partial, update a counter or just set a container to empty. Since it accepts a string, you can pass a value to it directly, use `render` to regenerate a partial or even connect it to a ViewComponent.

Updating a target element with a Selector morph does _not_ invoke ActionDispatch. There is no routing, your controller is not run, and the view template is not re-rendered. This means that if your content is properly fragment cached, you should see round-trip updates in **10-15ms**... which is a nice change from the before times. 🐇

### &#x20;<a href="#tutorial" id="tutorial"></a>

Let's first establish a baseline HTML sample to modify. Our attention will focus primarily on the `div` known colloquially as **#foo**.

\<header data-reflex="click->Example#change">

<%= render partial: "path/to/foo", locals: {message: "Am I the medium or the massage?"} %>

Behold! For this is the `foo` partial. It is an example of perfection:

\<span class="spa"><%= message %>\</span>

You create a Selector morph by calling the `morph` method. In its simplest form, it takes two parameters: **selector** and **html**. We pass any valid CSS DOM selector that returns a reference to the first matching element, as well as the value we're updating it with.

app/reflexes/example\_reflex.rb

class ExampleReflex < ApplicationReflex

morph "#foo", "Your muscles... they are so tight."

If you consult your Elements Inspector, you'll now see that #foo now contains a text node and your `header` has gained some attributes. This is just how StimulusReflex makes the magic happen.

\<header data-reflex="click->Example#change" data-controller="stimulus-reflex" data-action="click->stimulus-reflex#\_\_perform">

\<div id="foo">Your muscles... they are so tight.\</div>

**Morphs only replace the children of the element that you are targeting.** If you need to update the target element (as you would with `outerHTML`) consider targeting the parent of the element you need to change. You could, for example, call `morph "header", "No more #foo."` and start fresh.

Cool, but _where did the span go_? We're glad you asked!

The truth is that a lot of complexity and nasty edge cases are being hidden away, while presenting you intelligent defaults and generally trying to follow the _principle of least surprise_.

There's no sugar coating the fact that there's a happy path for all of the typical use cases, and lots of gotchas to be mindful of otherwise. We're going to tackle this by showing you best practices first. Start by #winning now and later there will be a section with all of the logic behind the decisions so you can troubleshoot if things go awry / [Charlie Sheen](https://www.youtube.com/watch?v=pipTwjwrQYQ).

### &#x20;<a href="#intelligent-defaults" id="intelligent-defaults"></a>

**Morphs work differently depending on whether you are replacing existing content with a new version or something entirely new.** This allows us to intelligently re-render partials and ViewComponents based on data that has been changed in the Reflex action.

yelling = element.value.upcase

morph "#foo", render(partial: "path/to/foo", locals: {message: yelling})

Since StimulusReflex v3.4, `render` has been delegated to the controller class responsible for rendering the current page. Of course, you're still free to use `ApplicationController` or any other ActionDispatch controller to render your content.

You'll have access to all the same helpers that you would in a normal Rails HTTP request and the subsequent SSR handling of it.

If ViewComponents are your thing, we have you covered:

morph "#foo", render(FooComponent.new(message: "React is making your muscles sore."))

The `foo` partial (listed in the [Tutorial ](broken-reference)section above) is an example of a best practice for several subtle but important reasons which you should use to model your own updates:

* it has a **single** top-level container element with the same CSS selector as the target
* inside that container element is another [element node](https://developer.mozilla.org/en-US/docs/Web/API/Node/nodeType), **not a text node**

If you can follow those two guidelines, you will see several important benefits regardless of how the HTML stream is generated:

* DOM changes will be performed by the morphdom library, which is highly efficient
* morph will respect elements with the `data-reflex-permanent` attribute
* any event handlers set on contents should remain intact (unless they no longer exist)

As you have already seen, it's **okay** to morph a container with a string, or a container element that has a different CSS selector. However, `morph` will treat these updates _slightly_ differently:

* DOM elements are replaced by updating innerHTML
* elements with the `data-reflex-permanent` attribute will be over-written
* any event handlers on replaced elements are immediately de-referenced
* you could end up with a nested container that might be jarring if you're not expecting it

Let's say that you update #foo with the following morph:

morph "#foo", "\<div id=\\"foo\\">Let's do something about those muscles.\</div>"

This update will use morphdom to update the existing #foo div. However, because #foo contains a [text node](https://developer.mozilla.org/en-US/docs/Web/API/Node/nodeType), `data-reflex-permanent` is ignored. (Sorry! We just work here.)

morph "#foo", "\<div id=\\"baz\\">\<span>Just breathe in... and out.\</span>\</div>"

Now your content is contained in a `span` element node. All set... except that you changed #foo to #baz.

\<header data-reflex="click->Example#change" data-controller="stimulus-reflex" data-action="click->stimulus-reflex#\_\_perform">

\<span>Just breathe in... and out.\</span>

That's great - if that's what you want. 🤨

Ultimately, we've optimized for two primary use cases for morph functionality:

1.  1\.

    Updating a partial or ViewComponent to reflect a state change.
2.  2\.

    Updating a container element with a new simple value or HTML fragment.

### &#x20;<a href="#real-world-example-pagy-refactoring" id="real-world-example-pagy-refactoring"></a>

If you're doing pagination in Rails, [pagy](https://github.com/ddnexus/pagy) is the tool for the job. pagy works great with StimulusReflex, Bootstrap and FontAwesome:

\<div id="paginator"><%= render partial: "paginator", locals: {pagy: @pagy} %>\</div>

\<div id="posts"><%= render @posts %>\</div>

@pagy, @posts = pagy(Post.all, page: 1)

class PagyReflex < ApplicationReflex

pagy, posts = pagy(Post.all, page: element.dataset\[:page].to\_i)

morph "#paginator", render(partial: "paginator", locals: {pagy: pagy})

morph "#posts", render(posts)

\<nav class="d-flex justify-content-center">

\<li class="page-item">\<a href="#" id="page\_prev\_li" class="page-link" data-reflex="click->Pagy#paginate" data-page="<%= pagy.prev || 1 %>">\<span class="far fa-angle-double-left">\</span>\</a>\</li>

<% pagy.series.each do |item| %>

\<li class="page-item disabled">\<a class="page-link" id="page\_gap\_li">...\</a>\</li>

\<li class="page-item <%= "active" if item.is\_a?(String) %>">

\<a href="#" id="page\_<%= item %>\_li" class="page-link" data-reflex="click->Pagy#paginate" data-page="<%= item %>"><%= item %>\</a>

\<li class="page-item">\<a href="#" id="page\_next\_li" class="page-link" data-reflex="click->Pagy#paginate" data-page="<%= pagy.next || pagy.last %>">\<span class="far fa-angle-double-right">\</span>\</a>\</li>

Hang on, though... if you watch the [client-side logging](broken-reference) when you click the button to advance to the 2nd page, you'll see that both `morph` calls used CableReady `inner_html` operations to update the divs. While this might be fine for some applications, `inner_html` completely wipes out any Stimulus controllers present in the replaced DOM hierarchy and doesn't respect the `data-reflex-permanent` attribute. How can we adapt this so that both `morph` operations are performed by the `morphdom` library?

The `paginator` partial is only rendered one time, so this one is easy: we have to move the top-level div into the partial. When it gets re-rendered, it will automatically match what `morph` needs to update the contents because it _is_ the contents:

<%= render partial: "paginator", locals: {pagy: @pagy} %>

\<div id="posts"><%= render @posts %>\</div>

\<nav class="d-flex justify-content-center">

\<li class="page-item">\<a href="#" id="page\_prev\_li" class="page-link" data-reflex="click->Pagy#paginate" data-page="<%= pagy.prev || 1 %>">\<span class="far fa-angle-double-left">\</span>\</a>\</li>

<% pagy.series.each do |item| %>

\<li class="page-item disabled">\<a class="page-link" id="page\_gap\_li">...\</a>\</li>

\<li class="page-item <%= "active" if item.is\_a?(String) %>">

\<a href="#" id="page\_<%= item %>\_li" class="page-link" data-reflex="click->Pagy#paginate" data-page="<%= item %>"><%= item %>\</a>

\<li class="page-item">\<a href="#" id="page\_next\_li" class="page-link" data-reflex="click->Pagy#paginate" data-page="<%= pagy.next || pagy.last %>">\<span class="far fa-angle-double-right">\</span>\</a>\</li>

The `posts` partial (not listed) is rendered as a collection, and so it must be handled differently. You cannot put the top-level div into each element of the collection!

Instead, simply wrap the `render` call itself with markup for the top-level div:

morph "#posts", "\<div id=\\"posts\\">" + render(posts) + "\</div>"

Now, both `paginator` and `posts` are being updated using `morphdom`.

### &#x20;<a href="#morphing-multiplicity" id="morphing-multiplicity"></a>

morph "#username": "hopsoft", "#notification\_count": 5

You can call `morph` multiple times in your Reflex action method.

You can use Ruby's implicit Hash syntax to update multiple selectors with one morph. These updates will all be sent as part of the same broadcast, and executed in the order they are defined. Any non-String values will be coerced into Strings. Passing no html argument is equivalent to `""`.

### &#x20;<a href="#dom_id" id="dom_id"></a>

One of the best perks of Rails naming conventions is that you can usually calculate what the name of an element or resource will be programmatically, so long as you know the class name and id.

Inside a Reflex class, you might find yourself typing code like:

morph "#user\_#{user.id}", user.name

The [dom\_id](https://apidock.com/rails/v6.0.0/ActionView/RecordIdentifier/dom\_id) helper is available inside Reflex classes and supports the optional prefix argument:

morph dom\_id(user), user.name

### &#x20;<a href="#view-helpers-that-emit-urls" id="view-helpers-that-emit-urls"></a>

If you are planning to render a partial that uses Rails routing view helpers to create URLs, you will need to [set up your environment configuration files](broken-reference) to make sure that your site's URL is available inside your Reflexes.

You'll know that you forgot this step if your URLs are coming out as **example.com**.

### &#x20;<a href="#things-go-wrong..." id="things-go-wrong..."></a>

We've worked really hard to make morphs easy to work with, but there are some rules and edge cases that you have to follow if you want your Selector Morphs to use a CableReady `morph` operation instead of an `inner_html` operation.

If you're not getting the results you expect, please consult the [Morphing Sanity Checklist](broken-reference) to make sure you're not accidentally using the wrong operation. Use of [radiolabel](https://github.com/leastbad/radiolabel) can help provide an early warning.

### &#x20;<a href="#nothing-morphs" id="nothing-morphs"></a>

Your user clicks a button. Something happens on the server. The browser is notified that this task was completed via the usual callbacks and events.

Sometimes you want to take advantage of the chasis and infrastructure of StimulusReflex, without any assumptions or expectations about changing your DOM afterwards. The bare metal nature of Nothing morphs means that the time between initiating a Reflex and receiving a confirmation can be low single-digit milliseconds, if you don't do anything to slow it down.

Nothing morphs usually initiate a long-running process, such as making calls to APIs or supervising I/O operations like file uploads or video transcoding. However, they are equally useful for emitting signals; you could send messages into a queue, tell your media player to play, or tell your Arduino to launch the rocket.

The key strategy when working with Nothing morphs is to **avoid blocking calls at all costs**. While we might still be years away from a viable asynchronous Ruby ecosystem, using Reflexes to initiate Rails ActiveJob instances - ideally processed by Sidekiq and powered by Redis - provides a reliable platform that financial institutions still pay millions of dollars to achieve.

In a sense, Nothing morphs are _yin_ to CableReady's _yang_. A Reflex conveys user intent to the server, and a broadcast is the vehicle for server intent to be realized on the client, completing the circle. ☯️

#### &#x20;<a href="#i-cant-take-the-suspense.-how-can-i-capture-this-raw-power-for-myself" id="i-cant-take-the-suspense.-how-can-i-capture-this-raw-power-for-myself"></a>

It's wickedly hard... but with practice, you'll be able to do it, too:

That's it. That's the entire API surface. 🙇

### &#x20;<a href="#multi-stage-morphs" id="multi-stage-morphs"></a>

You can morph the same target element multiple times in one Reflex by calling CableReady directly. One clever use of this technique is to morph a container to display a spinner, call an API or access computationally intense results - which you cache for next time - and then replace the spinner with the new update... all in the same Reflex action.

cable\_ready\[stream\_name].morph({ spinner... }).broadcast

cable\_ready\[stream\_name].morph({ progress update... }).broadcast

cable\_ready\[stream\_name].morph({ final update... }).broadcast

### &#x20;<a href="#activejob-example" id="activejob-example"></a>

Let's step through creating a simple ActiveJob that will be triggered by a Nothing morph. Upon completion, the job will increment a counter and direct CableReady to update the browser. Note that you'll have to ensure that your ActiveJob infrastructure is up and running, ideally backed by Sidekiq and Redis.

First, some quick housekeeping: you need to create an ActionCable channel. Running `rails generate channel counter` should do the trick. We want to stream updates to anyone listening in on the `counter` stream.

app/channels/counter\_channel.rb

class CounterChannel < ApplicationCable::Channel

When the channel client receives data, send it to CableReady for processing.

app/javascript/channels/counter\_channel.js

import consumer from "./consumer"

consumer.subscriptions.create("CounterChannel", {

if (data.cableReady) CableReady.perform(data.operations)

Create a simple view template that contains a `button` to launch the Reflex as well as a `span` to hold the current value. We'll pull in the current value of the counter key in the Rails cache. If it doesn't yet exist, set the value to 0.

\<button data-reflex="click->Counter#increment">Increment Counter\</button>

The counter currently reads: \<span id="counter"><%= Rails.cache.fetch("counter", raw: true) {0} %>\</span>

This is the complete implementation of a minimum viable Nothing morph Reflex action. Note that in a real application, you would almost certainly pass parameter arguments into your ActiveJob constructor. An ActiveJob can accept [a wide variety of data types](https://guides.rubyonrails.org/active\_job\_basics.html#supported-types-for-arguments). This includes ActiveRecord models, which makes use of the [Global ID](https://guides.rubyonrails.org/active\_job\_basics.html#globalid) system behind the scenes.

You need to give the job enough information to successfully broadcast any important results to the correct places. For example, if you're planning to broadcast notifications to a specific user, make sure to pass the user resource (ActiveRecord model instance) to the ActiveJob.

app/reflexes/counter\_reflex.rb

class CounterReflex < ApplicationReflex

IncrementJob.set(wait: 3.seconds).perform\_later

Finally, the job includes CableReady::Broadcaster so that it can send commands back to the client. We then use CableReady to queue up a text\_content operation with the newly incremented value before ultimately sending the broadcast.

app/jobs/increment\_job.rb

class IncrementJob < ApplicationJob

include CableReady::Broadcaster

cable\_ready\["counter"].text\_content(selector: "#counter", text: Rails.cache.increment("counter")).broadcast

This setup might seem like overkill to increment a number on your page, but you only need to setup the channel once, and then you really just need an ActiveJob class to make the magic happen. You can use these examples as starting points for applications of arbitrary sophistication and complexity.

There's an amazing resource on best practices with ActiveJob and Sidekiq [available here](https://github.com/toptal/active-job-style-guide).
