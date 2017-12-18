+++
date = "2017-12-17T00:00:00+00:00"
draft = false 
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

## Declarative Code
I argued against a test per entity.  Certainly I want _some_ tests around this logic--I probably want to run each built-in callable through, and any custom functions should have their own tests.  But executing the transform function against every entity doesn't reduce my risk, at least not enough to justify the cost of writing and maintaining those tests.  

Suppose I have entities like this:

```python
entity_one_list = [
    {
        "Name": "Bob",
        "Age": "42",
    }, 
    {
        "Name": "Jane",
        "Age": "17",
    }
]

ENTITY_ONE_CONVERTERS = defaultdict(lambda: lambda x: x)
ENTITY_ONE_CONVERTERS.update({
    'Age': int,
})

entity_two_list = [
    {
        "FirstName": "Billy",
        "LastName": "Bob",
        "NumberOfCats": "3",
    }, 
    {
        "FirstName": "Jane",
        "LastName": "Doe",
        "NumberOfCats": "0",
    }
]

ENTITY_TWO_CONVERTERS= defaultdict(lambda: lambda x: x)
ENTITY_TWO_CONVERTERS.update({
    'NumberOfCats': int
})
``` 

What am I actually verifying by putting a test around each entity?  Only that I've built the transform dictionaries correctly.  I could write that test more explicitly:

```python
import ENTITY_ONE_CONVERSIONS from somewhere

test_entity_one_conversions_are_correct():

    expected = defaultdict(lambda: lambda x: x)
    expected.update({
        'Age': int,
    })
    
    assert ENTITY_ONE_CONVERSIONS == expected
```

But that's the very definition of a implementation test, which isn't surprising--there is no behavior here to test.

## Abstraction, Functions, and Data Structures

We started off trying to abstract our data transformation, and  what we ended up with is a single function with arity 2, and the data structures that it takes as arguments.  Like any other piece of logic, I want to test the edge cases, and I'm not interested in writing tests that capture every possible combination of arguments.  It's easier to see this with a simpler transform:

```python
def transform(a, b):
    return a + b
```

What tests would I want around this code?  Assuming that `a` and `b` are integers, I probably want a test to capture the [identity](https://en.wikipedia.org/wiki/Additive_identity) operation, and a handful of other cases--two positive numbers, two negatives, a positive and a negative, etc. 

This is really the definition of abstractions--that we can logically reason about behavior without having to examine each and every case.

## Test Pyramid

One of the concerns that was raised was that we were violating the concept of the [Test Pyramid](https://martinfowler.com/bliki/TestPyramid.html); there was a desire to have additional integration tests in place, at a lower level than our acceptance tests.

In general, I view integration tests as a _sometimes_ necessary evil, and I want as few of them as possible; certainly I don't want to test each and every combination of modules.  Every additional level of integration tests I write is another place I may have to make changes in the future--so those tests need to _significantly_ reduce risk or iteration time in order to pay for themselves.

For us, the system in question was a CLI tool that inserts records to a local database, and is made up of three python modules.  Our top-level acceptance tests verify the full behavior for each entity, and the entire test suite runs locally in about 20 seconds.

That's not a system that makes me want another level of integration tests--to me, the benefits just don't outweigh the costs.  If you look at Martin Fowler's original article, many of the scenarios that he outlines just don't apply, and he even adds a footnote:

>The pyramid is based on the assumption that broad-stack tests are expensive, slow, and brittle compared to more focused tests, such as unit tests. While this is usually true, there are exceptions. If my high level tests are fast, reliable, and cheap to modify - then lower-level tests aren't needed.

I believe that treating the test pyramid as something that must be religiously followed is a mistake; the context matters immensely, and I would argue that many simple systems don't need three tiers of tests. 

From the Lean point of view, tests are a necessary waste, and they surely have a cost.  If I'm satisfied with my suite of top-level acceptance tests, I don't think it makes sense to write additional integration tests just so that I can say that my test suite is pyramid shaped.
