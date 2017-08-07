+++
date = "2017-07-16T00:00:00+00:00"
draft = true 
title = "Refactoring an ArcPy App - Part 1"
+++

I thought it might be instructive to refactor a small ArcPy tool, in order to demonstrate some techniques and ~best~ good practices.  You can see the original state [here]().

## First Impressions

This is a single script tool, called from within a toolbox in ArcMap.  It takes in 7 parameters:

* A US State
* A 4-digit year
* A Feature Class of Regulatory Sites
* A boolean
* A workspace
* An SDE connection
* Another boolean

The parameter types and metadata are hidden within the binary `.tbx` file; we can view that from ArcMap, and we should be able to deduce what they do from the code.  The logic itself is using `arcpy.GetParameterAsText`, so we're treating all the parameters as strings.

This is a pretty sizable chunk of code, weighing in at 836 lines; the logic itself is broken out into 13 different functions--that's better than a lot of arcpy code, but still has some room for improvement, since some of those functions are pretty long.  All of the code lives in a single module.

We don't have any tests, or any real scaffolding--we'll probably start there.

## Goals

When working on software projects, I like to ask myself, "What would this look like in an ideal world?"  In this case, I'd like to see a [python toolbox](), so that the parameters, types and metadata are clearly visible to the developer.  
The logic could probably be broken out into a few, smaller module, and the functions could certainly be split out into smaller pieces.  There's no hard and fast rule, but in general, functions start feeling long to me at around 10 lines, and modules between 200-300.

I want to make it easy for a developer to pick this up and keep working with it--that means standardizing the structure as a package.  I'd also like to have test scaffolding in place, so that we can increase test coverage over time.

## A Note on Refactoring

This is production code, so it's vital that we keep it working at every step in the process; a big re-write is usually not an option, and it's almost never the **right** option in any case!

## Getting Started

The first thing we'll do is set up our repository; this particular tool came from a Subversion server, but it's often the case that arcpy scripts aren't version controlled at all.  We'll pretend that we're versioning this for the first time, and setting up a brand new repo.

```bash
mkdir site_creation
cd site_creation
git init
```

I also want to go ahead and bring in a standard `.gitignore` file for Python, as well as PyCharm:

```bash
getignore get Python Global/JetBrains >> .gitignore
```

Now I can drop my source file and toolbox into this directory, and I'm ready to make my initial commit:

```bash
$ git status 
On branch master  

Initial commit  

Untracked files:
  (use "git add <file>..." to include in what will be committed)
  
  .gitignore
  SiteCreation.tbx
  siteCreation_main.py

nothing added to commit but untracked files present (use "git add" to track)

$ git add .

$ git commit -m "Initial commit"
[master (root-commit) f06132a] Initial commit
 3 files changed, 992 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 SiteCreation.tbx
 create mode 100644 siteCreation_main.py
```
setup(
    name='site_creation',
    version='1.0.0',
    description='Creates Sites',
    url='https://github.com/gypsyMoth/site_creation',
    author='Ian Firkin',
    author_email='ian.firkin@gmail.com',
    packages=find_packages(exclude=['contrib', 'docs', 'tests']),
    install_requires=[],
    extras_require={
        'test': ['pytest', 'pytest-cov']
    },
    entry_points={
        'console_scripts': []
    },
)
We're going to go ahead and check in the binary `.tbx` file, since it's necessary for our tool to work properly.

## Adding a `setup.py` file, virtual environment, and test runner

```bash
virtualenv --python C:/Python27/ArcGIS10.5/python.exe venv
source venv/Scripts/activate
```

Now we can create a `setup.py` file to hold some basic info about our tool--it's canonical name, version, and a few dependencies:

```python
setup(
    name='site_creation',
    version='1.0.0',
    description='Creates Sites',
    url='https://github.com/gypsyMoth/site_creation',
    author='Ian Firkin',
    author_email='ian.firkin@gmail.com',
    packages=find_packages(exclude=['contrib', 'docs', 'tests']),
    install_requires=[],
    extras_require={
        'test': ['pytest', 'pytest-cov']
    },
    entry_points={
        'console_scripts': []
    },
)
```

So far we haven't really changed anything with our code, but just added a few files and directories; now we'll start making some real changes.

## Converting to a Python Toolbox

The first thing we want to do is convert the main method into a normal function; we'll call it `create_sites`, and it will be the entry point of our script.

Next we'll add a tool wrapper module
