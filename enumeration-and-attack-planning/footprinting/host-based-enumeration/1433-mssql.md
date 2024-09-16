---
description: >-
  Microsoft SQL (MSSQL) is Microsoft's SQL-based relational database management
  system.
---

# MSSQL(1433)

## About

SQL Server Management Studio (SSMS) is available as an optional feature during the MSSQL installation or can be downloaded and installed separately. It is typically installed on the server to facilitate initial setup and ongoing database management by administrators.

### Clients

Many other clients can be used to access a database running on MSSQL such us:

* [Impacket's mssqlclient.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/mssqlclient.py)
* [SQLPro](https://www.macsqlclient.com/)
* [HeidiSQL](https://www.heidisql.com/)
* [SQL Server PowerShell](https://docs.microsoft.com/en-us/sql/powershell/sql-server-powershell?view=sql-server-ver15)
* [mssql-cli](https://docs.microsoft.com/en-us/sql/tools/mssql-cli?view=sql-server-ver15)

### Default system databases

MSSQL have a default system database that can help us navigate and understand the structure of all the databases that are hosted on your target. Here are few of them:

| `master`   | Tracks all system information for an SQL server instance                                                                                                                                               |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `model`    | Template database that acts as a structure for every new database created. Any setting changed in the model database will be reflected in any new database created after changes to the model database |
| `msdb`     | The SQL Server Agent uses this database to schedule jobs & alerts                                                                                                                                      |
| `tempdb`   | Stores temporary objects                                                                                                                                                                               |
| `resource` | Read-only database containing system objects included with SQL server                                                                                                                                  |

### Dangerous Settings

> This is not an extensive list because there are countless ways MSSQL databases can be configured by admins based on the needs of their respective organizations. We may benefit from looking into the following:
>
> * MSSQL clients not using encryption to connect to the MSSQL server
> * The use of self-signed certificates when encryption is being used. It is possible to spoof self-signed certificates
> * The use of [named pipes](https://docs.microsoft.com/en-us/sql/tools/configuration-manager/named-pipes-properties?view=sql-server-ver15)
> * Weak & default `sa` credentials. Admins may forget to disable this account

## Footprinting

### Nmap

```shell-session
$ sudo nmap <TARGET IP> -sV -p 1433 --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER  



PORT     STATE SERVICE  VERSION
1433/tcp open  ms-sql-s Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-ntlm-info: 
|   Target_Name: SQL-01
|   NetBIOS_Domain_Name: SQL-01
|   NetBIOS_Computer_Name: SQL-01
|   DNS_Domain_Name: SQL-01
|   DNS_Computer_Name: SQL-01
|_  Product_Version: 10.0.17763

Host script results:
| ms-sql-dac: 
|_  Instance: MSSQLSERVER; DAC port: 1434 (connection failed)
| ms-sql-info: 
|   Windows server name: SQL-01
|   10.129.201.248\MSSQLSERVER: 
|     Instance name: MSSQLSERVER
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|     TCP port: 1433
|     Named pipe: \\10.129.201.248\pipe\sql\query
|_    Clustered: false
```

### Metasploit

```sql
msf6 auxiliary(scanner/mssql/mssql_ping) > set rhosts <TARGET IP>
```

### Connecting to the server

```shell-session
$ python3 mssqlclient.py username@TARGET -windows-auth
```

Upon connecting to the server, we will need a bit of knowledge with T-SQL. The following command will list the databases available present in our system:

```sql
SQL> select name from sys.databases
```
