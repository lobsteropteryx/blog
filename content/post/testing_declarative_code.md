+++
date = "2017-12-17T00:00:00+00:00"
draft = true 
title = "Testing Declarative Code"
+++

We've had some interesting discussions on our team recently, about the level of testing required for some very declarative sections of our codebase.  I've been thinking about this subject a lot, especially after reading a recent [post](https://www.facebook.com/notes/kent-beck/unit-tests/1726369154062608/) by Kent Beck.

The fundamental problem we were trying to solve was this:  We have a Python dictionary that represents an entity--in this case, a person.  All of its values are strings:

```python
person = {
  "Name": "Bob",
  "Age": "42",
  "Rate": "2.345"
}
```

Certain keys within the dictionary need their values converted in some way--for example, `"42"` should be converted to an `int`, and `"2.345"` should be a `Decimal`, but `"Bob"` should remain a string.  We might have several of these  dictionaries in a list, but we can assume that all entries in the list have the same schema.
  
We wanted an abstraction to handle the data transforms that could be extended to handle additional entities; we settled on a solution like the following, using Python's [`defaultdict`](https://docs.python.org/2/library/collections.html#collections.defaultdict):

```python
person_entities = [
    {
        "Name": "Bob",
        "Age": "42",
        "Rate": "2.345"
    }, 
    {
        "Name": "Jane",
        "Age": "17",
        "Rate": "1.234"
    }
]

PERSON_CONVERTERS = defaultdict(lambda: lambda x: x)
PERSON_CONVERTERS.update({
    'Age': int,
    'Rate': Decimal
})

def transform_data(entities, converters):
    for entity in entities:
        yield {key: converters[key](value) for key, value in entity.items()}
```

Essentially this says, "If you have a converter defined for the key, apply it to the value; otherwise, just return the value itself."  To handle new entity types, we would define another set of converters specific to that entity.

The question that arose was:  At what level should we test this transformation code?    

We already had higher-level acceptance tests in place for each entity, which exercised the transform logic; the discussion centered around the need for lower-level unit or integration tests.  Do we want a test for each transform type (`int`, `Decimal`, etc)?  Each entity?  How much behavior is really here, and to what extent are we just testing built-in Python functions?