# Glossary

### &#x20;<a href="#stimulusreflex" id="stimulusreflex"></a>

The name of this project. It has a JS client and a Ruby based server component that rides along on top of Rails' ActionCable websockets framework.

### &#x20;<a href="#stimulus" id="stimulus"></a>

An incredibly simple yet powerful JS framework by the creators of Rails.

### &#x20;<a href="#a-reflex" id="a-reflex"></a>

Used to describe the full, round-trip life-cycle of a StimulusReflex operation, from client to server and back again.

### &#x20;<a href="#reflex-class" id="reflex-class"></a>

A Ruby class that inherits from `StimulusReflex::Reflex` and lives in your `app/reflexes` folder. This is where your Reflex actions are implemented.

### &#x20;<a href="#reflex-action" id="reflex-action"></a>

A method in a Reflex class, called in response to activity in the browser. It has access to several special accessors containing all of the Reflex controller element's attributes

### &#x20;<a href="#reflex-controller-element" id="reflex-controller-element"></a>

The DOM element upon which the `data-reflex` attribute is placed or the `stimulate` method is called. It often has data attributes intended to be delivered to the server during a Reflex action. It is the default element to hold the Reflex controller and by extension, emit Reflex life-cycle events. It will be treated as the top-level container for Selector Morphs that target it.

### &#x20;<a href="#reflex-controller" id="reflex-controller"></a>

A Stimulus controller that imports the StimulusReflex client library directly, or extends from `ApplicationController`. It has a `stimulate` method for triggering Reflexes and like all Stimulus controllers, it's aware of the element it is attached to - as well as any Stimulus [targets](https://stimulus.hotwire.dev/reference/targets) in its DOM hierarchy.

### &#x20;<a href="#applicationcontroller" id="applicationcontroller"></a>

The Stimulus controller created during installation which Reflex controllers can extend. It imports the StimulusReflex client and is where you might place callback methods that apply to all Reflexes

### &#x20;<a href="#morphs" id="morphs"></a>

The three ways to use StimulusReflex are Page, Selector and Nothing morphs. Page morphs are the default, and covered extensively on this page. See the [Morphs](broken-reference) page for more information.

### &#x20;<a href="#reflex-root" id="reflex-root"></a>

Page Morphs update with the `body` element as the default Reflex root. You can specify one or more elements to constrain what part of the DOM will be updated using the `data-reflex-root` attribute. See [Scoping Page Morphs](broken-reference) for more information.

### &#x20;<a href="#reflexid" id="reflexid"></a>

Every Reflex is assigned a unique UUIDv4 string to identify it. There are several ways to access this value, and you can use it to track the completion of Reflexes, effectively treating them as a transaction.

### &#x20;<a href="#operation" id="operation"></a>

A CableReady concept, operations are "things CableReady can do" such as changing the DOM or updating an element. Multiple operations of different types can be queued together for later delivery by calling `broadcast`.

### &#x20;<a href="#broadcast" id="broadcast"></a>

Operations are batched up by CableReady until a `broadcast` method is invoked, which immediately delivers all queued operations to one or multiple connected clients

### &#x20;<a href="#life-cycle-stage" id="life-cycle-stage"></a>

Each Reflex moves through a distinct set of moments, ideally _before_ -> _success_ -> _after_ -> _finalize_, although other stages such as error and halted are possible.

### &#x20;<a href="#life-cycle-event" id="life-cycle-event"></a>

At every stage, each Reflex can implement optional responses, both generally and specifically to individual Reflex actions. Developers can choose to work with DOM events, callback methods in a Stimulus controller or responding to the resolved Promise representing the Reflex.
