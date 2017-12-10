+++
date = "2017-12-09T00:00:00+00:00"
draft = true
title = "Publishing Geoprocessing Services with SLAP"
+++

I've been working to add support in [slap](https://github.com/lobsteropteryx/slap) for publishing Geoprocessing (GP) services; the process for automating GP services isn't well documented, and there are some gotchas.  The workflow outlined here will work with both [Python Toolboxes](http://pro.arcgis.com/en/pro-app/arcpy/geoprocessing_and_python/creating-a-new-python-toolbox.htm) as well as the older, binary `.tbx` file format.

Similar to [map services](/post/docker-publishing-docker-build), it's possible to publish geoprocessing services as part of a docker build; all of the scripts, tools, and dockerfiles used in this example are available on github:

* [slap](https://github.com/lobsteropteryx/slap)
* [base docker images](https://github.com/lobsteropteryx/docker-esri)
* [test image](https://github.com/lobsteropteryx/slap-docker-test)
* [test data and scripts](https://github.com/lobsteropteryx/slap-test)

## Publishing from a result file

The [createGPSDDraft](http://pro.arcgis.com/en/pro-app/arcpy/functions/creategpsddraft.htm) method takes either a [Result](http://pro.arcgis.com/en/pro-app/arcpy/classes/result.htm) object or a path to a result file as its first argument.  I had hoped to use a results file (`.rlt`) as an artifact for publishing a GP service, similar to the way that `.mxd` files work for map services.  The problem with using result files as artifacts is that path information gets encoded in the file, and there's no relativePaths setting for a GP result the way there is for [map documents](http://desktop.arcgis.com/en/arcmap/10.3/analyze/arcpy-mapping/mapdocument-class.htm).  It's difficult to determine the cause based on the error messages--the only output is a `ValueError`, even when the `.rlt` file itself exists.

As an example, if you generate a results file on a windows machine, then try to publish it from an AGS instance running on linux, you'll see output like the following:

```bash
  File "C:\Program Files\ArcGIS\Server\arcpy\arcpy\__init__.py", line 1960, in CreateGPSDDraft
    return gp.createGPSDDraft(result, out_sddraft, service_name, server_type, connection_file_path, copy_data_to_server, folder_name, summary, tags, executionType, resultMapServer, showMessages, maximumRecords, minInstances, maxInstances, maxUsageTime, maxWaitTime, maxIdleTime)
  File "C:\Program Files\ArcGIS\Server\arcpy\arcpy\geoprocessing\_base.py", line 483, in createGPSDDraft
    return self._gp.createGPSDDraft(result, out_sddraft, service_name, server_type, connection_file_path, copy_data_to_server, folder_name, summary, tags, executionType, resultMapServer, showMessages, maximumRecords, minInstances, maxInstances, maxUsageTime, maxWaitTime, maxIdleTime)
ValueError: Z:\home\arcgis\services\test.rlt
```

Because AGS on linux runs in WINE and maps the root directory to `Z:/`, the paths in the results file no longer match, and the publishing call fails.

If we generate the results file from the same location that it will be published from on the server, we can avoid the above error.  In the example above, the `.rlt` file will be published from `Z:/home/arcgis/services`.  If we want to generate the result file from a windows machine and then publish it on a linux server, we can create a [vhd](https://technet.microsoft.com/en-us/library/gg318052(v=ws.10).aspx), assign the drive letter `Z:`, and make sure that we have a matching directory structure.  Running the tool manually from this location will give us a result file that will be compatible with the publication directory on the linux server.  We can do this once, then check in the `.rlt` file (and it's associated `.xml` metadata files) as artifacts.

![vhd-path](/images/slap-gp-1.png)

## Filling in Tool Metadata

Once we've made sure that the paths within the result file match, we get a different error:

```bash
RuntimeError: ('Analysis contained errors: ', {(u'ERROR 001243: The SlapTest/in_features parameter is missing a syntax dialog explanation in the item description', 92): [], (u'ERROR 001242: Tool Slap Test is missing item description summary', 80): []})
```

As it turns out, certain metadata is required for publication of the tool succeed; each tool in the toolbox needs to have it's summary field filled in.  You can do this from ArcMap by right-clicking the tool, then selecting "Item Description" from the context menu:

![metadata](/images/slap-gp-2.png)

 This will open a new metadata dialog, where you can edit the item summary:

![metadata-edit](/images/slap-gp-3.png)

Once the summary metadata is filled in, we're off to the races:

```bash
Publishing test.pyt
test.pyt published successfully
```

![docker-services](/images/slap-gp-4.png)

![docker-gp-success-pyt](/images/slap-gp-5.png)

![docker-gp-success-tbx](/images/slap-gp-6.png)








