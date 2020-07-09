**Database powerShell scripts**

- [Import Lcs bacpac to environnement Tier 1](#import-lcs-bacpac-to-environnement-tier-1)
  - [Get-LCS Token and set Config ](#get-lcs-token-and-set-config )
  - [Database import](#database-import)


# Import Lcs bacpac to environnement Tier 1

## Get-LCS Token and set Config 

$clientId of registred application with Dynamics Lifecycle services authorization

```powershell
$clientId = ""
$userName = "YourUserMail"
$passWord = 'YourPassword'

Get-D365LcsApiToken -ClientId $clientId -Username $userName -Password $passWord -LcsApiUri "https://lcsapi.lcs.dynamics.com" -Verbose | Set-D365LcsApiConfig -ProjectId $projectId -ClientId $clientId
```

## Database import

```powershell
$currentDate = Get-Date -Format yyyyMMdd
$bacpacName = "UAT{0}" -f $currentDate
$downloadPath = "D:\UAT{0}.bacpac" -f $currentDate
$newDBName = "AxDB_{0}" -f $currentDate

Get-D365LcsApiConfig | Invoke-D365LcsApiRefreshToken | Set-D365LcsApiConfig

$backups = Get-D365LcsDatabaseBackups

$fileLocation = $backups[0].FileLocation

Invoke-D365AzCopyTransfer -SourceUri $fileLocation -DestinationUri $downloadPath

Import-D365Bacpac -ImportModeTier1 -BacpacFile $downloadPath -NewDatabaseName $newDBName

Invoke-D365DbSync -DatabaseName $newDBName

Stop-D365Environment

$destSuffix = "_OLD_" + $currentDate

Switch-D365ActiveDatabase -NewDatabaseName $newDBName -DestinationSuffix $destSuffix

Start-D365Environment

$d365user = Get-D365User

foreach ($user in $d365user)
{
  if ($user.email -match "silverprod")
  {
    Update-D365User -Email $user.email
    Write-Host $user.Email
  }
}
```
