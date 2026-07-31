# Host-Based Enumeration

## FTP



FTP runs through port 21 for establishing a connnection, then comms pass through port 20

In **active FTP**, client establishes connection in port 21 and informs the server which port they can send their response through. If client has a firewall, this causes problems.

In **passive FTP**, server announces a port. Since client initiates this connection, firewall does not block

[Huge list of FTP commands](https://web.archive.org/web/20230326204635/https://www.smartfile.com/blog/the-ultimate-ftp-commands-list/)

[FTP Server Return Codes](https://en.wikipedia.org/wiki/List_of_FTP_server_return_codes)



### TFTP (Trivial FTP)

Simpler, but does not provide user auth and other features that FTP does

Uses UDP



### Default Configuration

One of Linux's most popular FTP servers is [vsFTPd](https://security.appspot.com/vsftpd.html)

Default config for vsFTPd is found in /etc/vsftpd.conf

| **Setting**                                                   | **Description**                                                                                          |
| ------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| `listen=NO`                                                   | Run from inetd or as a standalone daemon?                                                                |
| `listen_ipv6=YES`                                             | Listen on IPv6 ?                                                                                         |
| `anonymous_enable=NO`                                         | Enable Anonymous access?                                                                                 |
| `local_enable=YES`                                            | Allow local users to login?                                                                              |
| `dirmessage_enable=YES`                                       | Display active directory messages when users go into certain directories?                                |
| `use_localtime=YES`                                           | Use local time?                                                                                          |
| `xferlog_enable=YES`                                          | Activate logging of uploads/downloads?                                                                   |
| `connect_from_port_20=YES`                                    | Connect from port 20?                                                                                    |
| `secure_chroot_dir=/var/run/vsftpd/empty`                     | Name of an empty directory                                                                               |
| `pam_service_name=vsftpd`                                     | This string is the name of the PAM service vsftpd will use.                                              |
| `rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem`          | The last three options specify the location of the RSA certificate to use for SSL encrypted connections. |
| `rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key` |                                                                                                          |
| `ssl_enable=NO`                                               |                                                                                                          |
|                                                               |                                                                                                          |

A file at /etc/ftpusers can tell us who is denied login to the FTP service

### Dangerous Settings

&#x20;Leaving the anonymous user can be dangerous.&#x20;

List of vsFTPd settings for anon user:

| **Setting**                    | **Description**                                                                    |
| ------------------------------ | ---------------------------------------------------------------------------------- |
| `anonymous_enable=YES`         | Allowing anonymous login?                                                          |
| `anon_upload_enable=YES`       | Allowing anonymous to upload files?                                                |
| `anon_mkdir_write_enable=YES`  | Allowing anonymous to create new directories?                                      |
| `no_anon_password=YES`         | Do not ask anonymous for password?                                                 |
| `anon_root=/home/username/ftp` | Directory for anonymous.                                                           |
| `write_enable=YES`             | Allow the usage of FTP commands: STOR, DELE, RNFR, RNTO, MKD, RMD, APPE, and SITE? |

Use `status` to get vsFTPd settings

Use `debug` and `trace` to get more information for us to use

Recursively list directories with `ls -R`

Download all available files on a FTP server:

```bash
wget -m --no-passive ftp://anonymous:anonymous@<ip>
```

### Footprinting the Service

First, update Nmap scripts db (NSE) with `sudo nmap --script-updatedb`

Then, we can try to find the FTP using an aggressive version scan with default script scanning:

```bash
sudo nmap -sV -p21 -sC -A <ip>
```

Depending on openings, this can show things like anon access, files open, writable, file size, etc

Trace progress of NSE scripts:

```shellscript
sudo nmap -sV -p21 -sC -A 10.129.14.136 --script-trace
```

## SMB

Protocol to regulate access to files, directories, and resources such as printers and router)

Just Windows systems, Samba is used on Linux and Unix

Both sides must establish a connection, done over TCP

### Samba

Samba implements the Common Internet File System (CIFS) protocol.

CIFS is a dialect of SMB

Samba to older NetBIOS services, ports 137, 138, and 139 are used

CIFS operates exclusively over port 445

| **SMB Version** | **Supported**                       | **Features**                                                           |
| --------------- | ----------------------------------- | ---------------------------------------------------------------------- |
| CIFS            | Windows NT 4.0                      | Communication via NetBIOS interface                                    |
| SMB 1.0         | Windows 2000                        | Direct connection via TCP                                              |
| SMB 2.0         | Windows Vista, Windows Server 2008  | Performance upgrades, improved message signing, caching feature        |
| SMB 2.1         | Windows 7, Windows Server 2008 R2   | Locking mechanisms                                                     |
| SMB 3.0         | Windows 8, Windows Server 2012      | Multichannel connections, end-to-end encryption, remote storage access |
| SMB 3.0.2       | Windows 8.1, Windows Server 2012 R2 |                                                                        |
| SMB 3.1.1       | Windows 10, Windows Server 2016     | Integrity checking, AES-128 encryption                                 |



### Default Configuration

Samba config can be found at /etc/samba/smb.conf

| **Setting**                    | **Description**                                                       |
| ------------------------------ | --------------------------------------------------------------------- |
| `[sharename]`                  | The name of the network share.                                        |
| `workgroup = WORKGROUP/DOMAIN` | Workgroup that will appear when clients query.                        |
| `path = /path/here/`           | The directory to which user is to be given access.                    |
| `server string = STRING`       | The string that will show up when a connection is initiated.          |
| `unix password sync = yes`     | Synchronize the UNIX password with the SMB password?                  |
| `usershare allow guests = yes` | Allow non-authenticated users to access defined share?                |
| `map to guest = bad user`      | What to do when a user login request doesn't match a valid UNIX user? |
| `browseable = yes`             | Should this share be shown in the list of available shares?           |
| `guest ok = yes`               | Allow connecting to the service without using a password?             |
| `read only = yes`              | Allow users to read files only?                                       |
| `create mask = 0700`           | What permissions need to be set for newly created files?              |

### Dangerous Settings

| **Setting**                 | **Description**                                                     |
| --------------------------- | ------------------------------------------------------------------- |
| `browseable = yes`          | Allow listing available shares in the current share?                |
| `read only = no`            | Forbid the creation and modification of files?                      |
| `writable = yes`            | Allow users to create and modify files?                             |
| `guest ok = yes`            | Allow connecting to the service without using a password?           |
| `enable privileges = yes`   | Honor privileges assigned to specific SID?                          |
| `create mask = 0777`        | What permissions must be assigned to the newly created files?       |
| `directory mask = 0777`     | What permissions must be assigned to the newly created directories? |
| `logon script = script.sh`  | What script needs to be executed on the user's login?               |
| `magic script = script.sh`  | Which script should be executed when the script gets closed?        |
| `magic output = script.out` | Where the output of the magic script needs to be stored?            |

Example of danger of browsing:

1. Add a new share
2. Restart the daemon with `sudo systemctl restart smbd`
3. Connect with `smbclient -N -L //<ip>`  (-N is null session aka anonymous)

Use `smbstatus` to see connections to samba instance

### Footprinting the Service

SMB scans in Nmap can take a long time, so try to look manually

```bash
sudo nmap <ip> -sV -sC -p139,445
```

**rpcclient** is a good tool to perform MS-RPC functions

The [Remote Procedure Call](https://www.geeksforgeeks.org/remote-procedure-call-rpc-in-operating-system/) (RPC) is a tool to realize operational and work-sharing structures in networks and client-server architectures

```bash
rpcclient -U "" <ip>
```

rpcclient commands:

| **Query**                 | **Description**                                                    |
| ------------------------- | ------------------------------------------------------------------ |
| `srvinfo`                 | Server information.                                                |
| `enumdomains`             | Enumerate all domains that are deployed in the network.            |
| `querydominfo`            | Provides domain, server, and user information of deployed domains. |
| `netshareenumall`         | Enumerates all available shares.                                   |
| `netsharegetinfo <share>` | Provides information about a specific share.                       |
| `enumdomusers`            | Enumerates all domain users.                                       |
| `queryuser <RID>`         | Provides information about a specific user.                        |

To enumerate users with rpcclien&#x74;**:**

```bash
enumdomusers
queryuser <pointer, ex: 0x3e8>
querygroup 0x201
```

To bruteforce User RIDs:

```bash
for i in $(seq 500 1100);do rpcclient -N -U "" 10.129.14.128 -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo "";done
```

A great alternative is the Python script [samrdump.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/samrdump.py)

[SMBMap](https://github.com/ShawnDEvans/smbmap) and [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) help with SMB enumeration&#x20;

```bash
smbmap -H <ip>
crackmapexec smb <ip> --shares -u '' -p ''
```

[enum3linux-ng](https://github.com/cddmp/enum4linux-ng) automates many of the queries and can give a lot of information

```bash
git clone https://github.com/cddmp/enum4linux-ng.git
cd enum4linux-ng
pip3 install -r requirements.txt
./enum4linux-ng.py <ip> -A
```



## NFS

Same purpose as SMB, completely different protocol

Used between Linux and Unix systems

Based on the [Open Network Computing Remote Procedure Call](https://en.wikipedia.org/wiki/Sun_RPC) (ONC-RPC/SUN-RPC)

Most common auth is via UID/GID and group memberships

list of filesystems available via NFS at /etc/exports

Additional options that can be added to hosts or subnets:

| **Option**         | **Description**                                                                                                                             |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `rw`               | Read and write permissions.                                                                                                                 |
| `ro`               | Read only permissions.                                                                                                                      |
| `sync`             | Synchronous data transfer. (A bit slower)                                                                                                   |
| `async`            | Asynchronous data transfer. (A bit faster)                                                                                                  |
| `secure`           | Ports above 1024 will not be used.                                                                                                          |
| `insecure`         | Ports above 1024 will be used.                                                                                                              |
| `no_subtree_check` | This option disables the checking of subdirectory trees.                                                                                    |
| `root_squash`      | Assigns all permissions to files of root UID/GID 0 to the UID/GID of anonymous, which prevents `root` from accessing files on an NFS mount. |

Use `exportfs` to grab the NFS Exports

```bash
echo '/mnt/nfs  10.129.14.0/24(sync,no_subtree_check)' >> /etc/exports
systemctl restart nfs-kernel-server 
exportfs
```

### Dangerous Settings

| **Option**       | **Description**                                                                                                      |
| ---------------- | -------------------------------------------------------------------------------------------------------------------- |
| `rw`             | Read and write permissions.                                                                                          |
| `insecure`       | Ports above 1024 will be used.                                                                                       |
| `nohide`         | If another file system was mounted below an exported directory, this directory is exported by its own exports entry. |
| `no_root_squash` | All files created by root are kept with the UID/GID 0.                                                               |

Recommended to use VM and experiment with settings

### Footprinting the Service

TCP ports 111 and 2049 are essential

Can use Nmap to get info abouit NFS and the host via RPC

```bash
sudo nmap <ip> -p111,2049 -sV -sC
```

The `rpcinfo` NSE script retrieves a list of all currently running RPC services, their names and descriptions, and the ports they use.

```bash
sudo nmap --script nfs* <ip> -sV -p111,2049
```

Once NFS service is found, we can mount to our local machine

Show available NFS shares:

```bash
showmount -e <ip>
```

Mounting a NFS share:

```bash
mkdir target-NFS
sudo mount -t nfs 10.129.14.128:/ ./target-NFS/ -o nolock
cd target-NFS
tree .
```

We can also use this access to find the usernames and groups that own the files. We can then take these usernames, groups, UIDs, and GUIDs on our local system and adapt them to the NFS share to view and modify files.

```bash
ls -l mnt/nfs/ # List files with Usernames and Group Names
ls -n mnt/nfs/ # List files with UIDs and GUIDs
```

After we take all necessary action, unmount:

```bash
sudo umount ./target-NFS
```

## DNS

Several types of DNS Servers:

| **Server Type**                | **Description**                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `DNS Root Server`              | The root servers of the DNS are responsible for the top-level domains (`TLD`). As the last instance, they are only requested if the name server does not respond. Thus, a root server is a central interface between users and content on the Internet, as it links domain and IP address. The [Internet Corporation for Assigned Names and Numbers](https://www.icann.org/) (`ICANN`) coordinates the work of the root name servers. There are `13` such root servers around the globe. |
| `Authoritative Nameserver`     | Authoritative name servers hold authority for a particular zone. They only answer queries from their area of responsibility, and their information is binding. If an authoritative name server cannot answer a client's query, the root name server takes over at that point. Based on the country, company, etc., authoritative nameservers provide answers to recursive DNS nameservers, assisting in finding the specific web server(s).                                              |
| `Non-authoritative Nameserver` | Non-authoritative name servers are not responsible for a particular DNS zone. Instead, they collect information on specific DNS zones themselves, which is done using recursive or iterative DNS querying.                                                                                                                                                                                                                                                                               |
| `Caching DNS Server`           | Caching DNS servers cache information from other name servers for a specified period. The authoritative name server determines the duration of this storage.                                                                                                                                                                                                                                                                                                                             |
| `Forwarding Server`            | Forwarding servers perform only one function: they forward DNS queries to another DNS server.                                                                                                                                                                                                                                                                                                                                                                                            |
| `Resolver`                     | Resolvers are not authoritative DNS servers but perform name resolution locally in the computer or router.                                                                                                                                                                                                                                                                                                                                                                               |

![Diagram showing domain hierarchy: Root, Top Level Domains (TLD) like net, org, com, dev, io; Second Level Domain inlanefreight.com; Sub-Domains dev.inlanefreight.com, www.inlanefreight.com, mail.inlanefreight.com; Host WS01.dev.inlanefreight.com.](https://academy.hackthebox.com/storage/modules/27/tooldev-dns.png)

Different DNS Records and their uses:

| **DNS Record** | **Description**                                                                                                                                                                                                                                   |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `A`            | Returns an IPv4 address of the requested domain as a result.                                                                                                                                                                                      |
| `AAAA`         | Returns an IPv6 address of the requested domain.                                                                                                                                                                                                  |
| `MX`           | Returns the responsible mail servers as a result.                                                                                                                                                                                                 |
| `NS`           | Returns the DNS servers (nameservers) of the domain.                                                                                                                                                                                              |
| `TXT`          | This record can contain various information. The all-rounder can be used, e.g., to validate the Google Search Console or validate SSL certificates. In addition, SPF and DMARC entries are set to validate mail traffic and protect it from spam. |
| `CNAME`        | This record serves as an alias for another domain name. If you want the domain www.hackthebox.eu to point to the same IP as hackthebox.eu, you would create an A record for hackthebox.eu and a CNAME record for www.hackthebox.eu.               |
| `PTR`          | The PTR record works the other way around (reverse lookup). It converts IP addresses into valid domain names.                                                                                                                                     |
| `SOA`          | Provides information about the corresponding DNS zone and email address of the administrative contact.                                                                                                                                            |

Find person responsible for operation of domain:

```bash
dig soa <domain>
```

### Default Configuration

All servers have 3 different types of configuration files

1. Local DNS config
2. Zone files
3. Reverse name resolution files

The DNS server [Bind9](https://www.isc.org/bind/) is often used on Linux. Local config file is **named.conf**

Find local DNS Configuration:

```bash
cat /etc/bind/named.conf.local
```

A `zone file` is a text file that describes a DNS zone with the BIND file format. In other words it is a point of delegation in the DNS tree.

There must be precisely one `SOA` record and at least one `NS` record. The SOA resource record is usually located at the beginning of a zone file. The main goal of these global rules is to improve the readability of zone files.

Search a zone file:

```bash
cat /etc/bind/db.domain.com
```

For the `Fully Qualified Domain Name` (`FQDN`) to be resolved from the IP address, the DNS server must have a reverse lookup file. In this file, the computer name (`FQDN`) is assigned to the last octet of an IP address, which corresponds to the respective host, using a PTR record. The PTR records are responsible for the reverse translation of IP addresses into names, as we have already seen in the above table.

Search Reverse Name Resolution Zone file:

```bash
cat /etc/bind/db.10.129.14
```

### Dangerous Settings

[List of BIND9 server vulnerabilities](https://www.cvedetails.com/product/144/ISC-Bind.html?vendor_id=64) (CVEDetails)

[SecurityTrails popular DNS server attacks](https://web.archive.org/web/20250329174745/https://securitytrails.com/blog/most-popular-types-dns-attacks)

| **Option**        | **Description**                                                                |
| ----------------- | ------------------------------------------------------------------------------ |
| `allow-query`     | Defines which hosts are allowed to send requests to the DNS server.            |
| `allow-recursion` | Defines which hosts are allowed to send recursive requests to the DNS server.  |
| `allow-transfer`  | Defines which hosts are allowed to receive zone transfers from the DNS server. |
| `zone-statistics` | Collects statistical data of zones.                                            |

### Footprinting the Service

We query using the NS record and specification of our desired server using the `@` character

```bash
dig ns <domain> @<ip>
```

We can also query a DNS server's version using a class CHAOS query and type TXT. It has to exist as a record, though.

```bash
dig CH TXT version.bind <ip>
```

The ANY option can view all available records (that the server is willing to disclose)

```bash
dig any <domain> @<ip>
```

`Zone transfer` refers to the transfer of zones to another server in DNS, which generally happens over TCP port 53. This procedure is abbreviated `Asynchronous Full Transfer Zone` (`AXFR`).

Synchronization between the servers involved is realized by zone transfer. Using a secret key `rndc-key`, which we have seen initially in the default configuration, the servers make sure that they communicate with their own master or slave.

Dig AXFR Zone Transfer:

```bash
dig axfr <domain> @<ip>
```

If the administrator used a subnet for the `allow-transfer` option for testing purposes or as a workaround solution or set it to `any`, everyone would query the entire zone file at the DNS server. In addition, other zones can be queried, which may even show internal IP addresses and hostnames.

Dig Internal AXFR Zone Transfer:

```bash
dig axfr internal.<domain> @<ip>
```

We can brute-force using individual A records. [List of possible hostnames](https://github.com/danielmiessler/SecLists/blob/master/Discovery/DNS/subdomains-top1million-5000.txt) to help send requests in order

```bash
for sub in $(cat /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt);do dig $sub.<domain> @<ip> | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt;done
```

[DNSenum](https://github.com/fwaeytens/dnsenum) is a tool that can help with this

```bash
dnsenum --dnsserver <ip> --enum -p 0 -s 0 -o subdomains.txt -f /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt <domain>
```

## SMTP

Defaults to port 25, newer servers also use ports such as TCP 587

Unencrypted, sends all commanda data, and auth info in plain text. Used with SSL/TLS to prevent unauthorized reading

Most modern servers use ESMTP with SMTP-Auth

The SMTP client is known as the Mail User Agent (MUA)

When MUA converts into header and body, the Mail Transfer Agent (MTA) checks email and stores it

A Mail Submission Agent (MSA) (aka Relay server) Checks the validity/origin of email.

<figure><img src="../.gitbook/assets/Screenshot 2026-07-29 at 12.06.47 PM.png" alt=""><figcaption></figcaption></figure>

### Default Configuration

Get default config:

```bash
cat /etc/postfix/main.cf 
```

Sending and communication are done by special commands

| **Command**  | **Description**                                                                                  |
| ------------ | ------------------------------------------------------------------------------------------------ |
| `AUTH PLAIN` | AUTH is a service extension used to authenticate the client.                                     |
| `HELO`       | The client logs in with its computer name and thus starts the session.                           |
| `MAIL FROM`  | The client names the email sender.                                                               |
| `RCPT TO`    | The client names the email recipient.                                                            |
| `DATA`       | The client initiates the transmission of the email.                                              |
| `RSET`       | The client aborts the initiated transmission but keeps the connection between client and server. |
| `VRFY`       | The client checks if a mailbox is available for message transfer.                                |
| `EXPN`       | The client also checks if a mailbox is available for messaging with this command.                |
| `NOOP`       | The client requests a response from the server to prevent disconnection due to time-out.         |
| `QUIT`       | The client terminates the session.                                                               |

We can use telnet to initialize a TCP connection with a SMTP server. From there, we can run the commands above

```bash
telnet <ip> 25
```

The VRFY command can enumerate existing users on the server, but does not always work depending on the server's configuration

If you have to work through a web proxy, the way to get the proxy to connect to the SMTP server would look like `CONNECT <ip>:25 HTTP/1.0`

### Dangerous Settings

**Open Relay Configuration**

```shell-session
mynetworks = 0.0.0.0/0
```

With this setting, this SMTP server can send fake emails and thus initialize communication between multiple parties. Another attack possibility would be to spoof the email and read it.

### Footprinting the Service

Nmap has a `smtp-commands` script that usees EHL0 command

```bash
sudo nmap <ip> -sC -sV -p25
```

We can use the `smtp-open-relay` script to identify the server as an open relay

```bash
sudo nmap <ip> -p25 --script smtp-open-relay -v
```

## IMAP/POP3

Connnection established via port 143

By default, ports `110` and `995` are used for POP3, and ports `143` and `993` are used for IMAP. The higher ports (`993` and `995`) use TLS/SSL to encrypt the communication between the client and server.

### Default Configuration

Both IMAP and POP3 have a large number of configuration options, making it difficult to deep dive into each component in more detail. If you wish to examine these protocol configurations deeper, we recommend creating a VM locally and install the two packages `dovecot-imapd`, and `dovecot-pop3d` using `apt` and play around with the configurations and experiment.

IMAP Commands

| **Command**                     | **Description**                                                                                               |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| `1 LOGIN username password`     | User's login.                                                                                                 |
| `1 LIST "" *`                   | Lists all directories.                                                                                        |
| `1 CREATE "INBOX"`              | Creates a mailbox with a specified name.                                                                      |
| `1 DELETE "INBOX"`              | Deletes a mailbox.                                                                                            |
| `1 RENAME "ToRead" "Important"` | Renames a mailbox.                                                                                            |
| `1 LSUB "" *`                   | Returns a subset of names from the set of names that the User has declared as being `active` or `subscribed`. |
| `1 SELECT INBOX`                | Selects a mailbox so that messages in the mailbox can be accessed.                                            |
| `1 UNSELECT INBOX`              | Exits the selected mailbox.                                                                                   |
| `1 FETCH <ID> all`              | Retrieves data associated with a message in the mailbox.                                                      |
| `1 CLOSE`                       | Removes all messages with the `Deleted` flag set.                                                             |
| `1 LOGOUT`                      | Closes the connection with the IMAP server.                                                                   |

POP3 Commands

| **Command**     | **Description**                                             |
| --------------- | ----------------------------------------------------------- |
| `USER username` | Identifies the user.                                        |
| `PASS password` | Authentication of the user using its password.              |
| `STAT`          | Requests the number of saved emails from the server.        |
| `LIST`          | Requests from the server the number and size of all emails. |
| `RETR id`       | Requests the server to deliver the requested email by ID.   |
| `DELE id`       | Requests the server to delete the requested email by ID.    |
| `CAPA`          | Requests the server to display the server capabilities.     |
| `RSET`          | Requests the server to reset the transmitted information.   |
| `QUIT`          | Closes the connection with the POP3 server.                 |

### Dangerous Settings

| **Setting**               | **Description**                                                                           |
| ------------------------- | ----------------------------------------------------------------------------------------- |
| `auth_debug`              | Enables all authentication debug logging.                                                 |
| `auth_debug_passwords`    | This setting adjusts log verbosity, the submitted passwords, and the scheme gets logged.  |
| `auth_verbose`            | Logs unsuccessful authentication attempts and their reasons.                              |
| `auth_verbose_passwords`  | Passwords used for authentication are logged and can also be truncated.                   |
| `auth_anonymous_username` | This specifies the username to be used when logging in with the ANONYMOUS SASL mechanism. |

### Footprinting the Service

Basic Nmap scan for IMAP/POP3

```bash
sudo nmap 10.129.14.128 -sV -p110,143,993,995 -sC
```

Once we get access credentials for an employee, you can log in and read messages

<pre class="language-bash"><code class="lang-bash">curl -k 'imaps://10.129.14.128' --user user:p4ssw0rd

# * LIST (\HasNoChildren) "." Important
<strong># * LIST (\HasNoChildren) "." INBOX
</strong></code></pre>

We can use openssl and ncat to interact over SSL

```bash
openssl s_client -connect <ip>:pop3s
openssl s_client -connect <ip>:imaps
```

## SNMP

The current version is `SNMPv3`

SNMP also transmits control commands using agents over UDP port `161`

SNMP also enables the use of so-called `traps` over UDP port `162`

### MIB

The `Management Information Base` (`MIB`) is a text file in which all queryable SNMP objects of a device are listed in a standardized tree hierarchy. It contains at least one `Object Identifier` (`OID`).

MIB files are written in the `Abstract Syntax Notation One` (`ASN.1`) based ASCII text format.

### OID

An OID represents a node in a hierarchical namespace

Many nodes in the OID tree contain nothing except references to those below them. The OIDs consist of integers and are usually concatenated by dot notation. We can look up many MIBs for the associated OIDs in the [Object Identifier Registry](https://www.alvestrand.no/objectid/).

### SNMPv1

Still used in many small networks

Supports grabbing info from devices, configuration of devices, provides traps which are notifications of events

**No build in authentication** system

Does **not** support encryption

### SNMPv2

Has different versions, today's version is v2c (c means community-based)

The community string that provides security is only transmitted in plain text, so no built-in encryption

### SNMPv3

Now has user/pass auth and transmission encryption

Way more configuration options than v2c

### Community Strings

Passwords used to determine if info can be viewed or not

### Default Configuration

The default configuration of the SNMP daemon defines the basic settings for the service, which include the IP addresses, ports, MIB, OIDs, authentication, and community strings.

Found at `/etc/snmp/snmpd.conf`

Way too much configuration, check the manpage

### Dangerous Settings

| **Settings**                                     | **Description**                                                                       |
| ------------------------------------------------ | ------------------------------------------------------------------------------------- |
| `rwuser noauth`                                  | Provides access to the full OID tree without authentication.                          |
| `rwcommunity <community string> <IPv4 address>`  | Provides access to the full OID tree regardless of where the requests were sent from. |
| `rwcommunity6 <community string> <IPv6 address>` | Same access as with `rwcommunity` with the difference of using IPv6.                  |

### Footprinting the Service

For footprinting SNMP, we can use tools like `snmpwalk`, `onesixtyone`, and `braa`. `Snmpwalk` is used to query the OIDs with their information. `Onesixtyone` can be used to brute-force the names of the community strings since they can be named arbitrarily by the administrator. Since these community strings can be bound to any source, identifying the existing community strings can take quite some time.

SNMPwalk Example

```bash
snmpwalk -v2c -c public <ip>
```

If we do not know the community string, we can use `onesixtyone` and `SecLists` wordlists to identify these community strings.

```bash
sudo apt install onesixtyone
onesixtyone -c /opt/useful/seclists/Discovery/SNMP/snmp.txt <ip>
```

We can use the tool [crunch](https://secf00tprint.github.io/blog/passwords/crunch/advanced/en) to create custom wordlists to guess the community string

Once we have the community string, we can use [braa](https://github.com/mteg/braa) to brute-force the individual OIDs&#x20;

```bash
sudo apt install braa
braa <community string>@<IP>:.1.3.6.*
```

## MySQL

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

### Default Configuration

```bash
cat /etc/mysql/mysql.conf.d/mysqld.cnf | grep -v "#" | sed -r '/^\s*$/d'
```

### Dangerous Settings

| **Settings**       | **Description**                                                                                              |
| ------------------ | ------------------------------------------------------------------------------------------------------------ |
| `user`             | Sets which user the MySQL service will run as.                                                               |
| `password`         | Sets the password for the MySQL user.                                                                        |
| `admin_address`    | The IP address on which to listen for TCP/IP connections on the administrative network interface.            |
| `debug`            | This variable indicates the current debugging settings                                                       |
| `sql_warnings`     | This variable controls whether single-row INSERT statements produce an information string if warnings occur. |
| `secure_file_priv` | This variable is used to limit the effect of data import and export operations.                              |

The settings `user`, `password`, and `admin_address` are security-relevant because the entries are made in plain text

### Footprinting the Service

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

## MSSQL

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

### Default Configuration

On initial install, the SQL service will likely run as `NT SERVICE\MSSQLSERVER`

Authentication being set to `Windows Authentication` means that the underlying Windows OS will process the login request and use either the local SAM database or the domain controller (hosting Active Directory) before allowing connectivity to the database management system.

### Dangerous Settings

Good things to look into:

* MSSQL clients not using encryption to connect to the MSSQL server
* The use of self-signed certificates when encryption is being used. It is possible to spoof self-signed certificates
* The use of [named pipes](https://docs.microsoft.com/en-us/sql/tools/configuration-manager/named-pipes-properties?view=sql-server-ver15)
* Weak & default `sa` credentials. Admins may forget to disable this account

### Footprinting the Service

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



## Oracle TNS

The `Oracle Transparent Network Substrate` (`TNS`) server is a communication protocol that facilitates communication between Oracle databases and applications over networks. Initially introduced as part of the [Oracle Net Services](https://docs.oracle.com/en/database/oracle/oracle-database/18/netag/introducing-oracle-net-services.html) software suite, TNS supports various networking protocols between Oracle databases and client applications, such as `IPX/SPX` and `TCP/IP` protocol stacks

Has been updated to support IPv6 and SSL/TLS which allows for:

* Name resolution
* Connection management
* Load balancing
* Security

### Default Configuration

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

## IPMI

[Intelligent Platform Management Interface](https://www.thomas-krenn.com/en/wiki/IPMI_Basics) (`IPMI`) is a set of standardized specifications for hardware-based host management systems used for system management and monitoring. It acts as an autonomous subsystem and works independently of the host's BIOS, CPU, firmware, and underlying operating system

IPMI is typically used in three ways:

* Before the OS has booted to modify BIOS settings
* When the host is fully powered down
* Access to a host after a system failure

To function, IPMI requires the following components:

* Baseboard Management Controller (BMC) - A micro-controller and essential component of an IPMI
* Intelligent Chassis Management Bus (ICMB) - An interface that permits communication from one chassis to another
* Intelligent Platform Management Bus (IPMB) - extends the BMC
* IPMI Memory - stores things such as the system event log, repository store data, and more
* Communications Interfaces - local system interfaces, serial and LAN interfaces, ICMB and PCI Management Bus

### Footprinting the Service

Communicates over UDP port 623

Systems that use the IPMI protocol are called Baseboard Management Controllers (BMCs). BMCs are typically implemented as embedded ARM systems running Linux, and connected directly to the host's motherboard

Nmap scan using ipmi-version script to footprint:

```bash
sudo nmap -sU --script ipmi-version -p 623 <ip> (ex: ilo.inlanfreight.local)
```

We can also use the Metasploit scanner module [IPMI Information Discovery (auxiliary/scanner/ipmi/ipmi\_version)](https://www.rapid7.com/db/modules/auxiliary/scanner/ipmi/ipmi_version/).

```
use auxiliary/scanner/ipmi/ipmi_version
```

During internal penetration tests, we often find BMCs where the administrators have not changed the default password. Some unique default passwords to keep in our cheatsheets include:

| Product         | Username      | Password                                                                  |
| --------------- | ------------- | ------------------------------------------------------------------------- |
| Dell iDRAC      | root          | calvin                                                                    |
| HP iLO          | Administrator | randomized 8-character string consisting of numbers and uppercase letters |
| Supermicro IPMI | ADMIN         | ADMIN                                                                     |

### Dangerous Settings

If default credentials do not work to access a BMC, we can turn to a [flaw](https://web.archive.org/web/20260421071724/http://fish2.com/ipmi/remote-pw-cracking.html) in the RAKP protocol in IPMI 2.0. During the authentication process, the server sends a salted SHA1 or MD5 hash of the user's password to the client before authentication takes place

These password hashes can then be cracked offline using a dictionary attack using `Hashcat` mode `7300`. In the event of an HP iLO using a factory default password, we can use this Hashcat mask attack command `hashcat -m 7300 ipmi.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u` which tries all combinations of upper case letters and numbers for an eight-character password

To retrieve IPMI hashes, we can use the Metasploit [IPMI 2.0 RAKP Remote SHA1 Password Hash Retrieval](https://www.rapid7.com/db/modules/auxiliary/scanner/ipmi/ipmi_dumphashes/) module.

```
use auxiliary/scanner/ipmi/ipmi_dumphashes 
```
