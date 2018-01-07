+++
date = "2017-12-23T00:00:00+00:00"
draft = false
title = "Automating Analysis Services Tabular Projects - Part 1: Deployment"
+++

My team recently starting using Microsoft's [Tabular Model](https://docs.microsoft.com/en-us/sql/analysis-services/tabular-models/tabular-models-ssas) databases at work, as an intermediate layer between an operational data store and the end users who consume this data from [Power BI](https://powerbi.microsoft.com).  Tabular Models are an [OLAP](https://en.wikipedia.org/wiki/Online_analytical_processing) technology, providing an in-memory data cube, with measures being defined using either [MDX](https://en.wikipedia.org/wiki/MultiDimensional_eXpressions) or [DAX](https://en.wikipedia.org/wiki/Data_analysis_expressions) query languages.

As we were learning about the stack, we realized that there wasn't much documentation around automated deployment or testing of Tabular Models; the typical deployment story seemed to be "right click -> publish" from Visual Studio.

In addition, we were finding a lot of errors and regressions in the measure calculations; part of the problem was technical, due to an unfamiliar query language, but there was also difficulty in capturing and communicating the nuances of the calculations when working with business analysts.

We've managed to come up with reasonable solutions to both of these problems--we now have an automated deployment pipeline based on Jenkins, as well as a full suite of acceptance tests around our measures.  And because I work with a really outstanding [group of people](https://www.proactivedigitalinsights.com/our-shared-culture), I've been encouraged to share the work we've done.  

This post will concentrate on our deployment, and [part 2](/post/analysis-services-2-testing) will cover the testing framework.  All of the code described in these posts is open sourced, and there is a full, working example on [github](https://github.com/lobsteropteryx/tabularexample); if anyone has better solutions to these issues, we'd love to hear about them!

## System Summary

A Tabular Model consists of two main parts:  A standard SQL database, which acts as the data source for our model, and a [SQL Server Analysis Services](https://docs.microsoft.com/en-us/sql/analysis-services/analysis-services) (SSAS) instance, which will house the in-memory cube that our model project defines.  The examples here will all be using SQL Server as the data source DBMS, and it is assumed that you have a Windows machine to develop on, with a local database and Analysis Services instance up and running.  Both of these are freely available in the [developer edition](https://www.microsoft.com/en-us/sql-server/sql-server-downloads) of SQL Server.

The production instances that these scripts were designed for all run in Azure, but the same workflow should apply to on-premises databases.  Note that, as of December 2017, Analysis Services is not supported on Linux, nor in Windows containers.

## Deployment Overview

There are five main steps to a Tabular Model deployment:

* Building the model project using [MSBuild](https://github.com/Microsoft/msbuild)
* Updating the build outputs to change deployment environments (i.e., from `validation` to `production`)
* Generating a deployment script from the build outputs using the SSAS [Deployment Wizard](https://docs.microsoft.com/en-us/sql/analysis-services/multidimensional-models/deploy-model-solutions-using-the-deployment-wizard)
* Executing the deployment script against the SSAS instance 
* Refreshing the cube to pull in data

We'll tie the individual build steps together via PowerShell.

### Building the Model

The first step is simple--we just need to build our Tabular Model project.  we can do this easily from PowerShell using MSBuild; assuming we've set up a build configuration called `validation`, it looks like this:

```PowerShell
msbuild TabularExample.smproj "/p:Configuration=validation" /t:Build /p:VisualStudioVersion=14.0
```

The outputs of a successful build process look like the following:

```PowerShell
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----       12/23/2017   7:45 PM           1561 Model.asdatabase
-a----       12/23/2017   7:45 PM           1356 Model.deploymentoptions
-a----       12/23/2017   7:45 PM            808 Model.deploymenttargets
```

These outputs are [used](https://docs.microsoft.com/en-us/sql/analysis-services/multidimensional-models/deploy-model-solutions-using-the-deployment-wizard) to generate an XMLA deployment script, and we may want to change them before passing them on to the Deployment Wizard.  In our particular case, we wanted to preserve the users and roles assigned to the database across deployments, and we needed to preserve the connection string to the SQL datasource.  

Most of the settings are configurable via Visual Studio, but there are several that are not exposed via the UI, and the typical configuration management workflows don't work properly with Tabular Model projects.

In order to work around these issues and set up the configuration files the way we want, we can check them into source control, and use a Powershell script to templatize them.

### Updating Build Outputs

Once we've generated the `Model.deploymentoptions` and `Model.deploymenttargets` files once, we can create a `deploymentoptions` directory in our project, and check those files in.  After our build step, we can templatize the files in `deploymentoptions` using the PowerShell `ExpandString` command, then overwrite the generated files.  The checked-in files look like this:

```xml
<DeploymentOptions xmlns:xsd="http://www.w3.org/2001/XMLSchema" 
                   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
                   xmlns:ddl2="http://schemas.microsoft.com/analysisservices/2003/engine/2" 
                   xmlns:ddl2_2="http://schemas.microsoft.com/analysisservices/2003/engine/2/2" 
                   xmlns:ddl100_100="http://schemas.microsoft.com/analysisservices/2008/engine/100/100" 
                   xmlns:ddl200="http://schemas.microsoft.com/analysisservices/2010/engine/200" 
                   xmlns:ddl200_200="http://schemas.microsoft.com/analysisservices/2010/engine/200/200" 
                   xmlns:ddl300="http://schemas.microsoft.com/analysisservices/2011/engine/300" 
                   xmlns:ddl300_300="http://schemas.microsoft.com/analysisservices/2011/engine/300/300" 
                   xmlns:ddl400="http://schemas.microsoft.com/analysisservices/2012/engine/400" 
                   xmlns:ddl400_400="http://schemas.microsoft.com/analysisservices/2012/engine/400/400" 
                   xmlns:ddl500="http://schemas.microsoft.com/analysisservices/2013/engine/500" 
                   xmlns:ddl500_500="http://schemas.microsoft.com/analysisservices/2013/engine/500/500" 
                   xmlns:dwd="http://schemas.microsoft.com/DataWarehouse/Designer/1.0">
      <TransactionalDeployment>true</TransactionalDeployment>
      <PartitionDeployment>DeployPartitions</PartitionDeployment>
      <RoleDeployment>RetainRoles</RoleDeployment>
      <ProcessingOption>DoNotProcess</ProcessingOption>
      <ImpactAnalysisFile></ImpactAnalysisFile>
      <ConfigurationSettingsDeployment>Retain</ConfigurationSettingsDeployment>
      <OptimizationSettingsDeployment>Deploy</OptimizationSettingsDeployment>
      <WriteBackTableCreation>UseExisting</WriteBackTableCreation>
</DeploymentOptions>
```

```xml
<DeploymentTarget xmlns:xsd="http://www.w3.org/2001/XMLSchema"
                  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:ddl2="http://schemas.microsoft.com/analysisservices/2003/engine/2"
                  xmlns:ddl2_2="http://schemas.microsoft.com/analysisservices/2003/engine/2/2" xmlns:ddl100_100="http://schemas.microsoft.com/analysisservices/2008/engine/100/100"
                  xmlns:ddl200="http://schemas.microsoft.com/analysisservices/2010/engine/200" 
                  xmlns:ddl200_200="http://schemas.microsoft.com/analysisservices/2010/engine/200/200">
      <Database>$environment</Database>
      <Server>$analysisServicesServer</Server>
      <ConnectionString>DataSource=$analysisServicesServer;Timeout=0;UID=$analysisServicesUsername;Password=$analysisServicesPassword;</ConnectionString>
</DeploymentTarget>
```

Note that we're setting the `RoleDeployment` to `RetainRoles`, and `ConfigurationSettingsDeployment` to `Retain`.  This means that the users and groups, as well as the data source connection information will be persistent across deployments.  We also set `ProcessingOption` to `DoNotProcess`; this means that we can build an empty cube without needing a connection to the underlying data source.  Refreshing the cube data can take hours, so we tackle that as a [separate step](#refreshing-the-cube). 

The PowerShell script to render and overwrite the generated files looks like:

```powershell
# Copy build outputs and deployment options to deployment directory
$deploymentDir = ".\deployment"
mkdir -Force $deploymentDir
cp "bin\$environment\*.*" $deploymentDir
cp .\deploymentoptions\*.* $deploymentDir

# Update deployment targets with parameters
$template = Get-Content .\deploymentoptions\Model.deploymenttargets
$expandedTemplate = $ExecutionContext.InvokeCommand.ExpandString($template)
$expandedTemplate | Set-Content "$deploymentDir\Model.deploymenttargets"
```

Once we've massaged the configuration files, we can pass them on to the Deployment Wizard to generate our deployment script.

### Creating the Deployment Script

The Deployment Wizard can be called from PowerShell like so:

```powershell
# Create the deployment script
Microsoft.AnalysisServices.Deployment.exe "$deploymentDir\Model.asdatabase" /s:"$deploymentDir\deploy.log" /o:"$deploymentDir\deploy.xmla" | Out-Default
```

There are a couple of gotchas with the Deployment Wizard; if you're updating an existing cube and you want to preserve its users and roles, the Deployment Wizard _must_ be able to connect to the existing in-memory database--running it in disconnected mode won't work.

In addition, troubleshooting error conditions can be a bit tricky; if a problem occurs, the `Microsoft.AnalysisServices.Deployment.exe` call won't respect any `$ErrorActionPreference` settings.  That means that your XMLA script may not get generated, and you'll have to check the log file (passed into the `/s` argument) to see what went wrong.

### Executing the Deployment Script

Assuming that all went well and we got an XMLA file generated, we just have to execute it to rebuild our cube:

```powershell
# Deploy the model
$SECURE_PASSWORD = ConvertTo-SecureString $analysisServicesPassword -AsPlainText -Force
$CREDENTIAL = New-Object System.Management.Automation.PSCredential ($analysisServicesUsername, $SECURE_PASSWORD)
Invoke-ASCmd –InputFile "$workspace\$deploymentDir\deploy.xmla" -Server $analysisServicesServer -Credential $CREDENTIAL
```

Note that SSAS doesn't have an equivalent of database-level security, so the username and password here will be a Windows or Active Directory account.

### Refreshing the Cube

If everything has succeed so far, we should be able to connect to our SSAS instance and see a database called `validate`.  Currently there's no data in our cube, but we can process the cube using a bit more PowerShell:

```powershell
Invoke-ProcessASDatabase -Server $analysisServicesServer -RefreshType Full -DatabaseName "$environment" -Credential $CREDENTIAL
```

Once this is complete, we should see data in our cube.  If we set the `TransactionalDeployment` flag in our deployment settings, we can do a refresh with minimal downtime--SSAS will build a full, new cube, then swap out the old version.  This is important, since we'll have to re-process in order to pull in any new data from our data source, even if there haven't been any changes to the model.

## Promoting Changes across  Environments

For the most part, promoting the model through a series of environments is straightforward; we can't promote the same artifact through a Dev -> Test -> Prod type of workflow, but by changing the configuration files and rebuilding, we can at least deploy the same source.

One of the pain points around promotion is changing the underlying data source for different environments; by design, the Deployment Wizard will not persist any database credentials, which means that we can't transform the data source the same way we did for the other deployment settings.  There is an additional [`Model.configsettings`](https://docs.microsoft.com/en-us/sql/analysis-services/multidimensional-models/deployment-script-files-solution-deployment-config-settings) file, but any changes to the data source credentials don't get persisted into the XMLA script.

The way we worked around this issue was to create an empty cube for each environment, then set up the data source connection strings manually, once.  We can then ensure that they don't get overwritten by setting `ConfigurationSettingsDeployment` to `Retain` in our `Model.deploymentoptions` file.  Once we have the initial cube built, we can run the full deployment script as part of our CI process whenever changes are pushed to master; in Jenkins, we can [inject](https://support.cloudbees.com/hc/en-us/articles/203802500-Injecting-Secrets-into-Jenkins-Build-Jobs) the credentials, and the [script](https://github.com/lobsteropteryx/tabular-example/blob/master/deploy_model.ps1) execution looks like this:

```powershell
.\deploy_model.ps1 -workspace "$($ENV:WORKSPACE)" -environment "$($ENV:ENVIRONMENT)" -analysisServicesUsername "$($ENV:ANALYSIS_SERVICES_USER)" -analysisServicesPassword "$($ENV:ANALYSIS_SERVICES_PASSWORD)"
```

Note that that we're using compatibility level 1200; at 1400, the `ConfigurationSettingsDeployment` setting seems to be ignored, and the connection string is wiped out on each deployment.

Once we had a way to do repeatable deployments, we worked to create a test environment where we could execute our measures against a known data set for validation.  That process will be covered in the next post.





