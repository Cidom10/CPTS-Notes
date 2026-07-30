# MySQL

MySQL works according to the `client-server principle` and consists of a MySQL server and one or more MySQL clients.&#x20;

Typically runs on **TCP port 3306**

The MySQL server is the actual database management system. It takes care of data storage and distribution. The data is stored in tables with different columns, rows, and data types. These databases are often exported or backed up as a single `.sql` file, for example `wordpress.sql`

### MySQL Clients

The MySQL clients can retrieve and edit the data using structured queries to the database engine. Inserting, deleting, modifying, and retrieving data, is done using the SQL database language

Depending on the use of the database, access is possible via an internal network or the public Internet

### MySQL Databases

MySQL is ideally suited for applications such as `dynamic websites`, where efficient syntax and high response speed are essential

Often cobined with LAMP (Linux, Apache, MySQL, PHP) or LEMP (Nginx)

Stores content required by PHP scripts, such as Headers Texts Meta tags Forms Customers Usernames Administrators Moderators Email addresses User information Permissions Passwords External/Internal links Links to Files Specific contents Values

## Default Configuration

```bash
cat /etc/mysql/mysql.conf.d/mysqld.cnf | grep -v "#" | sed -r '/^\s*$/d'
```

## Dangerous Settings

| **Settings**       | **Description**                                                                                              |
| ------------------ | ------------------------------------------------------------------------------------------------------------ |
| `user`             | Sets which user the MySQL service will run as.                                                               |
| `password`         | Sets the password for the MySQL user.                                                                        |
| `admin_address`    | The IP address on which to listen for TCP/IP connections on the administrative network interface.            |
| `debug`            | This variable indicates the current debugging settings                                                       |
| `sql_warnings`     | This variable controls whether single-row INSERT statements produce an information string if warnings occur. |
| `secure_file_priv` | This variable is used to limit the effect of data import and export operations.                              |

The settings `user`, `password`, and `admin_address` are security-relevant because the entries are made in plain text

## Footprinting the Service

Scanning with Nmap:

```bash
sudo nmap <ip> -sV -sC -p3306 --script mysql*
```

As with all our scans, we must be careful with the results and manually confirm the information obtained because some of the information might turn out to be a false-positive.&#x20;

This scan above is an excellent example of this, as we know for a fact that the target MySQL server does not use an empty password for the user `root`, but a fixed password. We can test this with the following command:

```bash
mysql -u root -h <ip>
```

Logging in with a known password:

```bash
mysql -u root -p P4SSw0rd -h <ip>
```

Useful SQL commands:

| **Command**                                          | **Description**                                     |
| ---------------------------------------------------- | --------------------------------------------------- |
| `show databases;`                                    | Show all databases.                                 |
| `use <database>;`                                    | Select one of the existing databases.               |
| `show tables;`                                       | Show all available tables in the selected database. |
| `show columns from <table>;`                         | Show all columns in the selected table.             |
| `select * from <table>;`                             | Show everything in the desired table.               |
| `select * from <table> where <column> = "<string>";` | Search for needed `string` in the desired table.    |

The `information schema` is also a database that contains metadata. However, this metadata is mainly retrieved from the `system schema` database. The reason for the existence of these two is the ANSI/ISO standard that has been established
