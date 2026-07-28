---
description: Server Message Block
---

# SMB

Protocol to regulate access to files, directories, and resources such as printers and router)

Just Windows systems, Samba is used on Linux and Unix

Both sides must establish a connection, done over TCP

## Samba

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



## Default Configuration

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

## Dangerous Settings

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

## Footprinting the Service

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

