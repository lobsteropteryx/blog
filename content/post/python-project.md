+++
date = "2017-07-04T00:00:00+00:00"
draft = true
title = "Setting up a Python Project"
+++

# Setting up a Python Project

One thing I've seen a lot of GIS developers struggle with is creating a good project structure when building Python applications; often there's a transition from one enormous file with a single method to a "real" software project, with modular design, well defined dependencies, and the necessary tooling.

There are several "big picture" goals that drive the technical choices and tools we use; in general, we'd like our projects to be **easy to develop against**, and **easy to consume**.

**Easy to develop against** means that our projects should have:

* a consistent structure
* an easy way to stand up a development environment from scratch
* an easy way to manage dependencies
* a quick way to give feedback to the developer.  This means tests!

**Easy to consume** means that we want:

* a very low barrier to entry, ideally just `pip install`
* a consistent experience across environments
* intuitive naming and patterns that follow [conventions](https://www.python.org/dev/peps/pep-0008/)

To achieve these goals, we can rely on some stock tools and conventions; [Pip](https://pip.pypa.io/en/stable/), [Virtualenv](http://python-guide-pt-br.readthedocs.io/en/latest/dev/virtualenvs/) and [Pytest] help with development, and a standard [Setup] file and structure make it easy for our users to consume the tools that we build.

Many developers may be familiar with these ideas, but maybe not the tools; python is often somewhat of a second-class citizen in the GIS world, and we get away with things that we would never even consider doing in a .NET or javascript project.   These are tools and techniques that even the smallest python projects can benefit from; outside of the GIS world, they would be considered standard for almost every python application. 

## Pip

If you aren't using a package manager to bring in your dependencies, you're making your life harder.  Modern python versions include [pip](https://pip.pypa.io/en/stable/) by default; if you're using a python version older than 2.7.9 (that's ArcMap 10.3 and older), you'll need to [install it](https://pip.pypa.io/en/stable/installing/#installing-with-get-pip-py).

With pip, you can bring in third-party libraries automatically, without having to rely on them already having been installed.  Rather than reading through documentation (or the source code!) and manually hunting down `requests`, `beautiful_soup`, etc.

## Virtualenv

GIS machines tend to get clogged up over time; a small number of staff responsible for a large number of projects is the norm, and those project tend to live a long time, requiring periodic maintenance.  Rather than installing all the dependencies of all our projects into the global `site_packages` directory, we can use [virtual environments](/post/arcpy-virtualenv) to keep a separate set of dependencies and library versions for each project we work on.

## Gitignore

Pulling in the [standard .gitignore](https://github.com/github/gitignore/blob/master/Python.gitignore) file right from the start will keep a lot of junk out of your repository--`.pyc` files, your virtual environment, and any editor-specific files don't need to be checked into version control.  If you want an easy tool to manage .gitignore templates, try [getignore](getignore.md).

## Directory Structure

The typical python project has a very simple directory structure--a top-level directory for the source files, a `tests` directory, and a few files (`setup.py`, `.gitignore`) in the root:


```bash
├── my_project
│   ├── __init__.py
│   └── my_module.py
├── setup.py
└── tests
    ├── __init__.py
    └── test_my_module.py
```

I generally try to keep my python projects fairly flat--each .py file will act as its own module.  If you need more nested directories, don't forget to add an [__init__.py](https://docs.python.org/3/tutorial/modules.html#packages) file.

## Setup

The [setup.py](https://docs.python.org/2/distutils/setupscript.html) file will define a few key pieces of information about our package--its name, version, and dependencies; if you're a javascript developer coming to python, `setup.py` is analogous to `package.json`.  A very simple setup.py might look like:

```python
from setuptools import setup, find_packages

setup(
    name='my_project',
    version='1.0.0',
    description='Sample project for arcpy applications',
    url='https://github.com/lobsteropteryx/arcpy-testing',
    author='Ian Firkin',
    author_email='ian.firkin@gmail.com',
    packages=find_packages(exclude=['contrib', 'docs', 'tests']),
    install_requires=['requests'],
    extras_require={
        'test': ['pytest', 'pytest-cov', 'pylint']
    },
    entry_points={
        'console_scripts': []
    },
)
```

`install_requires` are the dependencies our project needs to run--in this case, we're using `requests`.  `extras_require` is a list of additional dependencies used for development--our testing tools, linters, etc.

## Developing against the project

Once we have everything in place, developing against this package is straightforward.  With a few lines, I can be up an running on a fresh machine:

```bash
git clone git@github.com:lobsteropteryx/testing-arcpy.git
cd testing-arcpy
virtualenv --python C:/Python27/ArcGIS10.5/python.exe --system-site-packages venv
source venv/Scripts/activate
pip install .
pip install .[test]
pytest --cov=my_project tests/
```

## Consuming the project

With the `setup.py` file in place, it's easy to [publish]()
