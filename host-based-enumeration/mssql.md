# MSSQL

Closed source, works very well with .NET

Defaults to TCP port 1443

There are versions of MSSQL that will run on Linux and MacOS, but we will more likely come across MSSQL instances on targets running Windows

[SQL Server Management Studio](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver15) (`SSMS`) comes as a feature that can be installed with the MSSQL install package or can be downloaded & installed separately. It is commonly installed on the server for initial configuration and long-term management of databases by admins

A compromised system with SSMS on it could leave saved credentials we can use to connect to the DB

Other clients that can access a MSSQL DB include:

* [mssql-cli](https://docs.microsoft.com/en-us/sql/tools/mssql-cli?view=sql-server-ver15)
* [SQL Server PowerShell](https://docs.microsoft.com/en-us/sql/powershell/sql-server-powershell?view=sql-server-ver15)
* ​[HeidiSQL](https://www.heidisql.com)
* [SQLPro](https://www.macsqlclient.com)

mssqlclient.py is the most useful since its present on many pentesting distros on install

```bash
locate mssqlclient
```

### MSSQL Databases

| Default System Database | Description                                                                                                                                                                                            |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `master`                | Tracks all system information for an SQL server instance                                                                                                                                               |
| `model`                 | Template database that acts as a structure for every new database created. Any setting changed in the model database will be reflected in any new database created after changes to the model database |
| `msdb`                  | The SQL Server Agent uses this database to schedule jobs & alerts                                                                                                                                      |
| `tempdb`                | Stores temporary objects                                                                                                                                                                               |
| `resource`              | Read-only database containing system objects included with SQL server                                                                                                                                  |

## Default Configuration

On initial install, the SQL service will likely run as `NT SERVICE\MSSQLSERVER`

Authentication being set to `Windows Authentication` means that the underlying Windows OS will process the login request and use either the local SAM database or the domain controller (hosting Active Directory) before allowing connectivity to the database management system.

## Dangerous Settings

Good things to look into:

* MSSQL clients not using encryption to connect to the MSSQL server
* The use of self-signed certificates when encryption is being used. It is possible to spoof self-signed certificates
* The use of [named pipes](https://docs.microsoft.com/en-us/sql/tools/configuration-manager/named-pipes-properties?view=sql-server-ver15)
* Weak & default `sa` credentials. Admins may forget to disable this account

## Footprinting the Service

Nmap MSSQL Scan with Script:

```bash
sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p <ip>
```

We can also use Metasploit to run an auxiliary scanner called `mssql_ping` that will scan the MSSQL service and provide helpful information in our footprinting process.

```
auxiliary(scanner/mssql/mssql_ping)
```

Once credentials are in place, use mssqlclient.py:

```bash
python3 mssqlclient.py Administrator@<ip> -windows-auth
```

