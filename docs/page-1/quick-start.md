# Quick Start

How to use StimulusReflex in your app

### &#x20;<a href="#before-you-begin..." id="before-you-begin..."></a>

**A great user experience can be created with Rails alone.** Tools such as [Russian Doll caching](https://www.speedshop.co/2015/07/15/the-complete-guide-to-rails-caching.html), [UJS](https://guides.rubyonrails.org/working\_with\_javascript\_in\_rails.html#remote-elements), [Stimulus](https://stimulusjs.org), and [Turbo Drive](https://turbo.hotwire.dev/handbook/drive) are incredibly powerful when combined. Could you build your application using these tools without introducing StimulusReflex?

We are only alive for a short while and learning any new technology is a sacrifice of time spent with those you love, creating art or walking in the woods. 👨‍👨‍👧‍👧🎨🌲

Every framework you learn is a lost opportunity to build something that could really matter to the world. **Please choose responsibly.** ⏳

It might strike you as odd that we would start by questioning whether you need this library at all. Our motivations are an extension of the question we hope more people will ask.

Instead of _"Which Single Page App framework should I use?"_ we believe that StimulusReflex can empower people to wonder "**Do we still need React, given what we now know is possible?**" 🤯

### &#x20;<a href="#video-tutorial-introduction-to-stimulusreflex" id="video-tutorial-introduction-to-stimulusreflex"></a>

​[Chris ](https://twitter.com/excid3)from [GoRails ](https://gorails.com)has released the first of hopefully many tutorial videos demonstrating how to get up and running with StimulusReflex in about ten minutes: ⏱️👍

![](https://gblobscdn.gitbook.com/assets%2F-Lpnm81iPOBUa9lAmLxg%2F-M6sksqaSV7fV1MX\_89U%2F-M6slxV1wY8azS1XCRxn%2Fgorails.jpg)

Introduction to Stimulus Reflex (Example) | GoRails

### &#x20;<a href="#hello-reflex-world" id="hello-reflex-world"></a>

There are two ways to enable StimulusReflex in your projects: use the `data-reflex` attribute to declare a reflex without any code, or call the `stimulate` method inside of a Stimulus controller. We can use these techniques interchangeably, and both of them trigger a server-side _"Reflex action"_ in response to users interacting with your UI.

### &#x20;<a href="#trigger-reflex-actions-with-data-reflex-attributes" id="trigger-reflex-actions-with-data-reflex-attributes"></a>

This example updates the page with the latest count when the link is clicked:

app/views/pages/index.html.erb

data-reflex="click->Counter#increment"

data-count="<%= @count.to\_i %>"

\>Increment <%= @count.to\_i %>\</a>

We use data attributes to declaratively tell StimulusReflex to pay special attention to this anchor link. The `data-reflex` attribute allows us to map an action on the client to code that will be executed on the server.

The syntax follows Stimulus format: `[DOM-event]->[ReflexClass]#[action]`

While `click` and `change` are two of the most common events used to initiate Reflex actions, you can use `mouseover`, `drop`, `play` and [any others](https://developer.mozilla.org/en-US/docs/Web/Events) that makes sense for your application.

We do caution you to be careful with events that can trigger many times in a short period such as `scroll`, `drag`, `resize` or `mousemove`. It's possible to use a [debounce strategy](broken-reference) to reduce how many events are emitted.

The other two attributes `data-step` and `data-count` are used to pass data to the server. You can think of them as arguments.

app/reflexes/counter\_reflex.rb

class CounterReflex < ApplicationReflex

@count = element.dataset\[:count].to\_i + element.dataset\[:step].to\_i

StimulusReflex maps your requests to Reflex classes that live in your `app/reflexes` folder. In this example, the `increment` action is called and the count is incremented by 1. The `@count` instance variable is passed to the template when it is re-rendered.

**Concerns like managing state and rendering views are handled server side.** Instance variables set in the Reflex action can be combined with cached fragments and potentially updated data fetched from ActiveRecord to modify the UI.

_The magic is that there is no magic_. What the user sees is exactly what they will see if they refresh the page in their browser.

StimulusReflex keeps a 1:1 relationship between application state and what is visible in the browser so that you simply don't have to manage state on the client. This translates to a massive reduction in application complexity and frees you to spend your time on features instead of state synchronization.

If you change the code in a Reflex class, you must refresh the page in your browser to interact with the new version of your code.

### &#x20;<a href="#trigger-reflex-actions-inside-stimulus-controllers" id="trigger-reflex-actions-inside-stimulus-controllers"></a>

Real-world applications will benefit from additional structure and more granular control. Building on the solid foundation that Stimulus provides, we can import StimulusReflex into our Stimulus controllers and build complex functionality.

Let's build on our increment counter example by adding a Stimulus controller and manually triggering a Reflex action by calling the `stimulate` method.

1.  1\.

    Declare the appropriate data attributes in HTML.
2.  2\.

    Create a client side StimulusReflex controller with JavaScript.
3.  3\.

    Create a server side Reflex object with Ruby.
4.  4\.

    Create a server side Example controller with Ruby.

We can use the standard Stimulus `data-controller` and `data-action` attributes, which can be [changed if you have a conflict](broken-reference). There's no StimulusReflex-specific markup required:

app/views/pages/index.html.erb

data-controller="counter"

data-action="click->counter#increment"

\>Increment <%= @count %>\</a>

Now we can create a simple Stimulus controller that extends `ApplicationController`, which is installed with StimulusReflex. It takes care of making your controller automatically inherit the `stimulate` method:

app/javascript/controllers/counter\_controller.js

import ApplicationController from './application\_controller.js'

export default class extends ApplicationController {

this.stimulate('Counter#increment', 1)

If you extend `ApplicationController` and need to create a `connect` method, make sure that the first line of your method is `super.connect()` or else you can't call `stimulate`.

When the user clicks the anchor, the Stimulus event system calls the `increment` method on our controller. In this example, we pass two parameters: the first one follows the format `[ReflexClass]#[action]` and informs the server which Reflex action in which Reflex class we want to trigger. Our second parameter is an optional argument that is passed to the Reflex action as a parameter.

If you're responding to an event like click on an element that would have a default action (such as an anchor, button or submit element) it's very important that you call `preventDefault()` on that event, or else you will experience undesirable side effects such as page navigation or form submission.

app/reflexes/counter\_reflex.rb

class CounterReflex < ApplicationReflex

session\[:count] = session\[:count].to\_i + step

Here, you can see how we accept a `step` argument to our `increment` Reflex action. We're also now switching to using the Rails session object to persist our values across multiple page load operations. Note that you can only provide parameters to Reflex actions by calling the `stimulate` method with arguments; there is no equivalent for Reflexes declared with data attributes.

app/controllers/pages\_controller.rb

class PagesController < ApplicationController

@count = session\[:count].to\_i

Finally, we set the value of the `@count` instance variable in the controller action. When the page is first loaded, there will be no `session[:count]` value and `@count` will be `nil`, which converts to an integer as 0... our initial value.

In a typical Rails app, we would set the value of `@count` after fetching it from a persistent data store such as Postgres or Redis. To keep this example simple, we use Rails' `session` to store our counter value.

### &#x20;<a href="#stimulusreflex-generator" id="stimulusreflex-generator"></a>

We provide a generator that performs a scaffold-like functionality for StimulusReflex. It will generate files and classes appropriate to whether you specify a singular or pluralized name for your reflex class. For example, `user` and `users` are both valid and useful in different situations.

bundle exec rails generate stimulus\_reflex user

This will create but not overwrite the following files:

1.  1\.

    `app/javascript/controllers/application_controller.js`
2.  2\.

    `app/javascript/controllers/user_controller.js`
3.  3\.

    `app/reflexes/application_reflex.rb`
4.  4\.

    `app/reflexes/user_reflex.rb`

If you later destroy a stimulus\_reflex "scaffold" using `bundle exec rails destroy stimulus_reflex user` your `application_reflex.rb` and `application_controller.js` will be preserved.

### &#x20;<a href="#stimulusreflex-cheatsheet" id="stimulusreflex-cheatsheet"></a>
