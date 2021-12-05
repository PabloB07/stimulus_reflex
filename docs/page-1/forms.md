# Forms

When developers learn StimulusReflex and re-consider how they approach building reactive user experiences, one of the first questions is how to submit a form using their shiny new hammer. We recommend that you approach every requirement from [the bottom of the Rails stack and move up](broken-reference), because **form submission in Rails is already really well-designed and powerful**. UJS-powered remote forms are great, especially with the [Optimism](https://optimism.leastbad.com) gem delivering validation errors over the wire. 🦸🏽

Seriously, though: if you're thinking of replacing UJS remote forms with StimulusReflex form handling without a specific reason for doing so... just stick with Rails!

StimulusReflex gathers all of the attributes on the element that initiates a Reflex. All of this data gets packed into an object that is made available to your Reflex action method through the `element` accessor. You can even [scoop up the attributes of parent elements](broken-reference). This leaves form submission in the cold, though... doesn't it? 🥶

_Heck no!_ If a Reflex is called on a `form` element - or a **child** of that `form` element - then the data for the whole form will be serialized and made available to the Reflex action method as the `params` accessor.

`params` is an instance of `ActionController::Parameters` which you can manipulate the same way you would with a real form submitted via a normal POST action, with `require` and `permit`. This is useful for validating models and setting multiple attributes of a model at the same time, even if it hasn't yet been saved to the datastore.

### &#x20;<a href="#forms-are-not-actually-forms" id="forms-are-not-actually-forms"></a>

StimulusReflex uses `form` elements as a familiar way to group related elements together. However, a lot of newcomers get attached to the idea that the form they are serializing is being used _as_ an HTML form, which it is **not**.

* forms are usually submitted via a classic POST operation or an Ajax fetch; this is _not the case_ when working with StimulusReflex
* forms usually have action and method attributes; we recommend against them - you really **can** just use `<form></form>` with no other mechanism or configuration required
* forms often have submit buttons; when using StimulusReflex, submit buttons have no effect
* there's no reason to set up a route or controller action for a form intended for SR

It's very likely that in a future version of StimulusReflex, form serialization will be both optional and configurable to use any container element.

### &#x20;<a href="#modifying-params-before-its-sent-to-the-server" id="modifying-params-before-its-sent-to-the-server"></a>

On the client, you can modify `params` in your `beforeReflex` callback by modifying `element.reflexData[reflexId]` before it is sent to the server.

export default class extends ApplicationController {

beforeReflex(element, reflex, noop, reflexId) {

const { params } = element.reflexData\[reflexId]

element.reflexData\[reflexId].params = { ...params, foo: true, bar: false }

Or, if you prefer working with events:

document.addEventListener('stimulus-reflex:before', event => {

const reflexId = event.detail.reflexId

const { params } = event.target.reflexData\[reflexId]

event.target.reflexData\[reflexId].params = { ...params, foo: true, bar: false }

### &#x20;<a href="#disabling-params-serialization-for-a-reflex" id="disabling-params-serialization-for-a-reflex"></a>

It's possible that you want to disable form parameter serialization, and you can do this by passing `serializeForm: false` as one of the possible optional arguments to the `stimulate` method.

this.stimulate('Example#foo', { serializeForm: false })

### &#x20;<a href="#working-with-the-params-accessor-in-your-reflex-class" id="working-with-the-params-accessor-in-your-reflex-class"></a>

The `params` accessor is available to your `before_reflex` `around_reflex` and `after_reflex` callbacks in your server-side Reflex class. You are also free to add additional business logic on the client using the Reflex [life-cycle callbacks](broken-reference) in your Stimulus controllers.

The `params` accessor behaves as it does in a Rails controller, so you are free to lock it down and add nested models as you expect:

params.require(:post).permit(:name, comments\_attributes: \[:id, :\_destroy, :name])

Your `@post` object is instantiated from `params` so if model validations fail, your Post model instance is still in scope when the page re-renders. The model's `errors` collection is available in the view. 🐛

One benefit of this design is that implementing an auto-save feature becomes as simple as adding `data-reflex="change->Post#update"` to each field. Since the field is inside the parent `form` element, all inputs are automatically serialized and sent to your Reflex class.

Working with `has_many` associations? No sweat! Building a new record for a nested model requires **no JavaScript**. Your Reflex calls `@post.comments.build` and because Rails knows about the association, any re-renders populate the empty form field as normal.

Reflex actions called outside of a form will still have a `params` accessor, pointing to an empty `ActionController::Parameters` instance.

If you call a full-page update Reflex outside of a form that has unsaved data, you will lose the data in the form. You will also lose the data if you throw your laptop into a volcano. 🌋

### &#x20;<a href="#a-note-about-file-uploads" id="a-note-about-file-uploads"></a>

At the time of this writing, **forms that upload files are unsupported by StimulusReflex**. We suggest that you design your UI in such a way that files can be uploaded directly, making use of the standard Rails UJS form upload techniques. You might need to use `data-reflex-permanent` so that you don't lose UI state when a Reflex is triggered.

You can explore using Optimism for live error handling, and there are excellent tools such as [Dropzone](https://www.dropzonejs.com) which make it possible to upload multiple files, work with ActiveStorage and even upload directly to a cloud storage bucket.

As websockets is a text-based protocol that doesn't guarantee packet delivery or the order of packet arrival, it is not well-suited to uploading binary files. This is an example of a problem best solved with vanilla Rails UJS form handling and [ActiveStorage](https://guides.rubyonrails.org/active\_storage\_overview.html).

### &#x20;<a href="#resetting-a-submitted-form" id="resetting-a-submitted-form"></a>

If you handle your form with StimulusReflex, and the resulting DOM diff doesn't touch the form, you will end up with stale data in your form `<input>` fields. You're going to need to clear your form so the user can add more data.

One simple technique is to use a Stimulus controller to reset the form after the Reflex completes successfully. We'll call this controller `reflex-form` and we'll use it to set a target on the first text field, as well as an action on the submit button:

<%= form\_with(model: model, data: {controller: "reflex-form"}) do |form| %>

<%= form.text\_field :name, data: {target: "reflex-form.focus"} %>

<%= form.button data: {action: "click->reflex-form#submit"} %>

This controller will make use of the [Promise](broken-reference) returned by the `stimulate` method:

app/javascript/controllers/reflex\_form\_controller.js

import ApplicationController from './application\_controller'

export default class extends ApplicationController {

static targets = \['focus']

this.stimulate('Reflex#submit').then(() => {

// optional: set focus on the freshly cleared input

We're going to build an example of StimulusReflex form handling for an **edit** action, starting with the ActiveRecord models for a classic Post with Comments relationship:

class Post < ApplicationRecord

validates :name, presence: true

accepts\_nested\_attributes\_for :comments

class Comment< ApplicationRecord

validates :name, presence: true

You should **memoize** the resource you'll be mutating in your controller action so that the same form data is available regardless of whether an action is called from a page navigation or a Reflex update:

class PostsController < ApplicationController

@post ||= Post.find(params\[:id])

Now, let's create the markup for our form, which will submit to the `Post` Reflex with a [signed global ID](https://github.com/rails/globalid).

<%= form\_with model: @post, data: { reflex: "submit->PostReflex#submit", signed\_id: @post.to\_sgid.to\_s } do |form| %>

<% if @post.errors.any? %>

<% @post.errors.full\_messages.each do |message| %>

<%= form.text\_field :name, data: { reflex: "change->PostReflex#submit", reflex\_dataset: "combined" } %>

<%= form.fields\_for :comments, @post.comments do |comment\_form| %>

<%= comment\_form.hidden :id %>

<%= comment\_form.label :name %>

<%= comment\_form.text\_field :name, data: { reflex: "change->PostReflex#submit", reflex\_dataset: "combined" } %>

<%= link\_to "New comment", "#", data: { reflex: "click->PostReflex#build\_comment" } %>

Finally, let's configure our Reflex class. Since the `@post` object is created from the `params` in the `before_reflex` callback, users can click _New comment_ many times to get new empty comments.

class PostReflex < ApplicationReflex

@post = GlobalID::Locator.locate\_signed(element.dataset.signed\_id)

@post.assign\_attributes(post\_params)

params.require(:post).permit(:name, comments\_attributes: \[:id, :name])

Moving actions that traditionally lived in the realm of the ActionDispatch REST controller is not going to be necessary for every project - or every form! However, this functionality is a welcome tool on our belt.

### &#x20;<a href="#single-source-of-truth" id="single-source-of-truth"></a>

While stateless form submissions have technically always suffered from the "last update wins" problem, it's only in recent years that developers have created interfaces that need to respond to changing application state in real-time.

There are a few guiding principles that we adhere to when building a technology that can change the page you're on, even while you're busy working on something important. One of the biggest wins associated with keeping the web server as the single source of truth about the state of your application and its data is that you don't have to worry about the synchronization of state with the client. Whatever you see on your screen is the same thing that you would see if you hit refresh. This makes developing applications with StimulusReflex faster and significantly less complicated than equivalent solutions which make use of SPAs like React.

However, **StimulusReflex works hard to ensure morph operations will not overwrite the value of a text input or textarea element if it has active focus in your browser**. This exception is important because there's no compelling UI experience where you want to change the contents of an input element _while the user is typing into it_.

We've worked really hard to make sure that developers can update other aspects of the active text input element. For example, it's possible to change the background color or even mark the element as disabled while you're typing into it. However, all attempts to overwrite the input element's value will be silently suppressed.

If you need to filter or constrain the contents of a text input, consider using a client-side library such as [Cleave.js](https://nosir.github.io/cleave.js/) instead of trying to circumvent the Single Source of Truth mechanisms, which are there to protect your users from their fellow collaborators.

Note that this concept only applies to the active text input element. Any elements which are marked with `data-reflex-permanent` will not be **morphed** in any way.

Unfortunately, it's not possible to protect elements from being replaced with a Selector Morph that uses an `inner_html` operation. The client-side logger will show you which operation is being used, and you can [tweak the data](broken-reference) you're sending to make sure it's delivered as a `morph` operation.

Similarly, custom CableReady operations broadcast by the developer do not automatically respect `data-reflex-permanent`. You can set the `permanent_attribute_name` option for the [morph](https://cableready.stimulusreflex.com/reference/operations/dom-mutations#morph) operation directly.

### &#x20;<a href="#modifying-forms-with-morphs" id="modifying-forms-with-morphs"></a>

If you need to change form elements in your document based on user input, you will find yourself needing to re-render partials inside of your Reflex. This raises the very good question of how to access the `form` context, since it's not just a simple view helper that you can include.

You will need the controller's view context, as well as the parent resource used to create the form initially:

class FormReflex < ApplicationReflex

delegate :view\_context, to: :controller

form = ActionView::Helpers::FormBuilder.new(

:post, post, view\_context, {}

html = render(partial: "path/to/partial", locals: {form: form})

Depending on how your DOM hierarchy is set up, make sure that you're giving `morph` the HTML content required to successfully update the children of your target element. This requires that the outermost element of the supplied HTML matches the parent element:

<%= form\_with model: @post do |form| %>

<%= form.text\_field :title %>

<%= render "path/to/partial", locals: {form: form} %>

<%= form.text\_area :body, size: "60x10" %>

The partial might look something like this:

<%= form.text\_field :summary %>

Since the partial does not include the parent `div`, in order to successfully replace the contents of "form\_swap" we'll need to wrap it ourselves:

html = render(partial: "path/to/partial", locals: {form: form})

morph "#form\_swap", "\<div id='form\_swap'>#{html}\</div>"

You can learn more about why wrapping Morph replacement content is necessary [here](broken-reference).
