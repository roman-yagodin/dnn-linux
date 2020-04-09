# DNN Linux Development Environment [DRAFT]

Yes, you could use Linux (or other "nixes") as development environment for the DNN Platform!

Things you'll need:

- VirtualBox (or any other) VM with DNN installed
- Visual Studio Code and/or MonoDevelop IDE - to do some coding
- Mono - to build the projects
- Some MSBuild/PowerShell scripts to copy build results to the "network" share
- Some scripts to easily start/stop VM and control share mounting

## Share mounting

### Mounting DNN share via /etc/fstab

The DNN share should be configured first by adding entry to the `/etc/fstab` file.

The following `/etc/fstab` entry will allow to mount DNN share (from any external machine or VM) to the local folder:

```
# DNN share
//192.168.56.101/dnn	/home/user/mnt/dnn	cifs	noauto,vers=3.0,iocharset=utf8,rw,user,uid=1000,gid=1000,credentials=/home/user/mnt/dnn.smbclient	0	0
```

there:

- `//192.168.56.101/dnn` - network path to the share
- `/home/user/mnt/dnn` - path to the local mount point
- `uid=1000,gid=1000` - UID and GID of local user to which we allow access (see `id` command output)
- `dnn.smbclient` - credentials file

### Mounting shares via `mount` command

Without `/etc/fstab` entry we would use `sudo mount`, not just `mount`:

```Shell
sudo mount -t cifs -rw -o vers=3.0,iocharset=utf8,username=Administrator,password=MyP@$_$w0rD,uid=$(id -u),gid=$(id -g) //192.168.56.101/dnn "/home/user/mnt/dnn"
```

### Mounting shares via GVFS (obsolete)

Mounting shares via GVFS will not work (at least for now), cause `System.IO.File.Copy` (and so MSBuild and PowerShell) have problems to write to the GVFS-mounted share (shame).

```Shell
gvfs-mount "smb://WORKGROUP;Administrator@192.168.56.101/dnn"
```
