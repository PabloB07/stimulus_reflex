# Life-cycle

class ExampleReflex < StimulusReflex::Reflex

\# will run only if the element has the step attribute, can use "unless" instead of "if" for opposite condition

before\_reflex :do\_stuff, if: proc { |reflex| reflex.element.dataset\[:step] }

\# will run only if the reflex instance has a url attribute, can use "unless" instead of "if" for opposite condition

before\_reflex :do\_stuff, if: :url

\# will run before all reflexes

\# will run before increment reflex, can use "except" instead of "only" for opposite condition

before\_reflex :do\_stuff, only: \[:increment]

\# will run around all reflexes, must have a yield in the callback

around\_reflex :do\_stuff\_around

\# will run after all reflexes

\# Example with multiple method names

before\_reflex :do\_stuff, :do\_stuff2

before\_reflex :run\_checks

throw :abort # this will prevent the Reflex from continuing
