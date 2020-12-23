# Microsoft Hosted Build

## Update build version

### Get LCS assets

![LCS Assets Package Nuget](./../Images/LCSPackageNuget.png)

### Upload Nuget Package to feed

``` PowerShell
.\nuget.exe push -Source "GMTBuildD365FO" -ApiKey az "C:\temp\nuget\microsoft.dynamics.ax.application.devalm.buildxpp.10.0.644.10018.nupkg"
.\nuget.exe push -Source "GMTBuildD365FO" -ApiKey az "C:\temp\nuget\Microsoft.Dynamics.AX.Platform.CompilerPackage.7.0.5816.35654.nupkg"
.\nuget.exe push -Source "GMTBuildD365FO" -ApiKey az "C:\temp\nuget\Microsoft.Dynamics.AX.Platform.DevALM.BuildXpp.7.0.5816.35654.nupkg"
```

### Update package.config on Devops repository

## Links

<https://ariste.info/en/2020/05/azure-hosted-build-dynamics365-finance-scm/>
