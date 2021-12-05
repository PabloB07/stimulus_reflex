# Reflex Classes

class ExampleReflex < ApplicationReflex

element.id # => "example"

element\[:id] # => "example"

element\["id"] # => "example"

element.value # => "on" (checkbox is always "on", use checked)

element.values # => nil, or Array for multiple values

element\[:tag\_name] # => "CHECKBOX"

element\[:checked] # => true

element\["checked"] # => true

element.checked # => true

element.label # => "Example"

element.data\_reflex # => "change->Example#accessors"

element\["data-reflex"] # => "change->Example#accessors"

element.dataset\[:reflex] # => "change->Example#accessors"

element.dataset\["reflex"] # => "change->Example#accessors"

element.dataset.value # => "123"

element.data\_value # => "123"

element\["data-value"] # => "123"

element.dataset\[:value] # => "123"

element.dataset\["value"] # => "123"
