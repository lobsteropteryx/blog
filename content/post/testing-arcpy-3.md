# Setting up a Python Project

One thing I've seen a lot of GIS developers struggle with is creating a good project structure when building Python applications; often there's a transition from one enormous file with a single method to a "real" software project--modular design, well defined dependencies, etc.  

Javascript developers who use [npm] will be familiar with most of these ideas, but maybe not the tools.  So in a nutshell, here is how I would build a new Python project using arcpy today:

## Pip


## Virtual Environment

[Virtualenv](https://virtualenv.pypa.io/en/stable/) allows you to create a repeatable, isolated environment for your project and its dependencies, without worrying about what packages and versions are installed globally on your development machine.  This is bog-standard for python projects, and is [included](https://docs.python.org/3/library/venv.html) in the standard library at 3.3.

If you're still using python 2.7, you'll need to install virtualenv separately, like so:

```bash
pip install virtualenv
```

Now we can create a virtual environment, but there are a couple of specific things we need in order to accomodate arcpy. 

First, we want to to tell `virtualenv` which version of python to use; it's often the case that we'll have multiple versions installed, especially if we've upgraded ArcGIS.  We can do that using the `--python` flag.

```bash
virtualenv --python C:/Python27/ArcGIS10.5/python.exe venv
```

In addition, `arcpy` is installed globally, and we need to have access to it in our virtual environment.  The simplest way to do that is to expose *all* globally installed packages in our virtual environment, via the `--system-site-packages` flag:

```bash
virtualenv --python C:/Python27/ArcGIS10.5/python.exe --system-site-packages venv
```

If we really need isolation we can link arcpy, along with its dependencies.  ArcMap ships with a .pth file already, which we can copy into our local virtual environment:

```bash
cp c:/Python27/ArcGIS10.5/Lib/site-packages/Desktop10.5.pth venv/Lib/site-packages/
```

The arcpy module depends on numpy; we can use a link to our global module (if you're using a windows `cmd` shell instead of gitbash, use `mklink` instead):

```bash
ln -s c:/Python27/ArcGIS10.5/Lib/site-packages/numpy venv/Lib/site-packages/numpy
```

Once the virtual environment is created, we activate it:

```bash
source venv/Scripts/activate
```

Now any additional dependencies we install will only be installed to our local virtual environment, and not pollute the global site-packages.

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