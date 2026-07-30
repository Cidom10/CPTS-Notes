# Linux RMPs

## SSH

Standard TCP port 22

The well-known [OpenBSD SSH](https://www.openssh.com/) (`OpenSSH`) server on Linux distributions is an open-source fork of the original and commercial `SSH` server from SSH Communication Security. Accordingly, there are two competing protocols: `SSH-1` and `SSH-2`

`SSH-2`, also known as SSH version 2, is a more advanced protocol than SSH version 1 in encryption, speed, stability, and security. For example, `SSH-1` is vulnerable to `MITM` attacks, whereas SSH-2 is not

OpenSSH has six different authentication methods:

1. Password authentication
2. Public-key authentication
3. Host-based authentication
4. Keyboard authentication
5. Challenge-response authentication
6. GSSAPI authentication

### Public Key Authentication

The server sends its `public host key` to the client, which the client uses to verify the server's identity. Only when contact is first established there is a risk of a third party interposing itself between the two participants and thus intercepting the connection. A `host key` cannot be imitated because it is a unique public-private `key pair`, and an attacker cannot forge the private key’s signature without access to it, assuming the client properly verifies the public key against a trusted source

### Default Configuration

The [sshd\_config](https://www.ssh.com/academy/ssh/sshd_config) file, responsible for the OpenSSH server, has only a few of the settings configured by default. However, the default configuration includes X11 forwarding, which contained a command injection vulnerability in version 7.2p1 of OpenSSH in 2016

```bash
cat /etc/ssh/sshd_config
```

### Dangerous Settings

| **Setting**                  | **Description**                             |
| ---------------------------- | ------------------------------------------- |
| `PasswordAuthentication yes` | Allows password-based authentication.       |
| `PermitEmptyPasswords yes`   | Allows the use of empty passwords.          |
| `PermitRootLogin yes`        | Allows to log in as the root user.          |
| `Protocol 1`                 | Uses an outdated version of encryption.     |
| `X11Forwarding yes`          | Allows X11 forwarding for GUI applications. |
| `AllowTcpForwarding yes`     | Allows forwarding of TCP ports.             |
| `PermitTunnel`               | Allows tunneling.                           |
| `DebianBanner yes`           | Displays a specific banner when logging in. |

### Footprinting the Service

One of the tools we can use to fingerprint the SSH server is [ssh-audit](https://github.com/jtesta/ssh-audit). It checks the client-side and server-side configuration and shows some general information and which encryption algorithms are still used by the client and server

```bash
git clone https://github.com/jtesta/ssh-audit.git && cd ssh-audit
./ssh-audit.py 10.129.14.132
```

Check OpenSSH version for vulns

Change Authentication Method

```bash
ssh -v user@<ip>
```

For potential brute-force attacks, we can specify the authentication method with the SSH client option `PreferredAuthentications`.

```bash
ssh -v user@<ip> -o PreferredAuthentications=password
```

## Rsync

[Rsync](https://linux.die.net/man/1/rsync) is a fast and efficient tool for locally and remotely copying files

By default, it uses port `873` and can be configured to use SSH for secure file transfers by piggybacking on top of an established SSH server connection

This [guide](https://hacktricks.wiki/en/network-services-pentesting/873-pentesting-rsync.html) covers some of the ways Rsync can be abused, most notably by listing the contents of a shared folder on a target server and retrieving files

Scanning for Rsync

```bash
sudo nmap -sV -p 873 <ip>
```

Probing for Accessible Shares

```bash
nc -nv <ip> 873
(UNKNOWN) [<ip>] 873 (rsync) open
@RSYNCD: 31.0
@RSYNCD: 31.0
#list
dev            	Dev Tools
@RSYNCD: EXIT
```

Enumerating an Open Share

```bash
rsync -av --list-only rsync://<ip>/dev
```

If Rsync is configured to use SSH to transfer files, we could modify our commands to include the `-e ssh` flag, or `-e "ssh -p2222"` if a non-standard port is in use for SSH. This [guide](https://phoenixnap.com/kb/how-to-rsync-over-ssh) is helpful for understanding the syntax for using Rsync over SSH

## R-Services

R-Services are a suite of services hosted to enable remote access or issue commands between Unix hosts over TCP/IP

`R-services` span across the ports `512`, `513`, and `514` and are only accessible through a suite of programs known as `r-commands`

The [R-commands](https://en.wikipedia.org/wiki/Berkeley_r-commands) suite consists of the following programs:

* rcp (`remote copy`)
* rexec (`remote execution`)
* rlogin (`remote login`)
* rsh (`remote shell`)
* rstat
* ruptime
* rwho (`remote who`)

| **Command** | **Service Daemon** | **Port** | **Transport Protocol** | **Description**                                                                                                                                                                                                                                                            |
| ----------- | ------------------ | -------- | ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `rcp`       | `rshd`             | 514      | TCP                    | Copy a file or directory bidirectionally from the local system to the remote system (or vice versa) or from one remote system to another. It works like the `cp` command on Linux but provides `no warning to the user for overwriting existing files on a system`.        |
| `rsh`       | `rshd`             | 514      | TCP                    | Opens a shell on a remote machine without a login procedure. Relies upon the trusted entries in the `/etc/hosts.equiv` and `.rhosts` files for validation.                                                                                                                 |
| `rexec`     | `rexecd`           | 512      | TCP                    | Enables a user to run shell commands on a remote machine. Requires authentication through the use of a `username` and `password` through an unencrypted network socket. Authentication is overridden by the trusted entries in the `/etc/hosts.equiv` and `.rhosts` files. |
| `rlogin`    | `rlogind`          | 513      | TCP                    | Enables a user to log in to a remote host over the network. It works similarly to `telnet` but can only connect to Unix-like hosts. Authentication is overridden by the trusted entries in the `/etc/hosts.equiv` and `.rhosts` files.                                     |

Trusted hosts found at `/etc/hosts.equiv`

Scanning for R-Services

```bash
sudo nmap -sV -p 512,513,514 <ip>
```

### Access Control & Trusted Relationships

The primary concern for `r-services`, and one of the primary reasons `SSH` was introduced to replace it, is the inherent issues regarding access control for these protocols

By default, these services utilize [Pluggable Authentication Modules (PAM)](https://web.archive.org/web/20241102161436/https://debathena.mit.edu/trac/wiki/PAM) for user authentication onto a remote system; however, they also bypass this authentication through the use of the `/etc/hosts.equiv` and `.rhosts` files on the system

The `hosts.equiv` and `.rhosts` files contain a list of hosts (`IPs` or `Hostnames`) and users that are `trusted` by the local host when a connection attempt is made using `r-commands`. Entries in either file can appear like the following:

```
cat .rhosts

htb-student     10.0.17.5
+               10.0.17.10
+               +
```

Logging in using Rlogin

```bash
rlogin <ip> -l <account>
```

Once successfully logged in, we can also abuse the `rwho` command to list all interactive sessions on the local network by sending requests to the UDP port 513

```
rwho

root     web01:pts/0 Dec  2 21:34
htb-student     workstn01:tty1  Dec  2 19:57  2:25 
```

From this information, we can see that the `htb-student` user is currently authenticated to the `workstn01` host, whereas the `root` user is authenticated to the `web01` host. We can use this to our advantage when scoping out potential usernames to use during further attacks on hosts over the network.&#x20;

However, the `rwho` daemon periodically broadcasts information about logged-on users, so it might be beneficial to watch the network traffic.

Listing Authenticated Users Using Rusers

```
rusers -al <ip>

htb-student     10.0.17.5:console          Dec 2 19:57     2:25
```

