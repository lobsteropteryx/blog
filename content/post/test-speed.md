+++
date = "2018-01-27T00:00:00+00:00"
draft = false
title = "Test Speed"
+++

I recently did some work on a somewhat neglected C# code base; the basic function of the application was to pull down some custom reports from a third-party web API, parse the results, and store them in a database.  There were a few obvious design problems, and the application hadn't been built or deployed in years, but it did include a small test suite--around 50 unit tests, written with Microsoft's [testing framework](https://github.com/Microsoft/testfx).

I was able to get the project to compile without too much trouble, but when I ran the test suite, I was shocked--the run time for those 50 tests was around ten minutes!  At first I thought that the tests might be integrated tests in disguise--hitting the production API and database--but all the tests seemed to be mocking out everything appropriately, using the [Moq](https://github.com/moq/moq4) framework.

The answer turned out to be some throttling logic, based on default configuration values.  Included in the configuration settings was a value indicating how long the application should wait between requesting customer reports--30 seconds.  In production this didn't matter a great deal--the reports were pulled down nightly, as a scheduled task.  The test suite was another matter, though--this setting didn't affect every test, but it pushed several tests into the minute range for run time.

## The Solution

 The project was using a [settings](https://www.dotnetperls.com/settings) file to specify configuration values; the configuration file gets wrapped in a class automatically, and the values can be changed at runtime.  By using the [`AssemblyInitialize`](https://msdn.microsoft.com/en-us/library/microsoft.visualstudio.testtools.unittesting.assemblyinitializeattribute.aspx) annotation, I was able to set the timeout value to zero during the test runs:

```C#
[TestClass]
public class TestInitialization
{
    [AssemblyInitialize]
    public static void BeforeEach(TestContext testContext)
    {
        Properties.Settings.Default.CustomerReportAccessDelayInSeconds = 0;
    }
}
```

Doing this brought the total runtime for the test suite down to 20 seconds--not great, but not bad for C#!

## The Takeaway:  Keep things Fast!

This might seem like an obvious fix, but the original developers lived with a ten minute test suite, and thought that was normal.  Twenty seconds seems fast in comparison, but for a typical Ruby or Python test suite, I'd expect to do at _least_ an order of magnitude better.  Faster is always preferable, but the context is obviously going to make a difference.

In general, when it comes to TDD, anything longer than a few seconds will have me looking to optimize--either looking at my design, checking my tooling, or for very large suites, by running a subset of tests.