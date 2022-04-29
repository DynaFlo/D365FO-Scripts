# Microsoft Hosted Build

## Update build version

### Get LCS assets

![LCS Assets Package Nuget](./../Images/LCSPackageNuget.png)

### Create nuget.config file

``` XML
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <clear />
    <add key="AASBuild" value="https://pkgs.dev.azure.com/aariste/aariste365FO/_packaging/AASBuild/nuget/v3/index.json" />
  </packageSources>
</configuration>
```

### Install the credential provider

<https://raw.githubusercontent.com/microsoft/artifacts-credprovider/master/helpers/installcredprovider.ps1>

Execute de script with "-AddNetfx" parameters

``` PowerShell
./installcredprovider.ps1 -AddNetfx
```

Si necessaire utiliser la commande suivante pour bypasser le message d'erreur
```
Set-ExecutionPolicy Bypass -Scope Process -Force;
```

### Upload Nuget Package to feed

``` PowerShell
.\nuget.exe push -Source "BuildD365FO" -ApiKey az "Microsoft.Dynamics.AX.Application.DevALM.BuildXpp.nupkg"
.\nuget.exe push -Source "BuildD365FO" -ApiKey az "Microsoft.Dynamics.AX.ApplicationSuite.DevALM.BuildXpp.nupkg"
.\nuget.exe push -Source "BuildD365FO" -ApiKey az "Microsoft.Dynamics.AX.Platform.CompilerPackage.nupkg"
.\nuget.exe push -Source "BuildD365FO" -ApiKey az "Microsoft.Dynamics.AX.Platform.DevALM.BuildXpp.nupkg"
```

### Update package.config on Devops repository

## Links

<https://ariste.info/en/2020/05/azure-hosted-build-dynamics365-finance-scm/>
