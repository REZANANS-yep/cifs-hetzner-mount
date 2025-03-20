# Tutorial goal
Many users encounter an issue where the network environment displays the error "The Host Is Down." In this tutorial, we use the auto-reconnect option when making a request, which helps avoid this issue.

# Debian 12 guide
**Mounting the Hetzner Storage Box as a CIFS (Samba) Network Share in Debian:**
1. **Open the `/etc/fstab` File:**  
   Use your preferred text editor (for example, `nano` or `vim`).
2. **Add the Following Line at the End of the File:**  
   ```bash
   //username-of-box.your-storagebox.de/username-of-box /mnt/storagebox cifs iocharset=utf8,rw,credentials=/etc/smbcredentials/username-of-box,vers=3.0,noauto,x-systemd.automount,x-systemd.idle-timeout=30,_netdev,uid=0,gid=0,file_mode=0660,dir_mode=0770 0 0
   ```  
   _Note #1:_ Replace `username-of-box` with your actual username. 
   
   _Note #2:_ Enter your login and password of storage box at /etc/smbcredentials/username-of-box
3. **Save the File and Reload the Systemd Daemon:**  
   Execute the following command:  
   ```bash
   sudo systemctl daemon-reload
   ```
4. **Check your mounted box:**  
   You can now use the `/mnt/storagebox` directory to work with your storage.

Below is a detailed technical breakdown of the fstab entry:

```
 //username-of-box.your-storagebox.de/username-of-box /mnt/storagebox cifs iocharset=utf8,rw,credentials=/etc/smbcredentials/username-of-box,vers=3.0,noauto,x-systemd.automount,x-systemd.idle-timeout=30,_netdev,uid=0,gid=0,file_mode=0660,dir_mode=0770 0 0
```

2.1 Example of credentials file at `/etc/smbcredentials/u111111`
```bash
username=u111111
password=1234512345
```

# Technical insight
- **Remote Share:**  
  `//username-of-box.your-storagebox.de/username-of-box`  
  - This is the network path to the CIFS (Samba) share provided by Hetzner.  
  - The first part (`//username-of-box.your-storagebox.de`) indicates the server address, and the second part (`username-of-box`) is the share name on that server.

- **Local Mount Point:**  
  `/mnt/storagebox`  
  - The directory on your local system where the remote share will be mounted.

- **Filesystem Type:**  
  `cifs`  
  - Specifies that the share is mounted using the Common Internet File System (CIFS) protocol, commonly used for Samba shares.

- **Mount Options:**  
  These are comma-separated options that control the behavior of the mount:
  
  - **`iocharset=utf8`:**  
    Sets the character encoding to UTF-8 for proper handling of file names.
  
  - **`rw`:**  
    Mounts the share with read and write permissions.
  
  - **`credentials=/etc/smbcredentials/username-of-box`:**  
    Points to a file containing the username and password required to access the share. This keeps credentials separate from the fstab file.
  
  - **`vers=3.0`:**  
    Specifies the version of the SMB protocol to use (version 3.0 in this case).
  
  - **`noauto`:**  
    Prevents the share from being mounted automatically at boot time. It will instead be mounted on demand.
  
  - **`x-systemd.automount`:**  
    Instructs systemd to automatically mount the share when it is accessed.
  
  - **`x-systemd.idle-timeout=30`:**  
    Tells systemd to unmount the share after 30 seconds of inactivity.
  
  - **`_netdev`:**  
    Indicates that the filesystem resides on a network device, ensuring that the system waits for network availability before attempting to mount.
  
  - **`uid=0` and `gid=0`:**  
    Sets the owner and group of the mounted files to the user and group with ID 0 (typically the root user).
  
  - **`file_mode=0660`:**  
    Sets the permission bits for files, allowing read and write permissions for the owner and group.
  
  - **`dir_mode=0770`:**  
    Sets the permission bits for directories, allowing read, write, and execute permissions for the owner and group.

- **Dump and Filesystem Check Options:**  
  `0 0` arguments  
  - The first `0` disables the dump utility (used for backups).  
  - The second `0` indicates that the filesystem should not be checked at boot time by `fsck`.

# Performance of CIFS
- At now CIFS is most fast solution for your backups, because its not have any encryption by default.

# Security TIPs (Hetzner only)
- External reachability should be not enabled, Hetzner Storage box can work without it at local network at highest speed possible, if you plan to mount CIFS at not hetzner server you should enable it. 
- Create new user for backup at linux with restricted access, its additional layer of security, uid=0 and gid=0: (change it to id of user)
- You can't use SSH keys at CIFS Debian 12, so make sure file with credentials is secured.

# Credits
Hetzner offical documentation: https://docs.hetzner.com/storage/storage-box/access/access-samba-cifs/
