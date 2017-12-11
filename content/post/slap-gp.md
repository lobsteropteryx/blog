+++
date = "2017-12-09T00:00:00+00:00"
draft = false
title = "Publishing Geoprocessing Services with SLAP"
+++

I've been working to add support in [slap](https://github.com/lobsteropteryx/slap) for publishing Geoprocessing (GP) services; the workflow outlined here will work with both [Python Toolboxes](http://pro.arcgis.com/en/pro-app/arcpy/geoprocessing_and_python/creating-a-new-python-toolbox.htm) as well as the older, binary `.tbx` file format.

Similar to [map services](/post/docker-publishing-docker-build), it's possible to publish geoprocessing services as part of a docker build; all of the scripts, tools, and dockerfiles used in this example are available on github:

* [slap](https://github.com/lobsteropteryx/slap)
* [base docker images](https://github.com/lobsteropteryx/docker-esri)
* [test image](https://github.com/lobsteropteryx/slap-docker-test)
* [test data and scripts](https://github.com/lobsteropteryx/slap-test)

## Publishing from a Result File

The [createGPSDDraft](http://pro.arcgis.com/en/pro-app/arcpy/functions/creategpsddraft.htm) method takes either a [Result](http://pro.arcgis.com/en/pro-app/arcpy/classes/result.htm) object or a path to a result (`.rlt`) file as its first argument.  It's possible to use a result file as an artifact for publishing a GP service, similar to the way that `.mxd` files work for map services, but there are some limitations.

The problem with using result files as artifacts is that path information gets encoded in the file, and there's no `relativePaths` setting for a GP result the way there is for [map documents](http://desktop.arcgis.com/en/arcmap/10.3/analyze/arcpy-mapping/mapdocument-class.htm).  It's difficult to determine the cause of failure based on the error messages--the only output is a `ValueError`, even when the `.rlt` file itself exists.

As an example, if you generate a result file on a windows machine, then try to publish it from an AGS instance running on linux, you'll see output like the following:

```bash
  File "C:\Program Files\ArcGIS\Server\arcpy\arcpy\__init__.py", line 1960, in CreateGPSDDraft
    return gp.createGPSDDraft(result, out_sddraft, service_name, server_type, connection_file_path, copy_data_to_server, folder_name, summary, tags, executionType, resultMapServer, showMessages, maximumRecords, minInstances, maxInstances, maxUsageTime, maxWaitTime, maxIdleTime)
  File "C:\Program Files\ArcGIS\Server\arcpy\arcpy\geoprocessing\_base.py", line 483, in createGPSDDraft
    return self._gp.createGPSDDraft(result, out_sddraft, service_name, server_type, connection_file_path, copy_data_to_server, folder_name, summary, tags, executionType, resultMapServer, showMessages, maximumRecords, minInstances, maxInstances, maxUsageTime, maxWaitTime, maxIdleTime)
ValueError: Z:\home\arcgis\services\test.rlt
```

AGS on linux runs in WINE, and maps the root directory to `Z:/`.  The result file was generated on a windows machine in `C:\ArcGIS`, and so the paths in the result file don't match, and the publishing call fails.

If we generate the result file in the same location where it will be published from, we can avoid the above error.  In the example above, we're publishing as part of a docker build, and the `.rlt` file will be published from `Z:/home/arcgis/services` on the server.

To generate the result file from a windows machine and then publish it on a linux server, we can create a [vhd](https://technet.microsoft.com/en-us/library/gg318052(v=ws.10).aspx), assign the drive letter `Z:`, and make sure that we have a matching directory structure.  Running the tool manually from this location will give us a result file that will be compatible with the publication directory on the linux server.  We can create this file once, and then use it (and it's associated `.xml` metadata files) as artifacts.

![vhd-path](/images/slap-gp-1.png)

Note that this can also be an issue when generating result files from a user directory on a workstation (i.e., `C:/Users/MyUsername/Documents/ArcGIS`)  which doesn't exist on the server.  The bottom line is, the path where the result file is generated must match the location it will be published from later.

## Filling in Tool Metadata

Once we've made sure that the paths within the result file match, we get a different error:

```bash
RuntimeError: ('Analysis contained errors: ', {(u'ERROR 001243: The SlapTest/in_features parameter is missing a syntax dialog explanation in the item description', 92): [], (u'ERROR 001242: Tool Slap Test is missing item description summary', 80): []})
```

As it turns out, certain metadata is required for publication of the tool succeed; each tool in the toolbox needs to have it's summary field filled in.  You can do this from ArcMap by right-clicking the tool, then selecting "Item Description" from the context menu:

![metadata](/images/slap-gp-2.png)

 This will open a new metadata dialog, where you can edit the item summary:

![metadata-edit](/images/slap-gp-3.png)

Once the summary metadata is filled in, we need to run the tool again to recreate our result file, and we're off to the races:

```bash
Publishing test.pyt
test.pyt published successfully
```

![docker-services](/images/slap-gp-4.png)

![docker-gp-success-pyt](/images/slap-gp-5.png)

![docker-gp-success-tbx](/images/slap-gp-6.png)








