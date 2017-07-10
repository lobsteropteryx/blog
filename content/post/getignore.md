+++
date = "2016-12-26T00:00:00+00:00"
draft = false
title = "Ignore this Post"
+++

Lately I've come across several software projects at work that have build artifacts checked into their repositories--nuget packages for .NET, minified bundles on javascript projects, and .pyc files in Python modules.

Consultants in the field of GIS often work across multiple languages, and it can be difficult to keep track of which files should be ignored by source control; luckily there is a collection of standard [.gitignore templates](https://github.com/github/gitignore).  If you find that your project has files checked in that are included in one of the templates, that's a good sign that something is amiss!

There are various tools to manage these templates; I'll demonstrate using one such tool, [getignore](https://github.com/gotgenes/getignore), to bootstrap some standard .gitignore files on a Windows machine.  Full disclosure:  I'm a contributor on this project.

## Installing the Tools
Getignore is a cross-platform tool, so you can always just grab the latest precompiled [binary](https://github.com/gotgenes/getignore/releases) for your OS, and drop it somewhere in your PATH.

On a Windows machine, an easier way to install is to use [chocolatey](https://chocolatey.org); if you aren't using this great tool, check out their [installation instructions](https://chocolatey.org/install).  Once you have chocolatey up and running, you can just open a command prompt as Administrator, and do

    choco install -y getignore

Once the package has installed, you can pull down the template(s) for a language or framework by using `getignore get <language>`.  By default the contents will be written to `stdout`, and you can use the normal [redirect operators](http://ss64.com/nt/syntax-redirection.html) (i.e., `>>`) to concatenate the output to a file.



## Global Ignore File
The global .gitignore file will be applied to all projects that you work on; this file lives in the `%USERPROFILE%` directory (something like `C:/Users/username`\), and typically contains entries for the OS and editor(s) you develop on. To ignore all outputs from Windows, PyCharm, Webstorm, and Visual Studio just do:

```shell
getignore get Global/Windows Global/JetBrains VisualStudio >> %USERPROFILE%/.gitignore
```

## Project Ignore Files
Language-specific `.gitignore` files will typically live in each git repository; to add the default ignore file to a Python project, simply navigate to the project directory and do:

```
getignore get Python >> .gitignore
```
For a javascript project, a good place to start is:

```
getignore get Node >> .gitignore
```

### Editing Ignore Files
These are just text files with a documented [format](https://git-scm.com/docs/gitignore), so the default templates can easily be customized.  For a javascript project, we may also want to ignore changes to `dist` for our build outputs, and `bower_components` if we're managing some packages using [Bower](https://bower.io/).  To add those directories, just open up the file and append

```
dist
bower_components
```

That's all there is to it!  Having proper `.gitignore` files in place means less time spent dealing with merge conflicts and versioning issues, and more time spent shipping features, so make sure they're in place for new projects, and double check existing ones.
