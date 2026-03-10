# NFS Enumeration and Access via Nmap NSE

## Overview

During the enumeration phase of a penetration test, an NFS service was discovered on the target system.  
Using Nmap and its NSE scripts, the exported NFS shares were identified and further analyzed.  
The shares were then mounted locally to inspect their contents.

Note: The target IP address has been anonymized as <TARGET_IP>.

---

# 1. Initial Service Discovery

The first step was identifying open ports and running services.

Command:

sudo nmap <TARGET_IP> -p111,2049 -sV -sC

Result:

PORT     STATE SERVICE VERSION
111/tcp  open  rpcbind 2-4 (RPC #100000)
2049/tcp open  nfs_acl 3 (RPC #100227)

RPC enumeration revealed multiple RPC programs related to NFS:

| Program | Service | Description |
|--------|--------|-------------|
| 100000 | rpcbind | RPC service mapper |
| 100003 | nfs | Network File System |
| 100005 | mountd | NFS mount daemon |
| 100021 | nlockmgr | File locking |
| 100227 | nfs_acl | Access control |

The presence of **mountd** indicated that the system exports NFS shares.

---

# 2. NFS Enumeration using Nmap NSE

To gather more information about the NFS service, Nmap NFS scripts were executed.

Command:

sudo nmap --script nfs* <TARGET_IP> -sV -p111,2049

The scripts revealed exported directories.

Example result:

/var/nfs
/mnt/nfsshare

Additional filesystem information was also returned:

Filesystem     Used       Available
/var/nfs       3330480    506336
/mnt/nfsshare  3330480    506336

The NSE script was also able to list files stored in the exported directories.

Example:

Volume /var/nfs
flag.txt

Volume /mnt/nfsshare
flag.txt

---

# 3. Verifying NFS Exports

To confirm the exported directories, the `showmount` utility was used.

Command:

showmount -e <TARGET_IP>

Example output:

Export list for <TARGET_IP>

/var/nfs
/mnt/nfsshare

---

# 4. Mounting the NFS Share

A local directory was created to mount the share.

mkdir target-NFS

The NFS share was mounted using:

sudo mount -t nfs <TARGET_IP>:/ ./target-NFS -o nolock

Navigate into the mounted directory:

cd target-NFS

---

# 5. Inspecting the Share

Once mounted, the contents of the share can be inspected.

ls -l

Example structure:

mnt/
var/

File permissions can also be inspected using numeric UID/GID values:

ls -n

This allows identification of:

- user IDs (UID)
- group IDs (GID)
- potential privilege escalation paths

---

# 6. Root Squash Consideration

If the NFS export uses the `root_squash` option, root privileges from the client machine are mapped to a non-privileged user (usually `nobody`).

This prevents the root user from modifying files on the NFS share.

---

# 7. Potential Privilege Escalation via NFS

Misconfigured NFS shares may lead to privilege escalation.

Possible scenario:

1. Attacker gains access to the system via SSH as a low privileged user.
2. A writable NFS share is available.
3. A binary with the SUID bit is placed on the share.
4. The binary executes with elevated privileges.

This technique depends on the NFS configuration (for example if `no_root_squash` is enabled).

---

# 8. Unmounting the Share

After the analysis was completed, the share was safely unmounted.

sudo umount ./target-NFS

---

# Conclusion

NFS enumeration using Nmap NSE scripts revealed multiple exported directories on the target system.  
Mounting the shares locally allowed inspection of files and permissions.  
Misconfigured NFS shares can potentially lead to information disclosure or privilege escalation.
