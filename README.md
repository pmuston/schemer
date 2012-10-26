# Mongothon

Mongothon is a MongoDB object-document mapping API for Python, loosely based on the awesome [mongoose.js](http://mongoosejs.com/) library.


# Quick Start

Mongothon allows you to declaratively express the structure and contraints of your Mongo document in a reusable Schema using Python dicts. Schemas can then be used to generate reusable Model classes which can be used in your application to perform IO with your associated Mongo collection.

## Example

Define the Mongo document structure and constraints in a Schema:
```python
car_schema = Schema({
    "make":         {"type": basestring, "required": True},
    "model":        {"type": basestring, "required": True},
    "num_wheels":   {"type": int,        "default": 4, "validates": gte(0)}
    "color":        {"type": basestring, "validates": one_of("red", "green", "blue")}
})
```

Define a virtual field on the schema:
```python
car_schema.virtual("description", getter=lambda doc: "{0} {1}".format(doc.make, doc.model))
```

Generate a reusable model class from the Schema and pymongo collection:
```python
Car = create_model(car_schema, db['car'])
```

Find, modify and save a document:
```python
car = Car.find_by_id(some_id)
car.color = "green"
car.save()
```

Create a new document:
```python
car = new Car({
    "make":     "Ford",
    "model":    "F-150",
    "color":    "red"
})
car.save()
```

Delete a document
```python
car.delete()
```

Validate a document
```python
car = new Car({
    "make":         "Ford",
    "model":        "F-150",
    "num_wheels":   -1
    "color":        "red"
})

try:
    car.validate()
except ValidationException:
    # num_wheels should be >= 0

```

# API Reference

## Schemas

### Types

Each field in a Mongothon schema must be given a type by adding a `"type"` key to the field spec dict. For example, this schema declares a single `"name"` field with a type of `basestring`:
```python
schema = Schema({"name": {"type": basestring}})
```
Supported field types are: `basestring`, `int`, `float`, `datetime`, `long`, `bool` and `Schema` (see Nested schemas below). 
If you attempt to save a model containing a value of the wrong type for a given a field a `ValidationException` will be thrown.

### Mandatory fields
You can require a field to be present in a document by adding `"required": True` to the Schema:
```python
schema = Schema({"name": {"type": basestring, "required": True}})
```
By default all fields are not required.
If `save()` is called on model which does not contain a value for a required field then the model will raise a `ValidationException`.

### Defaults
Schemas allow you to specify default values for fields which are used in the event a value is not provided in a given document.
A default can either be specified as literal:
```python
schema = Schema({"num_wheels": {"type": int, "default": 4}})
```
or as a reference to parameterless function which will be called at the point the document is saved:
```python
import datetime
schema = Schema({"created_date": {"type": datetime, "default": datetime.now}})
```

### Validation
Mongothon allows you to specify validation for a field using the `"validates"` key in the field spec. 
You can specify a single validator:
```python
schema = Schema({"color": {"type": basestring, "validates": one_of("red", "green", "blue")}})
```
or multiple validators:
```python
schema = Schema({"num_wheels": {"type": int, "validates": [gte(0), lte(6)]}})
```

#### Provided validators
Mongothon provides the following validators out-of-the-box:
```python
# Validator                         # Validates that the field...
gte(value)                          # is greater than or equal to the given value
lte(value)                          # is less than or equal to the given value
gt(value)                           # is greater than the given value
lt(value)                           # is less than the given value
between(min_value, max_value)       # is between the given min and max values
length(min_length, [max_length])    # is at least the given min length and (optionally) at most the given max length 
match(pattern)                      # matches the given regex pattern
```

#### Creating custom validators
In addition to the provided validators it's easy to create your own custom validators. 
To create a custom validator:
 - declare a function which accepts any arguments you want to provide to the validation algorithm
 - the function should itself return a function which will ultimately be called by Mongothon when validating a field value. The function should:
    - accept a single argument - the field value being validated
    - return nothing if the given value is valid
    - return a string describing the validation error if the value is invalid

Here's the declaration of an example custom validator:
```python
def startswith(prefix):
    def validate(value):
        if not value.startswith(prefix):
            return "String must start with %s" % prefix

# Usage:
schema = Schema({"full_name": {"type": basestring, "validates": startswith("Mr")}})
```

### Nested schemas

### Embedded collections

### Virtual fields

## Models

### Creating a model class

### Instance methods

### Class methods

### Middleware

#### The save document flow