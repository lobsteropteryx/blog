+++
date = "2017-07-16T00:00:00+00:00"
draft = false
title = "Using Virtualenv with ArcPy"
+++

[Virtualenv](https://virtualenv.pypa.io/en/stable/) allows you to create a repeatable, isolated environment for your project and its dependencies, without worrying about what packages and versions are installed globally on your development machine.  This is a standard tool for most python projects, but since arcpy is installed as a separate, global package, using virtual environments is a little more difficult.

There are a couple of approaches to tackling this problem; either adding a [.pth file](https://my.usgs.gov/confluence/display/cdi/Calling+arcpy+from+an+external+virtual+Python+environment) to the local virtualenv, or by using the `--system-site-packages` flag.  Both of these have some disadvantages--the first requires manual modification of the virtual environment and requires storing the `.pth` file separately, and the second brings in all the global dependencies into each virtual environment.

A third possibility is to use the [site](https://docs.python.org/2.7/library/site.html) module, which will allow us to selectively expose global packages to all of our virtual environments; this is the approach we'll take.

## Installing

Virtualenv is [included](https://docs.python.org/3/library/venv.html) in the standard library at 3.3, but if you're still using python 2.7, you'll need to install it separately:

```bash
pip install virtualenv
```

## Customizing Site Packages

Since `arcpy` is installed globally and isn't available as an independent package, we need to allow access to it in our virtual environment.  We can do this by creating a `sitecustomize.py` file; this will allow us to specify a subset of the globally installed packages, and that subset will be included in every virtual environment we create.   At a minimum, we need arcpy and numpy, but you can add additional packages if you need to.

First we'll create a directory to hold our subset; it doesn't matter where we put it or what its called, but for this example, it will be `C:/Python27/ARcGIS10.5/arcpy_includes`.

We can add the required arcpy directories by copying the global `Desktop10.5.pth` file; this is just a list of directories for python to search, and it looks like this:

```bash
C:\Program Files (x86)\ArcGIS\Desktop10.5\bin
C:\Program Files (x86)\ArcGIS\Desktop10.5\ArcPy
C:\Program Files (x86)\ArcGIS\Desktop10.5\ArcToolBox\Scripts
```

To handle numpy, we create a link (`mklink` from a windows `cmd` shell, `ln` from gitbash) to the global directory; putting it all together, it looks like this:

```bash
cd c:/Python27/ArcGIS10.5
mkdir arcpy_includes
cd arcpy_includes
cp c:/Python27/ArcGIS10.5/Lib/site-packages/Desktop10.5.pth ./ 
ln -s c:/Python27/ArcGIS10.5/Lib/site-packages/numpy numpy 
```

Once this is done, our `arcpy_includes` directory should look like:

```bash
$ ls
Desktop10.5.pth  numpy/
```

Now we create a file called `sitecustomize.py`, in `C:/Python27/ArcGIS10.5/Lib` (note that the documentation recommends placing `sitecustomize.py` in the global `Lib/site-packages` directory, but virtualenv won't be able to find it there).  The contents of the file are short:

```python
import site

site.addsitedir('c:/Python27/ArcGIS10.5/arcpy_includes')
```

And that's it--now arcpy and numpy will be included in every virtual environment we create.

## Using the Virtual Environment

To use the virtual environment in a project, just navigate to your project directory; the first time around, we need to create the virtual environment--by convention, we'll put in a local folder called `venv`.  We may also want to to tell `virtualenv` which version of python to use; it's often the case that we'll have multiple versions installed, especially if we've upgraded ArcGIS.  We can do that using the `--python` flag:

```bash
virtualenv --python C:/Python27/ArcGIS10.5/python.exe venv
```

Once the virtual environment is created, we activate it (from a `cmd` shell, you don't need the `source` command):

```bash
source venv/Scripts/activate
```

Now any additional dependencies we install will only be installed to our local virtual environment, and not pollute the global site-packages.  To deactivate the environment, just do:

```bash
deactivate
```
