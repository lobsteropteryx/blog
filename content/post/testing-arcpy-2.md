# Testing with ArcPy - Isolation and Mocking

[Test fixtures]() can help us run our each of our tests against a clean set of known data, but arcpy can still throw a few curve balls at us--there are global singletons we need to be aware of (i.e., `arcpy.env`), and many GP tools can have side effects (such as changing the current working directory) or limitations of their own (i.e., the [Project](http://pro.arcgis.com/en/pro-app/tool-reference/data-management/project.htm) and [Copy](http://pro.arcgis.com/en/pro-app/tool-reference/data-management/copy.htm) tools don't support in-memory workspaces).  In addition, just loading arcpy has a significant performance cost, and we want our tests to be as fast as possible.

We also want to be sure that we're testing **our** business logic and not arcpy's; ideally our tests will be making assertions about the state of the system after some public method has been called, and not testing the implementation details--we care about *what* the state of the system looks like, not *how* it got there.

To avoid some of the difficulties around the arcpy module, and keep focused on our own, custom, business logic, we can use a couple of techniques:  **Isolating** arcpy within our project, and **Mocking** it when we need to test some adjacent logic.

## Isolating arcpy calls

In a project of even moderate size and complexity, it's almost always worth wrapping arcpy calls, keeping them isolated to their own module(s), rather than sprinkling them throughout the codebase.  In practice, this often means that we want only a single `import arcpy` statement in our project, with the majority of our business logic in other modules.  By doing so, we can protect ourselves against future changes and decrease our iteration time substantially.

### Protection against changes 

We don't have control over the arcpy module, and there's no guarantee that the function signatures or pubic interfaces won't change the next time we upgrade ArcMap; we might be asked to port our application to use ESRI's new [Python API](https://developers.arcgis.com/python/), or even [GDAL](http://www.gdal.org/)!  By keeping our arcmap calls isolated, we minimize the number of places we need to make code changes in the future if our underlying GIS system changes.

### Iteration time

There's no getting around it:  Testing with arcpy is SLOW.  As the codebase grows and the number of tests increase, it can make sense to run a subset of the tests while you're iterating, and the full suite less frequently, before checking in.  By keeping arcpy calls isolated, we can get orders of magnitude better performance out of our tests when working in a part of the codebase that doesn't need arcpy.

By way of example, consider a small module that updates field values in a geodatabase; we'll call it `updater.py`:

```python
import arcpy

def update_field(feature_class):
    fields = ['MY_STRING_FIELD'] 
    with arcpy.da.UpdateCursor(feature_class, fields) as cursor:
        for row in cursor:
            row[0] = init_cap(row[0])
            cursor.updateRow(row)
            
def init_cap(field_value):
    return " ".join(word.capitalize() for word in field_value.split())
```

Our tests for the `init_cap` method might look like:

```python
from unittest import TestCase
from updater import init_cap

class TestInitCap(TestCase):
    def test_returns_empty_string_if_field_is_empty(self):
        expected = ''
        actual = init_cap('')
        self.assertEqual(actual, expected)
        
    def test_capitalizes_first_letter_of_single_word(self):
        expected = 'Foo'
        actual = init_cap('foo')
        self.assertEqual(actual, expected)
        
    def test_capitalizes_first_letter_of_multiple_words(self):
        expected = 'Foo Bar'
        actual = init_cap('foo bar')
        self.assertEqual(actual, expected)        

    def test_does_not_capitalize_words_that_start_with_numbers(self):
        expected = '3g'
        actual = init_cap('3g')
        self.assertEqual(actual, expected)                
```

Even without using a single arcpy call, the runtime of our tests is terrible:

```bash
$ pytest tests/test_updater.py
============================= test session starts =============================
platform win32 -- Python 2.7.12, pytest-3.1.2, py-1.4.34, pluggy-0.4.0
rootdir: C:\develop\testing-arcpy, inifile:
collected 4 items

tests\test_updater.py ....

========================== 4 passed in 14.35 seconds ==========================
```

By splitting the `init_cap` method out into its own module, we can avoid importing arcpy, and get a much better runtime:

```bash
$ pytest tests/test_updater.py
============================= test session starts =============================
platform win32 -- Python 2.7.12, pytest-3.1.2, py-1.4.34, pluggy-0.4.0
rootdir: C:\develop\testing-arcpy, inifile:
collected 4 items

tests\test_updater.py ....

========================== 4 passed in 0.03 seconds ===========================
```

That's almost 500 times faster!

## Mocking arcpy calls

Sometimes we can't help but have arcpy objects mixed in with our business logic; we may be working closely with domain objects in the GIS, (i.e. [MapDocument](http://desktop.arcgis.com/en/arcmap/latest/analyze/arcpy-mapping/mapdocument-class.htm) or [Layer](http://desktop.arcgis.com/en/arcmap/latest/analyze/arcpy-mapping/layer-class.htm) objects), and our business logic might be interacting with them directly.  In these cases, we can [mock](https://en.wikipedia.org/wiki/Mock_object) the arcpy calls and objects, to "get at" our actual business logic and test its functionality.

To mock arcpy, we can use the [patch](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.patch) decorator of python's mock class; at Python 2.7, the `mock` module needs to be installed separately:

```bash
pip install mock
```

Now we can use the `patch` decorator to mock out the arcpy module; note that the path we pass to the decorator is relative to where the arcpy module is used, as detailed in the [documentation](https://docs.python.org/3/library/unittest.mock.html#where-to-patch).  Since we're only importing arcpy from our wrapper module, the path will be `my_project.arcpy_wrapper.arcpy`: 

```python
from unittest import TestCase
from mock import patch
import my_project.arcpy_wrapper

@patch('my_project.arcpy_wrapper.arcpy')
class TestArcpyWrapper(TestCase):
  pass
```

To demonstrate how to use the mock module, we need an example. 

### Listing unique workspaces within a map document
Suppose we want to get a list of the unique workspaces used by a particular map document; perhaps we're setting up a new server instance, and we need to register the data sources that our map services will consume.  There could be SDE connections, shapefiles, and cloud-based services all within a single map, and each data source could be used by multiple layers.

We can get a list of `Layer` objects from the `MapDocument.ListLayers` method, and we'll have to ask each layer for its [workspacePath]() property to build our list.  Once we have the list of *all* workspaces for all the layers, we need to filter out the unique ones.  We'd like to be able to test the deduplication logic, and not arcpy's ability to list layers or workspaces.  In order to take the arcpy logic out of consideration, we need to mock both the `ListLayers` call, as well as any `Layer` objects that would be returned.  Our first test might look like this:


```python
from unittest import TestCase
from mock import patch, MagicMock
from arcpy_wrapper import list_workspaces_for_mxd

@patch('my_project.arcpy_helper.arcpy')
class TestListDataSources(TestCase):

    def test_list_single_data_source(self, mock_arcpy):
        data_source = 'layer1'
        
        layer = MagicMock()
        layer.supports.return_value=True
        layer.workspacePath = data_source
        
        mock_arcpy.mapping.ListLayers = MagicMock(return_value=[layer])
        mxd = {}  
        expected = [data_source]
        actual = list_workspaces_for_mxd(mxd)
        
        self.assertEqual(expected, actual)
```


We use the [MagicMock](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.MagicMock) first to create a "fake" `Layer` object, and then again to force the `arcpy.mapping.ListLayers` call to return an array of our fake `Layer`s.  We can run this test without errors, and watch our assertion fail:

```bash
$ pytest tests/test_arcpy_wrapper.py
============================= test session starts =============================
platform win32 -- Python 2.7.12, pytest-3.1.2, py-1.4.34, pluggy-0.4.0
rootdir: C:\develop\testing-arcpy, inifile:
collected 1 items

tests\test_arcpy_wrapper.py F

================================== FAILURES ===================================
______________ TestListDataSources.test_list_single_data_source _______________

self = <tests.test_arcpy_wrapper.TestListDataSources testMethod=test_list_single_data_source>
mock_arcpy = <MagicMock name='arcpy' id='107829008'>

   def test_list_single_data_source(self, mock_arcpy):
       data_source = 'layer1'

       layer = MagicMock()
       layer.supports.return_value = True
       layer.workspacePath = data_source

       mock_arcpy.mapping.ListLayers = MagicMock(return_value=[layer])
       mxd = {}
       expected = [data_source]
       actual = list_workspaces_for_mxd(mxd)

>       self.assertEqual(actual, expected)
E       AssertionError:  None != ['layer1']
tests\test_arcpy_wrapper.py:20: AssertionError
========================== 1 failed in 4.91 seconds ===========================
```

We implement our business logic:

```python
import arcpy

def list_workspaces_for_mxd(mxd):
    workspaces = []
    layers = arcpy.mapping.ListLayers(mxd)
    for layer in layers:
        if layer.supports("WORKSPACEPATH"):
            workspaces.add(layer.workspacePath)
    return workspaces
```

And the test should pass:

```bash
$ pytest tests/test_arcpy_wrapper.py
============================= test session starts =============================
platform win32 -- Python 2.7.12, pytest-3.1.2, py-1.4.34, pluggy-0.4.0
rootdir: C:\develop\testing-arcpy, inifile:
collected 1 items

tests\test_arcpy_wrapper.py .

========================== 1 passed in 4.90 seconds ===========================
```

Now we can do some refactoring, extracting the layer mocking into a helper method.  Then we add another test to remove duplicate layers from our list, which is the real logic we're trying to test:

```python
from unittest import TestCase
from mock import patch, MagicMock
from arcpy_wrapper import list_workspaces_for_mxd

@patch('my_project.arcpy_helper.arcpy')
class TestListDataSources(TestCase):

    def create_mock_layer(self, workspace):
        layer = MagicMock()
        layer.supports = MagicMock(return_value=True)
        layer.workspacePath = workspace
        return layer

    def test_list_single_data_source(self, mock_arcpy):
        data_source = 'layer1'
        layer = self.create_mock_layer(data_source)
        mock_arcpy.mapping.ListLayers = MagicMock(return_value=[layer])
        mxd = {}  
        expected = [data_source]
        actual = list_workspaces_for_mxd(mxd)
        self.assertEqual(expected, actual)

    def test_lists_unique_data_sources(self, mock_arcpy):
        data_source1 = 'layer1'
        data_source2 = 'layer2'
        layer1 = self.create_mock_layer(data_source1)
        layer2 = self.create_mock_layer(data_source2)
        mock_arcpy.mapping.ListLayers = MagicMock(return_value=[layer1, layer1, layer2])
        expected = sorted([data_source2, data_source1])
        actual = sorted(list_workspaces_for_mxd({}))
        self.assertEqual(expected, actual)
```

This will fail, since we have a duplicate layer:

```bash
$ pytest tests/test_arcpy_wrapper.py
============================= test session starts =============================
platform win32 -- Python 2.7.12, pytest-3.1.2, py-1.4.34, pluggy-0.4.0
rootdir: C:\develop\testing-arcpy, inifile:
collected 2 items

tests\test_arcpy_wrapper.py .F

================================== FAILURES ===================================
_____________ TestListDataSources.test_lists_unique_data_sources ______________

self = <tests.test_arcpy_wrapper.TestListDataSources testMethod=test_lists_unique_data_sources>
mock_arcpy = <MagicMock name='arcpy' id='106170320'>

   def test_lists_unique_data_sources(self, mock_arcpy):
       data_source1 = 'layer1'
       data_source2 = 'layer2'
       layer1 = self.create_mock_layer(data_source1)
       layer2 = self.create_mock_layer(data_source2)
       mock_arcpy.mapping.ListLayers = MagicMock(return_value=[layer1, layer1, layer2])
       expected = sorted([data_source2, data_source1])
       actual = sorted(list_workspaces_for_mxd({}))
>       self.assertEqual(expected, actual)
E       AssertionError: Lists differ: ['layer1', 'layer2'] != ['layer1', 'layer1', 'layer2']
E
E       First differing element 1:
E       'layer2'
E       'layer1'
E
E       Second list contains 1 additional elements.
E       First extra element 2:
E       'layer2'
E
E       - ['layer1', 'layer2']
E       + ['layer1', 'layer1', 'layer2']
E       ?            ++++++++++

tests\test_arcpy_wrapper.py:31: AssertionError
===================== 1 failed, 1 passed in 5.26 seconds ======================
```

We need to add something to our business logic to remove duplicates; the easiest way is by using a `set`:

```python
import arcpy

def list_workspaces_for_mxd(mxd):
    workspaces = set()
    layers = arcpy.mapping.ListLayers(mxd)
    for layer in layers:
        if layer.supports("WORKSPACEPATH"):
            workspaces.add(layer.workspacePath)
    return list(workspaces)
```

And now everything is green:

```bash
$ pytest tests/test_arcpy_wrapper.py
============================= test session starts =============================
platform win32 -- Python 2.7.12, pytest-3.1.2, py-1.4.34, pluggy-0.4.0
rootdir: C:\develop\testing-arcpy, inifile:
collected 2 items

tests\test_arcpy_wrapper.py ..

========================== 2 passed in 5.20 seconds ===========================
```

In general, we don't want to lean heavily on mocks; for cases where extensive mocking is needed, integration tests with real MXDs, etc. as fixtures may be a better alternative.  But when we have significant amounts of business logic to test, together with a few arcpy objects, mocking can be a good solution.

All of the examples shown here are available on [github](https://github.com/lobsteropteryx/testing-arcpy).