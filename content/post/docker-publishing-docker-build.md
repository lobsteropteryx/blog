+++
date = "2017-08-10T00:00:00+00:00"
draft = false 
title = "Publishing Map Services as Part of a Docker Build"
+++

A lot of [smart people](https://github.com/mraad/docker-arcgis) have worked to get ArcGIS Server (AGS) up and running in a docker container; it's a fast and convenient way to test applications that require an AGS instance to function.  

It's possible to streamline things even further by registering data and publishing services as part of the docker [build](https://docs.docker.com/engine/reference/commandline/build/) process; this allows you to keep your infrastucture, data, and service definitions under source control, and use those to build an image with all the necessary services included.  You can version this image and store it your own [registry](https://docs.docker.com/registry/), allowing developers to spin up a local, fully provisioned AGS instance in a few seconds.

It's possible to extend this approach with tools like [docker compose](https://docs.docker.com/compose/) to include a database server and Portal instance, but for this example, we'll use a very simple setup:

* A base AGS image with no security or Portal--we'll use AGS 10.4
* A single File Geodatabase (FGDB) containing our data
* A set of map documents sourced to the above FGDB
* A [slap](https://github.com/lobsteropteryx/slap) configuration file describing the map services we want to publish

A full working example is available on github; the code is split across three repositories for the [base images](https://github.com/lobsteropteryx/docker-esri/tree/10.4), [data](https://github.com/lobsteropteryx/slap-test) and [custom image](https://github.com/lobsteropteryx/slap-docker-test/tree/10.4).

### A Note about Support
ESRI does *not* support running AGS in a container (they only barely support running on Linux, and much of the code is actually executed in [WINE](https://www.winehq.org/)).  Running AGS in a container is obviously not recommended for production systems, but it still has some utility as a test environment.

## Base AGS Image
First you'll need to build your base AGS images, at whichever version(s) you want to use for testing; a  set of dockerfiles and scripts with a short walkthrough is available on [github](https://github.com/lobsteropteryx/docker-esri).
 
## Data and Map Documents
Our data and service definitions may change independently of the infrastructure; for example, we may want to publish the same services to multiple different AGS versions, in order to test a new upgrade.  Because of this, we want to store our data in a separate repository from our dockerfiles.

For this example, we create an FGDB containing the feature classes we want to build our services off of.  We also need a map document for each service we want to publish, with all of its layers sourced to the FGDB.

Note that when we save the map documents, we want to set the "[Store Relative Paths](http://desktop.arcgis.com/en/arcmap/latest/map/working-with-arcmap/referencing-data-in-the-map.htm)" checkbox, so that the map documents and data can travel together.

Once you have the database and map documents created, you can check them into source control; the result should look something like [this](https://github.com/lobsteropteryx/slap-test).

## SLAP Configuration File
For this example, we're using [slap](https://github.com/lobsteropteryx/slap) to publish our services; you can see an example config file [here](https://github.com/lobsteropteryx/slap-test/blob/master/config.json).  

There are a couple of things to note--the base URL should be the name of our `docker-machine`, and we need to register our FGDB as a data source.  The example also includes the path to the trusted certificate store on our centos machine, so that we can publish over HTTPS; if you want to use HTTP and port 6080 for your base URL, this isn't necessary.
  
The config file can be checked in alongside the FGDB and map documents; together these three pieces will describe all the map services on our server.

Slap isn't required to publish services; if you have custom arcpy scripts and service definition files that you're already using to automate publishing, those will work as well--you may need to adjust some of the scripting, and you'll have to figure out the site creation.

## Creating a Custom Docker Image
Now we need to create an image to represent our AGS instance with a specific set of services on it; in order to create this artifact, we need to do a few things:

* Copy our data and map documents onto the server
* Install slap
* Call slap to create a new AGS site and publish our services

We can pull our data from git using a shallow copy, since we don't need the history:

```bash
git clone --depth 1 git@github.com:lobsteropteryx/slap-test.git data
```

Making python calls in an AGS linux instance is a little complex--because arcpy and many of the AGS libraries run in WINE, we *don't* want to use the system python; instead, we need to call a shell script that sets up the WINE environment, and then calls python.  This script is included with the AGS installation, and is located at `/home/arcgis/server/tools/python`.

To install slap into the WINE version of `site_packages`, we use the `-m` argument of python, to call the pip module:

```bash
/home/arcgis/server/tools/python -m pip install slap
```

To call slap itself, we use a similar call, passing the slap cli as the module:

```bash
/home/arcgis/server/tools/python -m slap.cli publish \
  --name $HOSTNAME \
  --username=siteadmin \
  --password=51734dm1n \
  --site
```

The `--site` argument will create a default site before publishing any services, with the username and password we specify; this will only ever be called once when we build the image, so we don't need to worry about overwriting anything.

If we want to publish over HTTPS, we also need to add the default, self-signed certificate to the trusted store on the OS; to do so, we can use openssl in a simple script:

```bash
cp /etc/pki/tls/certs/ca-bundle.crt /etc/pki/tls/certs/ca-bundle-orig.crt

CERTFILE=~/sscert.pem
echo -n | \
  openssl s_client -host $HOSTNAME -port 6443 | \
  sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > $CERTFILE && \
  cat $CERTFILE >> /etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem && \
  cat $CERTFILE >> /etc/pki/ca-trust/extracted/pem/ca-bundle.trust.crt
```

If publishing over HTTP is adequate, this step is optional.

It's possible to execute these commands directly from our Dockerfile using the `RUN` command, but saving the commands as shell scripts helps keep the logic separate and allows us to test the commands outside of the container; once we have a set of scripts in place we can add them to the image using the `COPY` command and then `RUN` the scripts during the build.

## Building the Image

Once the necessary scripts and Dockerfile are created, we can call `docker build` to create the image.  As part of the build, we'll copy our data and map documents into the image, install slap, and publish the services.  If we want to publish the same services across different versions of AGS, we can simply use a different base image for our dockerfile.  Our build command will look something like this:

```bash
docker build\
  --force-rm=true\
  --no-cache\
  --ulimit nofile=65535:65535\
  --ulimit nproc=25059:25059\
  -t slap-test-10.5 .
```

The image will only need to be rebuilt if something in our data or map documents changes; once the image is created, anyone can pull it down and use it *without* needing to publish any services.

## Running the Image

Once the build is complete, we can spin up an AGS instance based on our image using `docker run`:

```bash
docker run \
    -it --rm \
    --hostname arcgis \
    --memory-swappiness=0 \
    -p 6080:6080 \
    -p 6443:6443 \
    -p 4000:4000 \
    -p 4001:4001 \
    -p 4002:4002 \
    -p 4003:4003 \
slap-test-10.5
```

We need to open some ports for AGS to work properly, but once the container is up and running, we can access it either from `localhost` (on a linux or mac machine), or from the ip address of our VM on windows.  At this point we can interact with the instance just like any other AGS server--developing and testing web applications, desktop workflows, or even server extensions.

![ags](images/docker-ags.png)
