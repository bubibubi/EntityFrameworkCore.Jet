# Notes for Developers/Builders

## Important Notes as of Feb 21, 2020
- System.Data.OleDb currently has several issues with x86, testing on x64 is recommended.

## System.Data.OleDb Issues
- Current Nuget.org package (version 4.7.0) contains several significant bugs:
    - https://github.com/dotnet/runtime/issues/32509
    - (fixed, awaiting merge) https://github.com/dotnet/runtime/issues/31177
    - (fixed, merged) https://github.com/dotnet/runtime/issues/981
- Use NuGet.Config (see below) to get nightly builds instead of using the published version

## Nuget.Config
- Used to get latest nightly builds of System.Data.OleDb
- When a public release is posted to NuGet.org, having the NuGet.Config file likely won't be necessary

## .NET Core 3.0/3.1 Issues
- using the dynamic keyword with COM objects not supported on .NET Core 3.0/3.1
- instead, System.Data.Jet.csproj includes two COMReference nodes for ADODB.dll & ADOX.dll, which generate strongly-typed references to these packages.  No x86/x64 issues have been encountered during preliminary tests, but I believe .NET 5.0 may support using the dynamic keyword with COM objects, so the code could possibly be reverted to the way it was, if issues with the COM wrappers are found.
- Affects AdoxWrapper.cs, JetStoreSchemaDefinitionRetrieve.cs & JetSyntaxHelper.cs
- FYI: Paths to msadox.dll:
    - C:\Program Files\Common Files\System\ado
    - C:\Program Files (x86)\Common Files\System\ado

## Directory.Build.targets
- Used due to conflict between the EF Core 2.2 and .NET Core 3.1 versions of IAsyncGrouping<,> & IAsyncEnumerable<>.  Invoked via setting an Alias attribute on the appropriate PackageReference.
- Affects SharedTypeExtensions.cs

## Test Design Guidelines
- Consider extracting out global test configuration settings to a single file:
    - Provider [string]: "Microsoft.ACE.OLEDB.15.0"
    - TempDirectoryPath [string]: "C:\TEMP"
    - RunSqlCeTests [bool] : true/false
    - RunSqlServerTests [bool] : true/false
    - RunSqliteTests [bool] : true/false

## Prerequisites for Building/Running Tests
- Install SQL Compact 4.0
  - x86 or x64 (whichever applies to the version of Windows used)
- Install Microsoft Access 2013 Runtime (https://www.microsoft.com/en-us/download/details.aspx?id=39358)
  - x86, x64, or Both (not sure if installing both x86 and x64 side-by-side works properly)
- The folder "C:\TEMP" must exist, certain tests will throw if it does not

## Test Runner Issues
- Test projects need to use Microsoft.NET.Test.SDK version 16.4.0 or higher to switch between x86/x64, otherwise Visual Studio test runner does not respect the "Processor Architecture for Any CPU" setting and still tries to run x86 tests on x64.
- Refer to: https://developercommunity.visualstudio.com/content/problem/697732/test-runner-wont-execute-net-core-tests-in-32-bit.html
- Ensure xUnit test projects contain a reference to the nuget package xunit.runner.visualstudio to run the test from Visual Studio Test Explorer

## General Resources
- Multitargeting
https://docs.microsoft.com/en-us/dotnet/core/tutorials/libraries#how-to-multitarget
- MS Access Data Manipulation Language
https://docs.microsoft.com/en-us/office/client-developer/access/desktop-database-reference/data-manipulation-language

## Outdated Notes that Probably are No Longer Important
- Probably want to set Visual Studio to use PackageReference instead of packages.config by default (Options->Nuget Package Manager)