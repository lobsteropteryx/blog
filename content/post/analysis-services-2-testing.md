+++
date = "2017-12-23T00:00:00+00:00"
draft = true 
title = "Automating Analysis Services Tabular Projects - Part 2: Testing"
+++

As part of my [day job](https://www.proactivedigitalinsights.com/), we've been building out a process for automation of a Microsoft [Tabular Model](https://docs.microsoft.com/en-us/sql/analysis-services/tabular-models/tabular-models-ssas) project; in a [previous post](/post/analysis-services-1-deployment) I described how we automated the deployment process, and this post will focus on testing.  A full, working sample project is available on [GitHub](https://github.com/lobsteropteryx/tabular-example).

## The Problem: Testing Measures

In SQL terms, a [measure](https://docs.microsoft.com/en-us/sql/analysis-services/tabular-models/measures-ssas-tabular) is somewhere between a view and an analytic function; it's a calculation used to dynamically aggregate and filter report tables, and it can contain a fair bit of logic.  Measures can also depend on other measures, and they're often used to encapsulation logic, even if they won't be exposed directly to the end user.

In our case, we needed to work closely with business analysts to define the measures in business terms, ensure that they were implemented correctly, and protect ourselves from regressions as we build out additional reporting.  We knew from the start that we wanted a BDD-style approach using [gherkin](https://github.com/cucumber/cucumber/wiki/Gherkin), and we settled on [SpecFlow](http://specflow.org/) as our test framework.

## Testing Process

There are six steps that need to be performed for each test scenario:

* Ensure that the database is in a clean (empty) state
* Load a small, known data set into the proper database tables
* Build and deploy the Tabular Model to an SSAS instance
* Process the Tabular Model against the database
* Query the measure under test
* Compare the query results to the expected values

To accomplish these tasks, we divided the work into three logical pieces:  A testing framework to do data comparisons, a helper module to interact with the SQL Server database, and a second helper to interact with the Tabular Model.

We want to run all of our tests locally, so we need to have both SQL Server and Analysis Services instances running on `localhost`; Both of these are freely available in the [developer edition](https://www.microsoft.com/en-us/sql-server/sql-server-downloads) of SQL Server.

## Testing Framework

## Interacting with the Database

The local SQL Server instance will act as the data source for our Tabular Model; in order to set up our tests, we need two things:  A way to set up our initial schema, and a way to manipulate data in the database.

### Schema Migration

The first thing we need is do build an empty database with the proper tables.  There are various ways to handle database migration, and the method you choose has no real effect on the rest of the testing pipeline; the only requirement is that the schema of the test database should match whatever is being used in production.

We use [alembic](https://pypi.python.org/pypi/alembic), but a [Database Project](https://msdn.microsoft.com/en-us/library/hh272677(v=vs.103).aspx) or [Entity Framework](https://msdn.microsoft.com/en-us/library/jj591621(v=vs.113).aspx) would work just as well.  Technically you could do this step by hand, and create the database as a one-off import, but if you aren't doing your database migrations in a repeatable way, you probably should be!

### Loading Data

Once we have our schema in place, we need a way to delete and insert data into our tables.  The [example](https://github.com/lobsteropteryx/tabular-example/blob/master/TestExample/SqlHelper.cs) uses a basic `SqlCommand`, but the details don't matter; EntityFramework or a micro-ORM would work just as well.  The only critical piece here is that we need to be able to expose our implementation to the test framework via [context injection](http://specflow.org/documentation/Context-Injection/).

## Interacting with the Tabular Model

To refresh and query the Tabular Model, we use the [ADOMDClient](https://msdn.microsoft.com/en-us/library/microsoft.analysisservices.adomdclient.aspx) library, which exposes a via a familiar set of interfaces (`Connection`, `Command`, `Adapter`, etc) for interacting with tabular data.

Note:  We needed to install the [unofficial version](https://github.com/ogaudefroy/Unofficial.Microsoft.AnalysisServices.AdomdClient) of the package, as outlined [here](https://stackoverflow.com/a/44896064).

Similar to the SQL Server module, we want to share this package among our test package;  in the example project, both the [`SQLHelper`](https://github.com/lobsteropteryx/tabular-example/blob/master/TestExample/SqlHelper.cs) and [`TabularHelper`](https://github.com/lobsteropteryx/tabular-example/blob/master/TestExample/TabularHelper.cs) modules are injected via the [ConnectionSupport](https://github.com/lobsteropteryx/tabular-example/blob/master/TestExample/ConnectionSupport.cs) class.

### Refreshing the Model

The ADOMD client can execute [commands](https://docs.microsoft.com/en-us/sql/analysis-services/tabular-model-scripting-language-tmsl-reference) directly against the model, in either XMLA or TMSL; to refresh (or "process") our Tabular Model, we use a simple [helper function](https://github.com/lobsteropteryx/tabular-example/blob/master/TestExample/TabularHelper.cs#L36).

### Querying the Measure under Test

Once we've loaded our test data, we want to query whichever measure we're testing, and check the output against our expected values.  Because there's we're only interested in the data itself (there's no real business logic), we opted to use a simple `DataTable` to return the data.

In the example project, there's a [single method](https://github.com/lobsteropteryx/tabular-example/blob/master/TestExample/TabularHelper.cs#L22) which executes an arbitrary DAX query and returns a `DataTable` object; you can easily flesh out the `TabularHelper` with additional query methods, to better encapsulate the query logic.

## Running the Test Suite

Now that we have our tests and helper modules in place, we need to build and deploy our model, then execute the test suite.  The build and deploy process is the same as outlined in the [previous post](/post/analysis-services-1-deployment).  We typically run our tests via a PowerShell script, but you can also run them from Visual Studio itself, using the [IDE Integration](https://specflow.org/documentation/Install-IDE-Integration/).

### Configuring Connection Strings

Assuming that we have local copies of both the database and Tabular model, the last thing we need to do is inject the proper connection strings to our helper modules.  We used environment variables to do this, but a config file could also work.

To set up the proper environment variables, execute the following:

```powershell
$env:TabularAcceptanceTestAnalysisServicesConnectionString = "Data Source=localhost;Catalog=validation;User ID=test_ssas;Password=test_ssas"

$env:TabularAcceptanceTestSqlConnectionString = "Server=localhost;Database=validation;User ID=test;Password=test;"
```

Note that if you want to run the tests from Visual Studio, you'll need these environment variables set in the process where Visual Studio is running; one way to do this is to luanch Visual Studio from a shell with the appropriate variables set:

```powershell
'C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\devenv.exe'
```

### Running the tests

Once the environment is set up, you can execute the scripts to build and deploy the model, then run the test suite:

```powershell
.\deploy_model.ps1 -workspace c:/develop/tabular-automation -environment validation -analysisServicesUsername test_ssas -analysisServicesPassword test_ssas
```

```powershell
.\run_validation.ps1 -databaseUsername test -databasePassword test -analysisServicesUsername test_ssas -analysisServicesPassword test_ssas
```

You should see output like the following:

```powershell
NUnit Console Runner 3.7.0
Copyright (c) 2017 Charlie Poole, Rob Prouse

Runtime Environment
   OS Version: Microsoft Windows NT 10.0.16299.0
  CLR Version: 4.0.30319.42000

Test Files
    TestExample\TestExample.csproj

=> TestExample.AverageAgeFeature.ASinglePerson
Given I have persons:
  --- table step argument ---
  | Name | Age |
  | Bob  | 5   |
-> done: AverageAgeSteps.GivenIHavePersons(<table>) (3.5s)
When I query for average age
-> done: AverageAgeSteps.WhenIQueryForAverageAge() (0.2s)
Then I expect the result to be:
  --- table step argument ---
  | Average |
  | 5       |
-> done: AverageAgeSteps.ThenIExpectTheResultToBe(<table>) (0.0s)
=> TestExample.AverageAgeFeature.BlankAgesAreTreatedAsZero
Given I have persons:
  --- table step argument ---
  | Name | Age |
  | Bob  | 5   |
  | Jane |     |
-> done: AverageAgeSteps.GivenIHavePersons(<table>) (0.3s)
When I query for average age
-> done: AverageAgeSteps.WhenIQueryForAverageAge() (0.0s)
Then I expect the result to be:
  --- table step argument ---
  | Average |
  | 2.5     |
-> done: AverageAgeSteps.ThenIExpectTheResultToBe(<table>) (0.0s)
=> TestExample.AverageAgeFeature.TwoPeopleWithTheSameAge
Given I have persons:
  --- table step argument ---
  | Name | Age |
  | Bob  | 5   |
  | Jane | 5   |
-> done: AverageAgeSteps.GivenIHavePersons(<table>) (0.2s)
When I query for average age
-> done: AverageAgeSteps.WhenIQueryForAverageAge() (0.0s)
Then I expect the result to be:
  --- table step argument ---
  | Average |
  | 5       |
-> done: AverageAgeSteps.ThenIExpectTheResultToBe(<table>) (0.0s)
=> TestExample.AverageAgeFeature
-> Using app.config

Run Settings
    BasePath: C:\develop\tabular-automation\TestExample\bin\Debug\
    DisposeRunners: True
    WorkDirectory: C:\develop\tabular-automation
    ImageRuntimeVersion: 4.0.30319
    ImageTargetFrameworkName: .NETFramework,Version=v4.5.2
    ImageRequiresX86: False
    ImageRequiresDefaultAppDomainAssemblyResolver: False
    NumberOfTestWorkers: 2

Test Run Summary
  Overall result: Passed
  Test Count: 3, Passed: 3, Failed: 0, Warnings: 0, Inconclusive: 0, Skipped: 0
  Start time: 2018-01-07 17:11:56Z
    End time: 2018-01-07 17:12:02Z
    Duration: 5.872 seconds

Results (nunit3) saved as TestResult.xml
```

## The End Result

Once we have the deployment and test scripts in place, we can integrate them with a CI server to set up a deployment pipeline.  To do this, we first deploy the new version of the model to a dedicated `validation` environment where the automated test suite is run; if all the tests pass, we automatically build and deploy again, to a staging environment.  In the staging environment, analysts can interact with the Tabular model through front-end tools like PowerBI; once the changes have been approved, we can promote the new version of the model to production.

