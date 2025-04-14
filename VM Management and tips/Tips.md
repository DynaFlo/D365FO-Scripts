# Tips for VMS

## Password 

To decrypt run this command:

C:\AOSService\webroot\bin\Microsoft.Dynamics.AX.Framework.ConfigEncryptor.exe -decrypt C:\AOSService\webroot\web.config


To encrypt run this command:

C:\AOSService\webroot\bin\Microsoft.Dynamics.AX.Framework.ConfigEncryptor.exe -encrypt C:\AOSService\webroot\web.config


## Erreur ERR_HTTP2_INADEQUATE_TRANSPORT_SECURITY

Désactiver HTTP2 sur IIS avec la clé de registre suivante

``` REGEDIT
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\HTTP\Parameters]
"EnableHttp2Tls"=dword:00000000
"EnableHttp2Cleartext"=dword:00000000
```
