# Setup

### &#x20;<a href="#heads-up-issue-with-webpack-dev-server" id="heads-up-issue-with-webpack-dev-server"></a>

There appears to be a recent incompatibility introduced where the latest version of `webpack-dev-server` doesn't work with a new Rails 6.1 app, which will be running Webpacker `5.4.2` by default. **`5.4.3` has been released and should mitigate the issue.**

Either update `@rails/webpacker` to **5.4.3**, or lock your `devDependencies` to only use `webpack-dev-server` **v3**:

"webpack-dev-server": "\~3"

### &#x20;<a href="#command-line-install" id="command-line-install"></a>

StimulusReflex relies on [Stimulus](https://stimulusjs.org), an excellent library from the creators of Rails. You can easily install StimulusReflex to new and existing Rails 6 projects. For Rails 5.2, see [here](broken-reference).

The terminal commands below will ensure that both Stimulus and StimulusReflex are installed. It creates common files and an example to get you started. It also handles some of the configuration outlined below, **including enabling caching in your development environment**. (You can read more about why we enable caching [here](broken-reference).)

bundle add stimulus\_reflex

bundle exec rails stimulus\_reflex:install

For now, we recommend that you use **Webpacker 5.4.x**, since the 6.0 branch is still in beta and changes how things are set up.

And that's it! You can start using StimulusReflex in your application with the _development_ environment. You'll need to keep reading to set up [test](broken-reference) and [production](broken-reference).

### &#x20;<a href="#manual-configuration" id="manual-configuration"></a>

Some developers will need more control than a one-size-fits-all install task, so we're going to step through what's actually required to get up and running with StimulusReflex in your Rails 6+ project in the _development_ environment. You'll need to keep reading to set up [test](broken-reference) and [production](broken-reference). For Rails 5.2, see [here](broken-reference).

You can learn more about optimizing your Redis configuration, why we enable caching in development and why we don't currently support cookie sessions on the [Deployment](broken-reference) page.

First, the easy stuff: let's make sure we have [Stimulus ](https://stimulusjs.org)installed as part of our project's Webpack configuration. We'll also install the StimulusReflex gem and client library before enabling caching in your development environment. An initializer called `stimulus_reflex.rb` will be created with default values.

rails dev:cache # caching needs to be enabled

rake webpacker:install:stimulus

bundle add stimulus\_reflex

rails generate stimulus\_reflex:initializer

For now, we recommend that you use **Webpacker 5.4.x**, since the 6.0 branch is still in beta and changes how things are set up.

StimulusReflex happily supports both Stimulus v1.1 and v2.

We need to modify our Stimulus configuration to import and initialize StimulusReflex, which will attempt to locate the existing ActionCable consumer. A new websocket connection is created if the consumer isn't found.

app/javascript/controllers/index.js

import { Application } from 'stimulus'

import { definitionsFromContext } from 'stimulus/webpack-helpers'

import StimulusReflex from 'stimulus\_reflex'

import consumer from '../channels/consumer'

const application = Application.start()

const context = require.context('controllers', true, /\_controller\\.js$/)

application.load(definitionsFromContext(context))

application.consumer = consumer

StimulusReflex.initialize(application, { consumer })

The installation information presented by the [StimulusJS handbook](https://stimulusjs.org/handbook/installing#using-webpack) conflicts slightly with the Rails default webpacker Stimulus installation. The handbook demonstrates requiring your controllers inside of your `application.js` pack file, while webpacker creates an `index.js` in your `app/javascript/controllers` folder. StimulusReflex assumes that you are following the Rails webpacker flow. Your application pack should simply `import 'controllers'`.

If you require your controllers in both 'application.js `and` index.js\` it's likely that your controllers will load twice, causing all sorts of strange behavior.

**Cookie-based session storage is not currently supported by StimulusReflex.**

Instead, we enable caching in the development environment so that we can assign our user session data to be managed by the cache store.

In Rails, the default cache store is the memory store. We want to change the cache store to make use of Redis:

config/environments/development.rb

Rails.application.configure do

\# CHANGE the following line; it's :memory\_store by default

config.cache\_store = :redis\_cache\_store, {url: ENV.fetch("REDIS\_URL") { "redis://localhost:6379/1" }}

\# ADD the following line; it probably doesn't exist

config.session\_store :cache\_store, key: "\_sessions\_development", compress: true, pool\_size: 5, expire\_after: 1.year

You can read more about configuring Redis on the [Deployment](broken-reference) page.

Configure ActionCable to use the Redis adapter in development mode:

url: <%= ENV.fetch("REDIS\_URL") { "redis://localhost:6379/1" } %>

channel\_prefix: your\_application\_development

You should also add the `action_cable_meta_tag`helper to your application template so that ActionCable can access important configuration settings:

app/views/layouts/application.html.erb

<%= action\_cable\_meta\_tag %>

### &#x20;<a href="#upgrading-package-versions-and-sanity" id="upgrading-package-versions-and-sanity"></a>

In the future, should you ever upgrade your version of StimulusReflex, it's very important that you always make sure your gem version and npm package versions match.

Since mismatched versions are the first step on the path to hell, by default StimulusReflex won't allow the server to start if your versions are mismatched.

If you have special needs, you can override this setting in your initializer. `:warn` will emit the same text-based warning but not prevent the server process from starting. `:ignore` will silence all mismatched version warnings, if you really just DGAF. ¯\\\_(ツ)\_/¯

config/initializers/stimulus\_reflex.rb

StimulusReflex.configure do |config|

config.on\_failed\_sanity\_checks = :warn

### &#x20;<a href="#upgrading-to-v3.4.0+" id="upgrading-to-v3.4.0+"></a>

* make sure that you update `stimulus_reflex` in **both** your Gemfile and package.json
* it's **very important** to remove any `include CableReady::Broadcaster` statements from your Reflex classes
* OPTIONAL: enable [isolation mode](broken-reference) by adding `isolate: true` to the initialize options
* OPTIONAL: generate an initializer with `rails g stimulus_reflex:config`
* OPTIONAL: `bundle remove cable_ready && yarn remove cable_ready`

### &#x20;<a href="#authentication" id="authentication"></a>

If you're just experimenting with StimulusReflex or trying to bootstrap a proof-of-concept application on your local workstation, you can actually skip this section until you're planning to deploy.

Out of the box, ActionCable doesn't give StimulusReflex the ability to distinguish between multiple concurrent users looking at the same page.

**If you deploy to a host with more than one person accessing your app, you'll find that you're sharing a session and seeing other people's updates**. That isn't what most developers have in mind!

When the time comes, it's easy to configure your application to support authenticating users by their Rails session or current\_user scope. Just check out the Authentication page and choose your own adventure.

### &#x20;<a href="#tab-isolation" id="tab-isolation"></a>

One of the most universally surprising aspects of real-time UI updates is that by default, Morph operations intended for the current user execute in all of the current user's open tabs. Since the early days of StimulusReflex, this behavior has shifted from being an interesting edge case curiosity to something many developers need to prevent. Meanwhile, others built applications that rely on it.

The solution has arrived in the form of **isolation mode**.

When engaged, isolation mode restricts Morph operations to the active tab. While technically not enabled by default, we believe that most developers will want this behavior, so the StimulusReflex installation task will prepare new applications with isolation mode enabled. Any existing applications can turn it on by passing `isolate: true`:

app/javascript/controllers/index.js

StimulusReflex.initialize(application, { consumer, controller, isolate: true })

If isolation mode is not enabled, Reflexes initiated in one tab will also be executed in all other tabs, as you will see if you have client-side logging enabled.

Keep in mind that tab isolation mode only applies when multiple tabs are open to the same URL. If your tabs are open to different URLs, Reflexes will not carry over even if isolation mode is disabled.

### &#x20;<a href="#session-storage" id="session-storage"></a>

We are strong believers in the Rails Doctrine and work very hard to prioritize convention over configuration. Unfortunately, there are some inherent limitations to the way cookies are communicated via websockets that make it difficult to use cookies for session storage in production.

We default to using the `:cache_store` for `config.session_store` (and enabling caching) in the development environment if no other option has been declared. Many developers switch to using the [redis-session-store gem](https://github.com/roidrage/redis-session-store), especially in production.

You can learn more about session storage on the Deployment page.

### &#x20;<a href="#rack-middleware-support" id="rack-middleware-support"></a>

While StimulusReflex is optimized for speed, some developers might be using Rack middleware that rewrites the URL, which could cause problems for Page Morphs.

You can add any middleware you need in your initializer:

config/initializers/stimulus\_reflex.rb

StimulusReflex.configure do |config|

config.middleware.use FirstRackMiddleware

config.middleware.use SecondRackMiddleware

Users of [Jumpstart Pro](https://jumpstartrails.com) are advised to add the `Jumpstart::AccountMiddleware` middleware if they are doing path-based multitenancy.

### &#x20;<a href="#viewcomponent-integration" id="viewcomponent-integration"></a>

There is no special process required for using [view\_component](https://github.com/github/view\_component) with StimulusReflex. If ViewComponent is setup and running properly, you're already able to use them in your Reflex-enabled views.

Many StimulusReflex + ViewComponent developers are enjoying using the [view\_component\_reflex](https://github.com/joshleblanc/view\_component\_reflex) gem, which automatically persists component state to your session between Reflexes.

### &#x20;<a href="#rails-5.2+-support" id="rails-5.2+-support"></a>

To use Rails 5.2 with StimulusReflex, you'll need the latest Action Cable package from npm: `@rails/actioncable`

1.  1\.

    Replace `actioncable` with `@rails/actioncable` in `package.json`

    *
    * `yarn add @rails/actioncable`
2.  2\.

    Replace any instance of `import Actioncable from "actioncable"` with `import { createConsumer } from "@rails/actioncable"`

    * This imports the `createConsumer` function directly
    * Previously, you might call `createConsumer()` on the `Actioncable` import: `Actioncable.createConsumer()`
    * Now, you can reference `createConsumer()` directly

There's nothing about StimulusReflex 3+ that shouldn't work fine in a Rails 5.2 app if you're willing to do a bit of manual package dependency management.

If you're having trouble with converting your Rails 5.2 app to work correctly with webpacker, you should check out "[Rails 5.2, revisited](broken-reference)" on the Troubleshooting page.

### &#x20;<a href="#polyfills-for-ie11" id="polyfills-for-ie11"></a>

If you need to provide support for older browsers, you can `yarn add @stimulus_reflex/polyfills` and include them **before** your Stimulus controllers:

app/javascript/packs/application.js

import '@stimulus\_reflex/polyfills'

### &#x20;<a href="#running-edge" id="running-edge"></a>

If you are interested in running the latest version of StimulusReflex, you can point to the `master` branch on Github:

"stimulus\_reflex": "stimulusreflex/stimulus\_reflex#master"

gem "stimulus\_reflex", github: "stimulusreflex/stimulus\_reflex", branch: "master"

Restart your server(s) and refresh your page to see the latest.

It is really important to **always make sure that your Ruby and JavaScript package versions are the same**!

### &#x20;<a href="#running-a-branch-to-test-a-github-pull-request" id="running-a-branch-to-test-a-github-pull-request"></a>

Sometimes you want to test a new feature or bugfix before it is officially merged with the `master` branch. You can adapt the "Edge" instructions and run code from anywhere.

Using [#335 - tab isolation mode v2](https://github.com/hopsoft/stimulus\_reflex/pull/335) as an example, we first need the Github username of the author and the name of their local branch associated with the PR. In this case, the answers are `leastbad` and `isolation_optional`. This is a branch on the forked copy of the main project; a pull request is just a proposal to merge the changes in this branch into the `master` branch of the main project repository.

"stimulus\_reflex": "leastbad/stimulus\_reflex#isolation\_optional"

gem "stimulus\_reflex", github: "leastbad/stimulus\_reflex", branch: "isolation\_optional"

Restart your server(s) and refresh your page to see the latest.
