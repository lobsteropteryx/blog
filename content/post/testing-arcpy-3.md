# Setting up a Python Project

One thing I've seen a lot of GIS developers struggle with is creating a good project structure when building Python applications; often there's a transition from one enormous file with a single method to a "real" software project--modular design, well defined dependencies, etc.  

Javascript developers who use [npm] will be familiar with most of these ideas, but maybe not the tools.  So in a nutshell, here is how I would build a new Python project using arcpy today:

## Pip


## Virtual Environment

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

I generally try to keep my python projects fairly flat--each .py file will act as its own module.  If you need more nested directories, don't forget to add an [__init__.py]https://docs.python.org/3/tutorial/modules.html#packages) file.

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

`install_requires` are the dependencies our project needs to run--in this case, `arcpy` and `requests`.  `extras_require` is a list of additional dependencies used for development--our testing tools, linters, etc.

**Note:** If you're using the `--system-site-packages` flag, it's a good idea to use `--ignore-installed` when installing your project dependencies; this will ensure that you get the correct version, even if the same package is installed globally.

## Developing against the project

Once we have everything in place, developing against this package is straightforward.  With a few lines, I can be up an running on a fresh machine:

```bash
git clone git@github.com:lobsteropteryx/testing-arcpy.git
cd testing-arcpy
virtualenv --python C:/Python27/ArcGIS10.5/python.exe --system-site-packages venv
source venv/Scripts/activate
pip install --ignore-installed .
pip install --ingore-installed .[test]
pytest --cov=my_project tests/
```