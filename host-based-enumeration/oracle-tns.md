# Oracle TNS

The `Oracle Transparent Network Substrate` (`TNS`) server is a communication protocol that facilitates communication between Oracle databases and applications over networks. Initially introduced as part of the [Oracle Net Services](https://docs.oracle.com/en/database/oracle/oracle-database/18/netag/introducing-oracle-net-services.html) software suite, TNS supports various networking protocols between Oracle databases and client applications, such as `IPX/SPX` and `TCP/IP` protocol stacks

Has been updated to support IPv6 and SSL/TLS which allows for:

* Name resolution
* Connection management
* Load balancing
* Security

## Default Configuration

Defaults to TCP port 1521, but can be changed at any time

TNS listener supports TCP/IP, UDP, IPX/SPX, and AppleTalk

By default, can be managed in Oracle 8i/9i but not 10g/11g

The configuration files for Oracle TNS are called `tnsnames.ora` and `listener.ora` and are typically located in the `$ORACLE_HOME/network/admin` directory. The plain text file contains configuration information for Oracle database instances and other network services that use the TNS protocol

Oracle 9 has a default password, `CHANGE_ON_INSTALL`, whereas Oracle 10 has no default password set. The Oracle DBSNMP service also uses a default password, `dbsnmp` that we should remember when we come across this one&#x20;

Another example would be that many organizations still use the `finger` service together with Oracle, which can put Oracle's service at risk and make it vulnerable when we have the required knowledge of a home directory

Each database or service has a unique entry in the [tnsnames.ora](https://docs.oracle.com/cd/E11882_01/network.112/e10835/tnsnames.htm#NETRF007) file, containing the necessary information for clients to connect to the service

Clients should use the service name `orcl` when connecting to the service

On the other hand, the `listener.ora` file is a server-side configuration file that defines the listener process's properties and parameters, which is responsible for receiving incoming client requests and forwarding them to the appropriate Oracle database instance

Oracle databases can be protected by using so-called PL/SQL Exclusion List (`PlsqlExclusionList`). It is a user-created text file that needs to be placed in the `$ORACLE_HOME/sqldeveloper` directory, and it contains the names of PL/SQL packages or types that should be excluded from execution

| **Setting**          | **Description**                                                                                                          |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `DESCRIPTION`        | A descriptor that provides a name for the database and its connection type.                                              |
| `ADDRESS`            | The network address of the database, which includes the hostname and port number.                                        |
| `PROTOCOL`           | The network protocol used for communication with the server                                                              |
| `PORT`               | The port number used for communication with the server                                                                   |
| `CONNECT_DATA`       | Specifies the attributes of the connection, such as the service name or SID, protocol, and database instance identifier. |
| `INSTANCE_NAME`      | The name of the database instance the client wants to connect.                                                           |
| `SERVICE_NAME`       | The name of the service that the client wants to connect to.                                                             |
| `SERVER`             | The type of server used for the database connection, such as dedicated or shared.                                        |
| `USER`               | The username used to authenticate with the database server.                                                              |
| `PASSWORD`           | The password used to authenticate with the database server.                                                              |
| `SECURITY`           | The type of security for the connection.                                                                                 |
| `VALIDATE_CERT`      | Whether to validate the certificate using SSL/TLS.                                                                       |
| `SSL_VERSION`        | The version of SSL/TLS to use for the connection.                                                                        |
| `CONNECT_TIMEOUT`    | The time limit in seconds for the client to establish a connection to the database.                                      |
| `RECEIVE_TIMEOUT`    | The time limit in seconds for the client to receive a response from the database.                                        |
| `SEND_TIMEOUT`       | The time limit in seconds for the client to send a request to the database.                                              |
| `SQLNET.EXPIRE_TIME` | The time limit in seconds for the client to detect a connection has failed.                                              |
| `TRACE_LEVEL`        | The level of tracing for the database connection.                                                                        |
| `TRACE_DIRECTORY`    | The directory where the trace files are stored.                                                                          |
| `TRACE_FILE_NAME`    | The name of the trace file.                                                                                              |
| `LOG_FILE`           | The file where the log information is stored.                                                                            |

### Setting Up - ODAT

```bash
sudo apt-get update
sudo apt-get install -y build-essential python3-dev libaio1
cd ~
wget https://files.pythonhosted.org/packages/source/c/cx_Oracle/cx_Oracle-8.3.0.tar.gz
tar xzf cx_Oracle-8.3.0.tar.gz
cd cx_Oracle-8.3.0
python3 setup.py build
sudo python3 setup.py install
cd ~
git clone https://github.com/quentinhardy/odat.git
cd odat/
pip install python-libnmap
git submodule init
git submodule update
sudo apt-get install python3-scapy -y
sudo pip3 install colorlog termcolor passlib python-libnmap
sudo apt-get install build-essential libgmp-dev -y
pip3 install pycryptodome
pip3 install openpyxl
```

Then, we can see if it was successful:

```bash
./odat.py -h
```

Oracle Database Attacking Tool (`ODAT`) is an open-source penetration testing tool written in Python and designed to enumerate and exploit vulnerabilities in Oracle databases. It can be used to identify and exploit various security flaws in Oracle databases, including SQL injection, remote code execution, and privilege escalation

Using nmap to scan default Oracle TNS listener port:

```bash
sudo nmap -p1521 -sV <ip> --open
```

In Oracle RDBMS, a System Identifier (`SID`) is a unique name that identifies a particular database instance. When a client connects to an Oracle database, it specifies the database's `SID` along with its connection string. The client uses this SID to identify which database instance it wants to connect to. Suppose the client does not specify a SID. Then, the default value defined in the `tnsnames.ora` file is used

There are various ways to enumerate, or better said, guess SIDs. Therefore we can use tools like `nmap`, `hydra`, `odat`, and others

Nmap SID Bruteforcing:

```bash
sudo nmap -p1521 -sV <ip> --open --script oracle-sid-brute
```

Enumeration and gathering using odat.py with all option:

```bash
./odat.py all -s <ip>
```

After valid credentials are found, use `sqlplus` to connect and interact:

```bash
sudo apt update
sudo apt upgrade parrot-core
sudo apt update
sudo apt install oracle-instantclient-sqlplus

sqlplus -v
sqlplus user/pass@<ip>/XE
```

[SQLplus list of commands](https://docs.oracle.com/cd/E11882_01/server.112/e41085/sqlqraa001.htm#SQLQR985)

Oracle RDBMS - Interaction

```sql
select table from all_tables;
select * from user_privs;
```

Oracle RDBMS - Database Enumeration

```bash
sqlplus user/pass@<ip>/XE as sysdba
```

We can follow many approaches once we get access to an Oracle database. It highly depends on the information we have and the entire setup. However, we can not add new users or make any modifications. From this point, we could retrieve the password hashes from the `sys.user$` and try to crack them offline

```bash
select name, password from sys.user$;
```

Another option is to upload a web shell to the target. However, this requires the server to run a web server, and we need to know the exact location of the root directory for the webserver

| **OS**  | **Path**             |
| ------- | -------------------- |
| Linux   | `/var/www/html`      |
| Windows | `C:\inetpub\wwwroot` |

Oracle RDBMS File Upload

```bash
./odat.py utlfile -s <ip> -d XE -U user -P pass --sysdba --putFile C:\\inetpub\\wwwroot testing.txt ./testing.txt
curl -X GET http://<ip>/testing.txt
```
