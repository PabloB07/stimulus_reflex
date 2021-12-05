# Authentication

If you're just trying to bootstrap a proof-of-concept application on your local workstation, you don't technically have to worry about giving ActionCable the ability to distinguish between multiple concurrent users. However, **the moment you deploy to a host with more than one person accessing your app, you'll find that you're sharing a session and seeing other people's updates**. That isn't what most developers have in mind.

Since StimulusReflex v3.4, there is now an additional concept that you should understand - [Tab Isolation](broken-reference) - which is adjacent to but not the same as authentication. Authentication is about who sees what, while Tab Isolation is about what **you** see if you open the same thing, twice.

### &#x20;<a href="#authentication-schemes" id="authentication-schemes"></a>

### &#x20;<a href="#encrypted-session-cookies" id="encrypted-session-cookies"></a>

You can use your Rails session to isolate your users so that they don't see each other's updates. This works great even if your application doesn't have a login system.

app/channels/application\_cable/connection.rb

class Connection < ActionCable::Connection::Base

identified\_by :session\_id

self.session\_id = request.session.id

reject\_unauthorized\_connection unless session\_id

### &#x20;<a href="#current-user" id="current-user"></a>

Many Rails apps use the current\_user convention or more recently, the [Current](https://api.rubyonrails.org/classes/ActiveSupport/CurrentAttributes.html) object to provide a global user context. This gives access to the user scope from _almost_ all parts of your application.

app/controllers/application\_controller.rb

class ApplicationController < ActionController::Base

before\_action :set\_action\_cable\_identifier

def set\_action\_cable\_identifier

cookies.encrypted\[:user\_id] = current\_user&.id

app/channels/application\_cable/connection.rb

class Connection < ActionCable::Connection::Base

identified\_by :current\_user

user\_id = cookies.encrypted\[:user\_id]

return reject\_unauthorized\_connection if user\_id.nil?

user = User.find\_by(id: user\_id)

return reject\_unauthorized\_connection if user.nil?

Note that without intervention, your Reflex classes will **not** be able to see current\_user. This is easily fixed by setting `self.current_user = user` above and then delegating `current_user` to your ActionCable connection:

app/reflexes/example\_reflex.rb

class ExampleReflex < StimulusReflex::Reflex

delegate :current\_user, to: :connection

### &#x20;<a href="#devise" id="devise"></a>

If you're using the versatile [Devise](https://github.com/plataformatec/devise) authentication library, your configuration is even easier.

app/channels/application\_cable/connection.rb

class Connection < ActionCable::Connection::Base

identified\_by :current\_user

self.current\_user = find\_verified\_user

if (current\_user = env\["warden"].user)

reject\_unauthorized\_connection

If you have multiple Devise user models, you [need to specify](https://stackoverflow.com/questions/43258458/envwarden-not-working-with-rails-5) `env["warden"].user(:user)` or the variable will return `nil`.

Delegate `current_user` to the ActionCable `connection` and be home by lunch:

app/reflexes/example\_reflex.rb

class ExampleReflex < StimulusReflex::Reflex

delegate :current\_user, to: :connection

### &#x20;<a href="#sorcery" id="sorcery"></a>

If you're using [Sorcery](https://github.com/Sorcery/sorcery) for authentication, you'll need to pull the user's `id` out of the session store.

app/channels/application\_cable/connection.rb

class Connection < ActionCable::Connection::Base

identified\_by :current\_user

self.current\_user = User.find\_by(id: request.session.fetch("user\_id", nil)) || reject\_unauthorized\_connection

Now you're free to delegate `current_user` to the ActionCable `connection`.

app/reflexes/example\_reflex.rb

class ExampleReflex < ApplicationReflex

delegate :current\_user, to: :connection

### &#x20;<a href="#tokens-subscription-based" id="tokens-subscription-based"></a>

You can clone [a simple but fully functioning example application](https://github.com/leastbad/stimulus\_reflex\_harness/tree/token\_auth) based on the Stimulus Reflex Harness. It uses Devise with the `devise-jwt` gem to create a JWT token which is injected into the HEAD. You can use it as a reference for all of the instructions below.

There are scenarios where developers might wish to use JWT or some other form of authenticated programmatic access to an application using websockets. For example, you can configure a GraphQL service to accept queries over ActionCable instead of providing an URL endpoint for traditional Ajax calls. You also might need to support multiple custom domains with one ActionCable endpoint. You might also need a solution that doesn't depend on cookies, such as when you want to deploy multiple AnyCable nodes on a service like Heroku.

Your first instinct might be to authenticate in `connection.rb` using ugly hacks where you pass a token as part of your ActionCable connection URL. While this seems to make sense - after all, this is close to how the other techniques above work - **putting your token into the URL is a real security vulnerability** and there's a better way: _move the responsibility for authentication from the ActionCable connection down to the channels themselves_. Let's consider a potential solution that uses the [Warden::JWTAuth](https://github.com/waiting-for-dev/warden-jwt\_auth) module:

app/channels/application\_cable/connection.rb

class Connection < ActionCable::Connection::Base

identified\_by :current\_user

We create the `current_user` accessor as usual, but we won't be able to set it until someone successfully create a subscription to a channel. If they fail to pass a valid token, we can deny them a subscription. That means that all channels will need to be able to authenticate tokens during the subscription creation process. We will create a `subscribed` method in `ApplicationCable`, which all of your channels inherit from.

app/channels/application\_cable/channel.rb

class Channel < ActionCable::Channel::Base

attr\_accessor :current\_user

@current\_user ||= decode\_user params\[:token]

reject unless @current\_user

connection.current\_user = @current\_user

Warden::JWTAuth::UserDecoder.new.call token, :user, nil if token

In this configuration, a failure to match a token with a Warden user results in a call to `reject`. This means that while they have successfully established an ActionCable connection, they do not have the credentials to subscribe to the individual channel. Notice how we manually set the `current_user` on the connection if the authentication is successful.

In order for this scheme to work, all of your ActionCable channels - including StimulusReflex - must conform to the same validation mechanism. StimulusReflex itself will access the `ApplicationCable::Channel` definition in your application. You can set additional channels to authenticate in this manner by making sure that they inherit from `ApplicationCable::Channel` and that the `subscribed` method calls `super` before your `stream_from` or `stream_for` statement:

app/channels/test\_channel.rb

class TestChannel < ApplicationCable::Channel

app/javascript/channels/test\_channel.js

import consumer from './consumer'

consumer.subscriptions.create(

token: document.querySelector('meta\[name=action-cable-auth-token]').content

connected () { console.log('Token accepted') },

rejected () { console.log('Token rejected') }

Set a JWT token for the current user in your layout template. Note that in this example we do assume that the `warden-jwt_auth` gem is in your project (possibly through `devise-jwt`) and that there is a valid `current_user` accessor in scope.

app/controllers/application\_controller.rb

class ApplicationController < ActionController::Base

@token = Warden::JWTAuth::UserEncoder.new.call(current\_user, :user, nil).first

app/views/layout/application.html.erb

\<meta name="action-cable-auth-token" content="<%= @token %>"/>

Now, make sure that StimulusReflex is able to access the JWT token from your DOM:

app/javascript/controllers/index.js

import { Application } from 'stimulus'

import { definitionsFromContext } from 'stimulus/webpack-helpers'

import StimulusReflex from 'stimulus\_reflex'

const application = Application.start()

const context = require.context('controllers', true, /\_controller\\.js$/)

const params = { token: document.head.querySelector('meta\[name=action-cable-auth-token]').content }

application.load(definitionsFromContext(context))

StimulusReflex.initialize(application, { params })

Finally, delegate `current_user` to the ActionCable `connection` as you would in any other Reflex class:

app/reflexes/example\_reflex.rb

class ExampleReflex < ApplicationReflex

delegate :current\_user, to: :connection

### &#x20;<a href="#unauthenticated-connections" id="unauthenticated-connections"></a>

Perhaps your application doesn't have users. And maybe it doesn't even have sessions! You just want to offer all visitors access for the duration of the time that they are looking at your page. This will give every browser looking at your page a unique ActionCable connection.

app/channels/application\_cable/connection.rb

class Connection < ActionCable::Connection::Base

self.uuid = SecureRandom.urlsafe\_base64

While there is no user concept in this scenario, you can still access the visitor's uuid:

app/reflexes/example\_reflex.rb

class ExampleReflex < ApplicationReflex

delegate :uuid, to: :connection

### &#x20;<a href="#hybrid-anonymous-+-authenticated-connections" id="hybrid-anonymous-+-authenticated-connections"></a>

When you are building an application which has authenticated users, but you wish to provide Reflex-powered functionality to all users of your site, you can combine multiple authentication strategies.

Here is an ActionCable connection class based on encrypted session cookies and Devise logins:

app/channels/application\_cable/connection.rb

class Connection < ActionCable::Connection::Base

identified\_by :current\_user

identified\_by :session\_id

self.current\_user = env\["warden"].user

self.session\_id = request.session.id

reject\_unauthorized\_connection unless self.current\_user || self.session\_id

This makes use of the ability to declare multiple `identified_by` values in a single connection class. Note that you still have to delegate both `current_user` and `session_id` to the connection so you can access these values in your Reflex action methods.

This approach could make some operations more complicated, because you cannot take for granted that a connection is attached to a valid user. Please ensure that you are double-checking that all destructive mutations are properly guarded based on whatever policies you have in place.

### &#x20;<a href="#multi-tenant-applications" id="multi-tenant-applications"></a>

Use of the `acts_as_tenant` gem has skyrocketed since the excellent [JumpStart Pro](https://jumpstartrails.com) came out. It's easy to create Reflexes that automatically support tenant scopes.

While a multi-tenant tutorial is out-of-scope for this document, the basic idea of the gem is that you have a model - often `Account` - that other models get scoped to. If you have an `Image` class that `acts_as_tenant :account` then every query (read and write) to the `Image` class will automatically include a `WHERE` clause restricting results to the current `Account`.

As is so typically the case with Rails, the actual technique for bringing the Tenant to your Reflex is shorter than the explanation. Just set the current tenant to an instance of the correct class in your `Connection` module:

app/channels/application\_cable/connection.rb

class Connection < ActionCable::Connection::Base

identified\_by :current\_user

self.current\_user = env\["warden"].user

ActsAsTenant.current\_tenant = current\_user.account

A slightly more sophisticated reference application with multiple account support and a Current object is available in the `tenant` branch of the [stimulus\_reflex\_harness](https://github.com/leastbad/stimulus\_reflex\_harness/tree/tenant) repo, if you'd like to dig into this approach further.

Just because you are authenticated as a user doesn't mean you should have access to every function in the system. Sometimes you need to enforce roles and privilege levels in your Reflex classes.

The `before_reflex` callback is the best place to handle privilege checks, because you can call `throw :abort` to prevent the Reflex if the user is making decisions above their pay grade.

### &#x20;<a href="#cancancan" id="cancancan"></a>

When using [CanCanCan](https://github.com/CanCanCommunity/cancancan) (CCC) for authorization, the `accessible_by` method ensures that you only access records permitted for the current user. Depending on your requirements, you might opt to use different strategies for Page Morphs than you do for other types of Reflexes. This is because the CCC `authorize!` method is designed to operate on the current ActionController instance. StimulusReflex only creates Controller instances for Page Morphs, as they incur a performance penalty.

The first solution that you should consider is to create an Ability instance for your user in your Reflex class. This is a technique that the CCC documentation describes as "[working in a Pundit way](https://github.com/CanCanCommunity/cancancan/blob/develop/docs/Defining-Abilities:-Best-Practices.md#split-your-abilityrb-file)". While it might be a departure from how you use CCC in your Controllers, it does have the advantage of working with all Morph types and doesn't force the instantiation of an otherwise unused Controller instance:

class ClassroomsReflex < ApplicationReflex

if element.value.present?

abilties = Ability.new(current\_user)

school = School.find(element.value)

classrooms = school.classrooms.accessible\_by(abilities)

classrooms = Classroom.none

\# uncomment for a Selector Morph

\# morph "#classrooms", render(partial: "classrooms/classrooms", locals: { school: school, classrooms: classrooms })

#### &#x20;<a href="#page-morphs" id="page-morphs"></a>

Since Page Morphs create an ActionController instance to render your page template, it's possible to piggy-back on your existing Controller-based CCC logic by moving authorization calls out of your Reflex and into your Controller:

class ClassroomsReflex < ApplicationReflex

@school = element.value.present? ?

School.find(element.value) :

class ClassroomsController < ApplicationController

authorize! :index, School

authorize! :index, Classroom

@school ||= School.find(params\[:school\_id)

@schools ||= School.accessible\_by(current\_ability)

@classrooms ||= @school.present? ?

@school.classrooms.accessible\_by(current\_ability) :

While it is possible to create a solution for non-Page Morph Reflexes that involves [creating a Controller instance and delegating](https://dalezak.medium.com/using-cancancan-with-stimulusreflex-in-your-rails-app-c3d00ea0fe1b) `current_ability` to it, it's hard to justify documenting that approach here since there is already a viable, one-size-fits-all solution available and there is a performance hit when you create a Controller.

You cannot use the `authorize!` method in your Reflex action, because a Reflex is not a Controller.

### &#x20;<a href="#pundit" id="pundit"></a>

The trusty [pundit](https://github.com/varvet/pundit) gem allows you to set up policy classes that you can use to lock down Reflex action methods in a structured way. Reflexes are similar enough to controllers that if you include the `Pundit` module, you can take advantage of the `authorize` method.

Pundit expects you to have a `current_user` in scope and a policy matching the name of your Reflex action. In the following example we create a `sing?` policy for our `sing` Reflex action in `song_policy.rb`

app/policies/song\_policy.rb

class SongPolicy < ApplicationPolicy

app/reflexes/song\_reflex.rb

class SongReflex < ApplicationReflex

@song = Song.find(params\[:song\_id])

\# sing your heart out, baby!

Pundit will match your Reflex action to the right policy. If the `authorize` call fails, a `Pundit::NotAuthorizedError` will be raised, which you can handle in your Reflex action or leave unhandled so that it bubbles up and gets picked up by a 3rd-party error handling mechanism such as [Sentry](https://sentry.io) or [HoneyBadger](https://www.honeybadger.io).

app/reflexes/application\_reflex.rb

class ApplicationReflex < StimulusReflex::Reflex

rescue\_from Pundit::NotAuthorizedError do |exception|

\# handle authorization issue

If you're using Pundit to safeguard data from being accessed by bad actors and unauthorized parties - due to bugs in your code - that's probably the correct approach. _However..._ you might also want to explicitly validate policies so that you can react to them in your browser:

#### &#x20;<a href="#explicit-policy-validation" id="explicit-policy-validation"></a>

You can also ask Pundit to validate a policy explicitly and then [abort the Reflex](broken-reference) before it begins. This is an action that can be handled by the client via the **halted** life-cycle event.

The following example assumes that you have a `current_user` in scope and an `application_policy.rb` already in place. In this application, the `User` model has a boolean attribute called `admin`.

app/policies/example\_reflex\_policy.rb

class ExampleReflexPolicy < ApplicationPolicy

app/reflexes/example\_reflex.rb

class ExampleReflex < ApplicationReflex

delegate :current\_user, to: :connection

unless ExampleReflexPolicy.new(current\_user, self).test?

puts "We are authorized!"

You can even pick up this failure to thrive in a callback on your Stimulus controller:

app/javascript/controllers/example\_controller.js

import ApplicationController from './application\_controller'

export default class extends ApplicationController {

### &#x20;<a href="#passing-params-to-actioncable" id="passing-params-to-actioncable"></a>

It's common to pass key/value pairs to your ActionCable subscriptions, which show up as a `params` hash in your ActionCable Channel class. While it's usually not necessary to send extra information to the StimulusReflex Channel, it is a mechanism available to you. You might have used it to implement the token-based JWT auth technique above.

In this example, we want to tell the server whether the user has granted permission to send them native notifications. We'll then pick it up on the server:

app/javascript/controllers/index.js

import { Application } from 'stimulus'

import { definitionsFromContext } from 'stimulus/webpack-helpers'

import StimulusReflex from 'stimulus\_reflex'

import consumer from '../channels/consumer'

import controller from './application\_controller'

const application = Application.start()

const context = require.context('controllers', true, /\_controller\\.js$/)

Notification.requestPermission().then(notifications => {

params = { notifications }

application.load(definitionsFromContext(context))

StimulusReflex.initialize(application, { consumer, controller, params })

app/channels/application\_cable/channel.rb

class Channel < ActionCable::Channel::Base

attr\_accessor :notifications

@notifications = params\[:notifications]

puts @notifications # "default", "granted" or "denied"

Once you know if you can send notifications, you could consider using CableReady's [notification operation](https://cableready.stimulusreflex.com/usage/dom-operations/notifications#notification) to send updates. If they denied your request, you could use the Rails flash object instead.
