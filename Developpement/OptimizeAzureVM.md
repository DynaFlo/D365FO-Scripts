# Optimisation des VM Azure

## Déplacement des fichiers tempdb

- Passage du service sql Server en 

Création d'un task de démarraage dans le task scheduler qui exécute la commande "md d:\tempdb"

Paramétrage du redémarrage de SQL Server après une première erreur.

``` SQL
CREATE PROCEDURE DBO.CREATE_TEMPDBDIRECTORY
AS
exec sp_configure 'xp_cmdshell', 1

EXEC xp_cmdshell  'md D:\\tempdb'
GO

USE MASTER
GO
EXEC SP_PROCOPTION CREATE_TEMPDBDIRECTORY, 'STARTUP', 'ON'
GO




USE master;
GO

ALTER DATABASE tempdb 
MODIFY FILE (NAME = tempdev, FILENAME = 'D:\tempDB\tempdb.mdf');
GO

ALTER DATABASE tempdb 
MODIFY FILE (NAME = templog, FILENAME = 'D:\tempDB\templog.ldf');
GO

ALTER DATABASE tempdb 
MODIFY FILE (NAME = tempdev2, FILENAME = 'D:\tempDB\tempdb2.mdf');
GO

ALTER DATABASE tempdb 
MODIFY FILE (NAME = tempdev3, FILENAME = 'D:\tempDB\tempdb3.mdf');
GO

ALTER DATABASE tempdb 
MODIFY FILE (NAME = tempdev4, FILENAME = 'D:\tempDB\tempdb4.mdf');
GO

tempdev	I:\TEMPDB_STORAGE\tempdb.mdf	ONLINE
templog	I:\TEMPDB_STORAGE\templog.ldf	ONLINE
tempdev2	I:\TEMPDB_STORAGE\tempdb2.ndf	ONLINE
tempdev3	I:\TEMPDB_STORAGE\tempdb3.ndf	ONLINE
tempdev4	I:\TEMPDB_STORAGE\tempdb4.ndf	ONLINE

USE master;
GO

ALTER DATABASE tempdb 
MODIFY FILE (NAME = tempdev, FILENAME = 'I:\TEMPDB_STORAGE\tempdb.mdf');
GO

ALTER DATABASE tempdb 
MODIFY FILE (NAME = templog, FILENAME = 'I:\TEMPDB_STORAGE\templog.ldf');
GO

ALTER DATABASE tempdb 
MODIFY FILE (NAME = tempdev2, FILENAME = 'I:\TEMPDB_STORAGE\tempdb2.mdf');
GO

ALTER DATABASE tempdb 
MODIFY FILE (NAME = tempdev3, FILENAME = 'I:\TEMPDB_STORAGE\tempdb3.mdf');
GO

ALTER DATABASE tempdb 
MODIFY FILE (NAME = tempdev4, FILENAME = 'I:\TEMPDB_STORAGE\tempdb4.mdf');
GO
```
