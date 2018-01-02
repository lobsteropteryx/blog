+++
date = "2017-12-23T00:00:00+00:00"
draft = true 
title = "Automating Analysis Services Tabular Projects - Part 2: Testing"
+++

```powershell
PS C:\develop\tabular-automation> .\deploy_model.ps1 -workspace c:/develop/tabular-automation -environment validation -analysisServicesUsername test_ssas -analysisServicesPassword test_ssas
```

```powershell
PS C:\develop\tabular-automation> .\run_validation.ps1 -databaseUsername test -databasePassword test -analysisServicesUsername test_ssas -analysisServicesPassword test_ssas
```

```powershell
PS C:\develop\tabular-automation> $env:TabularAcceptanceTestAnalysisServicesConnectionString = "Data Source=localhost;Catalog=validation;User ID=test_ssas;Password=test_ssas"
```

```powershell
PS C:\develop\tabular-automation> $env:TabularAcceptanceTestSqlConnectionString = "Server=localhost;Database=validation;User ID=test;Password=test;"
```

