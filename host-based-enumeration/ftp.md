# FTP

## FTP

FTP runs through port 21 for establishing a connnection, then comms pass through port 20

In **active FTP**, client establishes connection in port 21 and informs the server which port they can send their response through. If client has a firewall, this causes problems.

In **passive FTP**, server announces a port. Since client initiates this connection, firewall does not block

[Huge list of FTP commands](https://web.archive.org/web/20230326204635/https://www.smartfile.com/blog/the-ultimate-ftp-commands-list/)

[FTP Server Return Codes](https://en.wikipedia.org/wiki/List_of_FTP_server_return_codes)



## TFTP (Trivial FTP)

Simpler, but does not provide user auth and other features that FTP does

Uses UDP



## Default Configuration

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

## Dangerous Settings

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

## Footprinting the Service

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
