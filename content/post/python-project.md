+++
date = "2017-07-23T00:00:00+00:00"
draft = false 
title = "Six Essentials for a Python Project"
+++

I've seen a lot of GIS developers struggle to create a good project structure when building Python applications; often there's a transition from one enormous file with a single method to a "real" software project, with modular design, well defined dependencies, and the necessary tooling.  

The goal of this post is to be a summary and short checklist; these steps can improve almost any project, and are easy to implement.  There is a wealth of great documentation about all of them; in particular, check out [The Hitchhiker's Guide to Python](http://python-guide-pt-br.readthedocs.io/en/latest/) and the [Python Packaging User's Guide](https://python-packaging-user-guide.readthedocs.io/).

Even the smallest applications can benefit from these tools and conventions, and it's rare to see any real-world project that doesn't include all of the following:

1. Package management via `pip`
1. Environmental isolation using `virtualenv`
1. Standard `.gitignore` file
1. Standard directory structure
1. `Setup.py` file to track versions and dependencies
1. Test runner and scaffolding

## 1. Pip

If you aren't using a package manager to bring in your dependencies, you're making your life harder.  Modern python versions include [pip](https://pip.pypa.io/en/stable/) by default; if you're using a python version older than 2.7.9 (that's ArcMap 10.3 and older), you'll need to [install it](https://pip.pypa.io/en/stable/installing/#installing-with-get-pip-py).

With pip, you can bring in third-party libraries automatically, without having to rely on them already having been installed; rather than reading through documentation (or the source code!) and manually hunting down `requests`, `beautiful_soup`, etc, you can bring them all in just by doing `pip install`. 

## 2. Virtualenv

GIS machines tend to get clogged up over time; a small number of staff responsible for a large number of projects is the norm, and those project tend to live a long time, requiring periodic maintenance.  Rather than installing all the dependencies of all our projects into the global `site_packages` directory, we can use [virtual environments](/post/arcpy-virtualenv) to keep a separate set of dependencies and library versions for each project we work on.

## 3. Gitignore

Pulling in the [standard .gitignore](https://github.com/github/gitignore/blob/master/Python.gitignore) file right from the start will keep a lot of junk out of your repository--`.pyc` files, your virtual environment, and any editor-specific files don't need to be checked into version control.  If you want an easy tool to manage .gitignore templates, try [getignore](/post/getignore.md).

## 4. Directory Structure

The typical python project has a very simple directory structure--a top-level directory for the source files, a `tests` directory, and a few files (`setup.py`, `.gitignore`) in the root:


```bash
├── my_project
│   ├── __init__.py
│   └── my_module.py
├── setup.py
├── .gitignore
└── tests
    ├── __init__.py
    └── test_my_module.py
```

It's usually a good idea to keep your python projects fairly flat--each .py file will act as its own module.  If you need more nested directories, don't forget to add an [__init__.py](https://docs.python.org/3/tutorial/modules.html#packages) file.


## 5. Setup

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

## 6. Test Runner

Even if you have don't have any tests around your code yet, setting up the test runner and getting into the habit of running it before checking in can be a great first step; all you need is a `tests` directory, a module with the prefix `test_`, and a simple `pip install pytest`.  

Once these are in place, you can add tests a bit at a time; you can find low-hanging fruit in your existing code--maybe you have a pure function that is easy to write tests for.  And the next time you have to add a brand-new feature, you can implement it in a separate module, and practice using [TDD](https://en.wikipedia.org/wiki/Test-driven_development).


## Working with the Project

Once we have everything in place, developing against this package is straightforward; with a few lines, a new developer can be up and running on a fresh machine:

```bash
git clone git@github.com:lobsteropteryx/testing-arcpy.git
cd testing-arcpy
virtualenv --python C:/Python27/ArcGIS10.5/python.exe --system-site-packages venv
source venv/Scripts/activate
pip install .
pip install .[test]
pytest --cov=my_project tests/
```

It's also easy to [publish](https://python-packaging-user-guide.readthedocs.io/tutorials/distributing-packages/) your code as a package, so that it can be shared between applications, departments, or even across organizations.
