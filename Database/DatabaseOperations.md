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
# Import Backup to environnement Tier 1

## Restore SQL Script
``` SQL
USE [master]

DECLARE @backupFile nvarchar(100) = N'C:\Temp\AxDB.bak'
DECLARE @toMdf varchar(1000) = N'C:\Program Files\Microsoft SQL Server\MSSQL13.MSSQLSERVER\MSSQL\DATA\AxDB_' + CONVERT(nvarchar,(CONVERT (date,getdate()))) + '_Primary.mdf'
DECLARE @toLdf varchar(1000) = N'C:\Program Files\Microsoft SQL Server\MSSQL13.MSSQLSERVER\MSSQL\DATA\AxDB_' + CONVERT(nvarchar,(CONVERT (date,getdate()))) + '_Primary.ldf'
DECLARE @toAxDBName nvarchar(20) = 'AxDB_' + CONVERT(nvarchar,CONVERT (date,getdate()))

DECLARE @logicalNameMdf NVARCHAR(128)
DECLARE @logicalNameLdf NVARCHAR(128)

DECLARE @fileListTable TABLE (
    [LogicalName]           NVARCHAR(128),
    [PhysicalName]          NVARCHAR(260),
    [Type]                  CHAR(1),
    [FileGroupName]         NVARCHAR(128),
    [Size]                  NUMERIC(20,0),
    [MaxSize]               NUMERIC(20,0),
    [FileID]                BIGINT,
    [CreateLSN]             NUMERIC(25,0),
    [DropLSN]               NUMERIC(25,0),
    [UniqueID]              UNIQUEIDENTIFIER,
    [ReadOnlyLSN]           NUMERIC(25,0),
    [ReadWriteLSN]          NUMERIC(25,0),
    [BackupSizeInBytes]     BIGINT,
    [SourceBlockSize]       INT,
    [FileGroupID]           INT,
    [LogGroupGUID]          UNIQUEIDENTIFIER,
    [DifferentialBaseLSN]   NUMERIC(25,0),
    [DifferentialBaseGUID]  UNIQUEIDENTIFIER,
    [IsReadOnly]            BIT,
    [IsPresent]             BIT,
    [TDEThumbprint]         VARBINARY(32),
	[SnapshotUrl]			NVARCHAR(260)
)
INSERT INTO @fileListTable EXEC('RESTORE FILELISTONLY FROM DISK = ''' + @backupFile + '''')
SELECT @logicalNameMdf = LogicalName FROM @fileListTable where Type = 'D'
SELECT @logicalNameLdf = LogicalName FROM @fileListTable where Type = 'L'

RESTORE DATABASE @toAxDBName FROM  DISK = @backupFile WITH  FILE = 1,  MOVE @logicalNameMdf TO @toMdf,  MOVE @logicalNameLdf TO @toLdf,  NOUNLOAD,  STATS = 5

GO
```


