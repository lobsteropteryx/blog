+++
date = "2017-12-09T00:00:00+00:00"
draft = true
title = "Publishing Geoprocessing Services with SLAP"
+++

I've been working to add support in [slap](https://github.com/lobsteropteryx/slap) for publishing Geoprocessing (GP) services; it's possible to get this to work, but the automation story for GP services is still pretty bad.  This is a summary of some gotchas, and one potential path forward.

## Publishing from a result file

The [createGPSDDraft](http://pro.arcgis.com/en/pro-app/arcpy/functions/creategpsddraft.htm) method takes either a [Result](http://pro.arcgis.com/en/pro-app/arcpy/classes/result.htm) object or a path to a result file as its first argument.  I had hoped to use a results file (`.rlt`) as an artifact for publishing a GP service, similar to the way that `.mxd` files work for map services.  The problem with using result files as artifacts is that path information gets encoded in the file, and there's no [relativePaths](http://desktop.arcgis.com/en/arcmap/10.3/analyze/arcpy-mapping/mapdocument-class.htm) setting for a GP result the way there is for map documents.  It's difficult to determine the problem based on the error messages; the only output is a `ValueError`, even when the `.rlt` file itself exists.

As an example, if you generate a results file on a windows machine, then try to publish it from an AGS instance running on linux, you'll see output like the following:

```
  File "C:\Program Files\ArcGIS\Server\arcpy\arcpy\__init__.py", line 1960, in CreateGPSDDraft
    return gp.createGPSDDraft(result, out_sddraft, service_name, server_type, connection_file_path, copy_data_to_server, folder_name, summary, tags, executionType, resultMapServer, showMessages, maximumRecords, minInstances, maxInstances, maxUsageTime, maxWaitTime, maxIdleTime)
  File "C:\Program Files\ArcGIS\Server\arcpy\arcpy\geoprocessing\_base.py", line 483, in createGPSDDraft
    return self._gp.createGPSDDraft(result, out_sddraft, service_name, server_type, connection_file_path, copy_data_to_server, folder_name, summary, tags, executionType, resultMapServer, showMessages, maximumRecords, minInstances, maxInstances, maxUsageTime, maxWaitTime, maxIdleTime)
ValueError: Z:\home\arcgis\services\test.rlt
```

Because AGS on linux runs in WINE and maps the root directory to `Z:/`, the paths in the results file no longer match, and the publishing call fails.

If we generate the results file from the same location that it will be published from on the server, we can avoid the above error.  In the example above, the `.rlt` file will be published from `Z:/home/arcgis/services`.  If we want to generate the result file from a windows machine and then publish it on a linux server, we can create a [vhd](https://technet.microsoft.com/en-us/library/gg318052(v=ws.10).aspx), assign the drive letter `Z:`, and make sure that we have a matching directory structure.  Running the tool manually from this location will give us a result file that will be compatible with the publication directory on the linux server.  We can do this once, then check in the `.rlt` file (and it's associated `.xml` metadata files) as artifacts.

### Dynamically generate a results file as part of the publication process

We can also execute the GP tool as part of the publication process, save the Result object,  and use the outputs for our publication; depending on the tool, this can be a good solution, but it means that any required inputs must be available, or parameters must be defaulted.

## Publishing Python Toolboxes

[Python Toolboxes](http://pro.arcgis.com/en/pro-app/arcpy/geoprocessing_and_python/creating-a-new-python-toolbox.htm) give us a way to specify toolboxes in normal Python code, and are generally a big improvement over the older, binary `.tbx` format.  Publishing Python Toolboxes, however, presents some additional hurdles to overcome.



Doing this gives us a different error:

```
RuntimeError: ('Analysis contained errors: ', {(u'ERROR 001243: The SlapTest/in_features parameter is missing a syntax dialog explanation in the item description', 92): [], (u'ERROR 001242: Tool Slap Test is missing item description summary', 80): []})
```




