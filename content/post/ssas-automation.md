+++
date = "2017-11-06T00:00:00+00:00"
draft = true
title = "Automatiom SSAS Tablular Model Deployment"
+++

There are three tasks we need to accomplish:

* Build the model itself
* Generate the XMLA deployment script from the build outputs
* Execute the script to deploy the model

```powershell
param(
    [string]$environment,
    [string]$asUsername,
    [string]$asPassword
)

Import-Module -Name SqlServer
$ErrorActionPreference = "Stop"

$SECURE_PASSWORD = ConvertTo-SecureString $asPassword -AsPlainText -Force
$CREDENTIAL = New-Object System.Management.Automation.PSCredential ($asUsername, $SECURE_PASSWORD)

Invoke-ProcessASDatabase -Server asazure://easternus.asazure.windows.net/myssas -RefreshType Full -DatabaseName "model-$environment" -Credential $CREDENTIAL
```

```powershell
param(
    [string]$workspace,
    [string]$environment,
    [string]$asUsername,
    [string]$asPassword
)

Import-Module -Name SqlServer
$ErrorActionPreference = "Stop"

# Build the model
$msbuild = 'c:\Program files (x86)\MSBuild\14.0\Bin\MSBuild.exe'
& "$msbuild" SemanticLayer.smproj "/p:Configuration=$environment" /t:Build /p:VisualStudioVersion=14.0

# Copy build outputs and deployment options to deployment directory
$deploymentDir = ".\deployment"
mkdir -Force $deploymentDir
cp "bin\$environment\*.*" $deploymentDir
cp .\deploymentoptions\*.* $deploymentDir

# Update deployment targets with parameters
$template = Get-Content .\deploymentoptions\Model.deploymenttargets
$expandedTemplate = $ExecutionContext.InvokeCommand.ExpandString($template)
$expandedTemplate | Set-Content "$deploymentDir\Model.deploymenttargets"

# Update data sources with parameters
$datasource = "database-$environment"
$template = Get-Content .\deploymentoptions\Model.configsettings
$expandedTemplate = $ExecutionContext.InvokeCommand.ExpandString($template)
$expandedTemplate | Set-Content "$deploymentDir\Model.configsettings"

# Create the deployment script
Microsoft.AnalysisServices.Deployment.exe "$deploymentDir\Model.asdatabase" /s:"$deploymentDir\deploy.log" /o:"$deploymentDir\deploy.xmla" | Out-Default

# Deploy the model
$SECURE_PASSWORD = ConvertTo-SecureString $asPassword -AsPlainText -Force
$CREDENTIAL = New-Object System.Management.Automation.PSCredential ($asUsername, $SECURE_PASSWORD)
Invoke-ASCmd –InputFile "$workspace\$deploymentDir\deploy.xmla" -Server asazure://easternus.asazure.windows.net/myssas -Credential $CREDENTIAL
```
