+++
date = "2017-07-16T00:00:00+00:00"
draft = true 
title = "Publishing Map Services as part of a Docker Build"
+++

A lot of [smart people](https://github.com/mraad/docker-arcgis) have worked to get ArcGIS Server (AGS) up and running in a docker container; it's a fast and convenient way to test applications that require an AGS instance to function.  

It's possible to streamline things even further by registering data and publishing services as part of the docker [build](https://docs.docker.com/engine/reference/commandline/build/) process; this allows you to keep your infrastucture, data, and service definitions under source control, and use those to build an image with all the necessary services included.  

You can version this image and store it your own [registry](https://docs.docker.com/registry/), allowing developers to spin up their own local instance in a few seconds.

It's possible to extend this approach with tools like [docker compose](https://docs.docker.com/compose/) to include a database server and Portal instance, but for this example, we'll use a very simple setup:

* A base AGS image with no security or Portal
* A single File Geodatabase (FGDB) containing our data
* A set of map documents sourced to the above FGDB
* A [SLAP](https://github.com/lobsteropteryx/slap) configuration file describing the map services we want to publish

## A Note about Support
ESRI does not support running ArcGIS in a container (in fact they barely support AGS running on Linux, and much of the code is actually executed in [WINE](https://www.winehq.org/)).  Running AGS in a container is obviously not recommended for production systems, but it still has some utility as a test environment.

## Base AGS Image
First you'll need to build your base AGS images, at whichever version(s) you want to use for testing; a  set of dockerfiles and scripts with a short walkthrough is available on [github](https://github.com/lobsteropteryx/docker-esri).
 
## Data and Map Documents
Our data and service definitions may change independently of the infrastructure; for example, we may want to publish the same services to multiple different AGS versions, in order to test a new upgrade.  Because of this, we want to store our data in a separate repository from our dockerfiles.

For this example, we create an FGDB containing the feature classes we want to build our services off of.  We also need a map document for each service we want to publish, with all of its layers sourced to the FGDB.

Note that when we save the map documents, we want to set the "[Store Relative Paths](http://desktop.arcgis.com/en/arcmap/latest/map/working-with-arcmap/referencing-data-in-the-map.htm)" checkbox, so that the map documents and data can travel together.

Once you have the database and map documents created, you can check them into source control; the result should look something like [this](https://github.com/lobsteropteryx/slap-test).

## SLAP Configuration File
For this example, we're using [SLAP](https://github.com/lobsteropteryx/slap) to publish our services; you can see an example config file [here](https://github.com/lobsteropteryx/slap-test/blob/master/config.json).  There are a couple of things to note--the base URL should be the name of our `docker-machine`, and we need to register our FGDB as a data source.

The example also includes the path to the trusted certificate store on our centos machine, so that we can publish over HTTPS; if you want to use HTTP and port 6080 for your base URL, this isn't necessary.
  
The config file can be checked in alongside the FGDB and map documents; together these three pieces will describe all the map services on our server.

## Creating a Custom Docker Image
Now we need to create an image to represent our AGS instance with a specific set of services on it; in order to create this artifact, we need to do a few things:

* Copy our data and map documents onto the server
* Install SLAP
* Call SLAP to create a new AGS site and publish our services


