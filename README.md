# About vm-scripts

VM control scripts for [cross-platform web] development environment

## Initial setup

- Install (TODO)
- Make your changes and rename the `__settings.template` file to just `__settings`
- Configure the mount point for the "network" share (see details in the **Share mounting** section below)

## General use

The `start-vm` script will start your VM by restoring its state from current (latest) snapshot and mounts its share to the local folder.
Generally this means that you will have it up and running in seconds.

Now you can do your coding session - by editing files in the share directly or by auto-deploying build results from IDE into it.

After that you would want to invoke the `stop-vm` script - it will unmount the share and shutdown VM, but take the snapshot of its current state first.

The `remount-share` script should be used for troubleshooting - if by any reason the mounted share become irresponsible.

The scripts are using `notify-send` to provide desktop notifications about errors.

From time to time you'll need to manually delete old VM snapshots.

## Share mounting

Following examples presume that the share is located on a VM with some kind of modern Windows OS as a guest,
and development is going on the Linux host, but this can be adopted to Linux-to-Linux scenarios almost as-is.

### Mounting SAMBA share via `/etc/fstab`

To allow easy mounting, the share should be configured first by adding entry to the `/etc/fstab` file.

The following `/etc/fstab` entry will allow you to mount share (from external machine or VM) to the local folder:

```
//192.168.56.101/share-name	/home/user/mnt/share-name	cifs	noauto,vers=3.0,iocharset=utf8,rw,user,uid=1000,gid=1000,credentials=/home/user/mnt/share-name.smbclient	0	0
```

there:

- `//192.168.56.101/share-name` - network path to the share
- `/home/user/mnt/share-name` - path to the local mount point
- `uid=1000,gid=1000` - UID and GID of local user to which we allow access (see `id` command output)
- `/home/user/mnt/share-name.smbclient` - credentials file

### Mounting share via `sudo mount`

Without `/etc/fstab` entry we will be enforced to use `sudo mount`, not just `mount` command. This way we will need to enter a password
every time we mounting the share - and what's the reason behind using `/etc/fstab` entry method described above.

```Shell
sudo mount -t cifs -rw -o vers=3.0,iocharset=utf8,username=Administrator,password=MyP@$_$w0rD,uid=$(id -u),gid=$(id -g) //192.168.56.101/share-name "/home/user/mnt/share-name"
```

### Note about mounting shares via GVFS

Mounting shares via GVFS will not work for you (at least for now) if you use .NET stack to automate deploy your build files to the share.
This is because `System.IO.File.Copy` (and so MSBuild and PowerShell scripts) have problems to write to the GVFS-mounted share - what a shame!

But if you use different technology stack to deploy your build files to the share, it can still be a viable option.
