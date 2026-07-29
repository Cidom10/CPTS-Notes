# NFS

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

## Dangerous Settings

| **Option**       | **Description**                                                                                                      |
| ---------------- | -------------------------------------------------------------------------------------------------------------------- |
| `rw`             | Read and write permissions.                                                                                          |
| `insecure`       | Ports above 1024 will be used.                                                                                       |
| `nohide`         | If another file system was mounted below an exported directory, this directory is exported by its own exports entry. |
| `no_root_squash` | All files created by root are kept with the UID/GID 0.                                                               |

Recommended to use VM and experiment with settings

## Footprinting the Service

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
