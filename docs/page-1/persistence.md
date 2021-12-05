# Persistence

We estimate that 80% of the pain points in web development are the direct result of maintaining state on the client. Even without considering the complexity of frameworks like React, how much time have you lost to fretting about model validation, stale data, and DOM readiness over your career?

#### &#x20;<a href="#stimulusreflex-applications-dont-have-a-client-state." id="stimulusreflex-applications-dont-have-a-client-state."></a>

> \* This is _at least_ 98% true.

Imagine if you could focus almost all of your time and attention on the fun parts of web development again. Exploring the best way to implement features instead of worrying about data serialization and forgotten user flows. Smaller teams working smarter and faster, then going home on time.

Designing applications in the StimulusReflex mindset is far simpler than what we're used to, and we don't have to give up responsive client functionality to see our productivity shoot through the roof. It does, however, require some unlearning of old habits. You're about to rethink how you approach persisting the state of your application. This can be jarring at first! Even positive changes feel like work.

### &#x20;<a href="#the-life-of-a-reflex" id="the-life-of-a-reflex"></a>

When you access a page in a StimulusReflex application, you see the current state of your user interface for that URL. There is no mounting process and no fetching of JSON from an API. Your request goes through the Rails router to Action Pack where your controller renders your view template and sends HTML to the browser. This is Rails in all its server-rendered glory.

Only once the HTML page is displayed in your browser, StimulusReflex wakes up. First, it opens a websocket connection and waits for messages. Then it scans your DOM for elements with `data-reflex` attributes. Those attributes become event handlers that map to methods in Stimulus controllers. The controllers connect events in your browser to methods in your Reflex classes on the server.

In a Reflex method you can call ActiveRecord, access data from Redis or the Rails cache, and set instance variables that get picked up in your view. After the Reflex method is complete, the Rails controller's action method is called and any instance variables set are passed on to the Rails controller's action method. The controller then passes its instance variables to the view template render engine itself. In this way, a Reflex is sort of like a `before_action` callback that fires before the controller even kicks in.

We find that people learn StimulusReflex quickly when they are pushed in the right direction. The order of operations can seem fuzzy until the light bulb flicks on.

This document is here to get you to the light bulb moment quickly.

### &#x20;<a href="#instance-variables" id="instance-variables"></a>

One of the most common patterns in StimulusReflex is to pass instance variables from the Reflex method through the controller and into the view template. Ruby's `||=` (pronounced "**or equals**") operator helps us manage this hand-off:

\<div data-controller="example">

\<input type="text" data-reflex-permanent

data-reflex="input->Example#update\_value">

\<p>The value is: <%= @value %>.\</p>

When you access the index page, the value will initially be set to 0. If the user changes the value of the text input, the value is updated to reflect whatever has been typed. This is possible because the `||=` will only set the instance variable to be 0 if there hasn't already been a value set in the Reflex method.

It's good to remember that in Ruby, **nil.to\_i** will return 0. This means that even without **||=** you can safely use **@value.to\_i** in your view template, because it will default to 0 in the rendered output.

StimulusReflex doesn't need to go through the Rails routing module. This means updates are processed much faster than requests that come from typing in a URL or refreshing the page.

Of course, instance variables are aptly named: they only exist for the duration of a single request, regardless of whether that request is initiated by accessing a URL or clicking a button managed by StimulusReflex.

### &#x20;<a href="#the-stimulus_reflex-instance-variable" id="the-stimulus_reflex-instance-variable"></a>

When StimulusReflex calls your Rails controller's action method, it passes any active instance variables along with a special instance variable called `@stimulus_reflex` which is set to `true`. **You can use this variable to create an if/else block in your controller that behaves differently depending on whether it's being called within the context of a Reflex update or not.**

In this example, the user is given 3 new balls every time they refresh the page in their browser, effectively restarting the game. If the page state is updated via StimulusReflex, no new balls are allocated.

This also means that `session[:balls_left]` will be set to 3 before the initial HTML page has been rendered and transmitted.

**The first time the controller action executes is your opportunity to set up the state that StimulusReflex will later modify.**

### &#x20;<a href="#the-rails-session-object" id="the-rails-session-object"></a>

The `session` object will persist across multiple requests; indeed, you can open multiple browser tabs and they will all share the same `session.id` value on the server. See for yourself: you can create a new session using Incognito Mode or using a 2nd web browser.

We can update our earlier example to use the session object, and it will now persist across multiple browser tabs and refreshes:

session\[:value] = element\[:value]

\<div data-controller="example">

\<input type="text" data-reflex-permanent

data-reflex="input->Example#update\_value">

\<p>The value is: <%= session\[:value] %>.\</p>

In general, you should be careful not to abuse the session object in a production app. First, sometimes sessions get lost or reset when people move between devices. It's also possible to accidentally reuse the same session variable key in multiple places, resulting in confusion and a frustrating bug hunt. Don't underestimate your ability to sabotage yourself in the future!

The Rails session object is perfect for prototyping during development, before potentially moving to the Rails cache, Redis or your database to store anything important. You have the power and flexibility to decide which data is ephemeral and which needs to survive the loss of a data centre, coast or continent. 🦖 👾 🌪

Cookie-based sessions are not _currently_ supported. Be sure to use a session store such as :cache\_store or you will be sad. You can find guidance on this topic on the Setup page.

### &#x20;<a href="#the-rails-cache-store" id="the-rails-cache-store"></a>

One of the most under-appreciated modules in Rails is `ActiveSupport::Cache` which provides the underlying infrastructure for [Russian doll](https://guides.rubyonrails.org/caching\_with\_rails.html#russian-doll-caching) caching. It can be called directly and that it has [a solid offering of utility methods](https://api.rubyonrails.org/classes/ActiveSupport/Cache/Store.html) such as `increment` and `decrement` for numeric values.

The Rails cache provides a consistent interface to a key/value storage container that behind the scenes can be anything from an in-memory database or temporary files to Redis or Memcached hosted on a different machine.

You can access the Rails cache easily from anywhere in your application:

Rails.cache.fetch("clicks:#{session.id}") {0}

Behold the sexy: we're using the user's `session.id` to help us build a unique key string to look up the current value. The structure of the key is free-form, but the convention is to use colon characters to build up a namespace, terminating in a unique identifier. For example:

Rails.cache.fetch("preferences:colors:foreground:#{session.id}") {"blue"}

If no key exists, it will evaluate the block, store the value for that key and return the value in one convenient, atomic action. Bam!

If you're planning to do more than set an initial simple value for the fetch default, it's good idiomatic Ruby to move to the `do..end` form of block declaration:

Rails.cache.fetch("fortune") do

In order to use the Rails cache store in development, you'll have to run `rails dev:cache` on the command line **once**. Otherwise, if you look in `config/environments/development.rb` you'll see that your cache store will be **:null\_store**, which is exactly as reliable as it sounds.

### &#x20;<a href="#activerecord" id="activerecord"></a>

The most common and powerful persistence mechanism you'll call from a Reflex method is also the most familiar.

The Reflex class makes use of the `session.id`, the `data-id` attributes from individual Todo model instances, and the new Rails `&` [safe navigation operator](http://mitrev.net/ruby/2015/11/13/the-operator-in-ruby/) (available since Ruby 2.3) to make short work of mapping events on the client to your permanent data store.

​[todos\_controller.rb](https://github.com/stimulusreflex/stimulus\_reflex\_todomvc/blob/master/app/controllers/todos\_controller.rb) only makes a single ActiveRecord query to render the current state of the view template. Well-designed StimulusReflex applications **leave the heavy-lifting associated with state changes to the Reflex class.**

### &#x20;<a href="#redis" id="redis"></a>

If Redis is your Rails cache store, you're already one step ahead!

Depending on your application and the kind of data you're working with, [calling the Redis engine directly](https://github.com/stimulusreflex/stimulus\_reflex/tree/fbbe93e5793f8e937d2fad14ec0d28c57f383d81/docs/appendices/deployment/README.md#use-redis-as-your-cache-store) (through the `redis` gem, in tandem with the `hiredis` gem for optimal performance) from your Reflex methods allows you to work with the full suite of data structure manipulation tools that are available in response to the state change operations your users initiate.

Just remember that while data in Redis could potentially be accessed faster than data in a traditional database engine such as Postgres, it is still ephemeral and you should act as though your data could **theoretically** disappear at any time.

It is a common pattern to store the results of API calls or long-running database queries in Redis, with the assumption that you could reconstitute your Redis store from scratch later in an emergency.

If you are deploying to Heroku or seeing sessions end prematurely, check out the section on [Deployment](broken-reference).

### &#x20;<a href="#kredis" id="kredis"></a>

​[Kredis](https://github.com/rails/kredis) is an exciting new addition to the Rails family. It provides a higher-level abstraction over the low-level Redis commands, allowing developers to "interact with them as coherent objects rather than isolated procedural commands."

Kredis also adds new methods to your ActiveRecord models, allowing you to treat Redis data structures as attributes on your model.

If you're running Ruby 2.7 or later, Kredis is likely a superior option to using the Rails cache or calling the Redis gem directly.

Since Kredis can optionally use its own separate Redis database instance, you might decide to use a different key expiration strategy from a cache. Caches can safely evict the least recently used keys, whereas model attributes and ActiveJob queues should be configured to scream as loudly as possible if they run out of room for more data.
