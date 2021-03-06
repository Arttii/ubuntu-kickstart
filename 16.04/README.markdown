# Kickstart configs for Ubuntu 16.04

- **Status**: stable

This folder contains kickstart configs designed for Ubuntu 16.04

## Usage

- Make any modifications you wish to the config.
    - the initial username and password
    - if you setup an `apt-cache-ng` server, uncomment those lines and update the server addres.  Remember it's specified in two places.  Once towards the top which is used by installer and one in `%post` section which sets it up to be used in final system
    - alter the partition layout if you wish
    - add any more packages you wish to install
- Create the VM just make sure the drive is at least 10GB
    - for vmware users, you can create the network interface using `vmxnext3` as that is supported in the kernel
- Attach the OS ISO to cdrom and make sure to activate it on boot
- Startup VM
- You'll first be prompted to choose language, go ahead and choose something and hit enter
- Press `F6` and then hit `ESC` and this will bring you to boot line.
- At the end of the line add `ks=http://your-server.example.com/ks-1604minimalvm.cfg` or whatever you named file.
- Press `ENTER` to start the installation

## Configuration

Here's what gets setup based on the distro and specific kickstart file you use.

### ks-1604minimalvm.cfg

- Intended to be used when creating a virtual machine
- Requires minimum 10GB disk
- Initial user is `ubuntu` with password `ChangeMe`
- Installs the following
    - openssl
    - ca-certificates
    - wget
    - curl
    - man
    - openssh-server
    - vim
    - latest version of git using [Ubuntu Git Maintainer's PPA](https://launchpad.net/~git-core/+archive/ubuntu/ppa)
- I install git from the PPA since I generally want the latest version of git and that PPA is managed by the same people that manage the git package in Ubuntu so I trust them.
- It's important to note that python v2.7 is not installed. This lines up with Ubuntu's announcement that 16.04 will not have python v2.7 installed by default.  All python stuff that ubuntu uses has been ported to python v3. Of course v2.7 is available but you will need to add `python2.7` package to list if you want it installed by default.
- Set vim background to dark
- Change default umask from `022` to `027`, meaning files created are not world readable
- Turns off installation of recommended packages
- Enables automatic security updates
- All partitions are `EXT4`
- All partitions except `/boot` are in a LVM
- After install, with 10GB disk, leaves about 2GB left to allocate to whatever partition you want using `lvextend`
- Creates several partitions.  Reason I made so many is to hopefully be left with as little stuff in `/` partition as possible.  Notably I don't create a `/tmp` partition.  If you are removing partitions I highly recommend keeping `/home` and `/var/log` since a lot of times those can fill up fast.  Depending on what you are using it for and where it gets installed you may want to create a `/opt` or `/srv`.  Although I would recommend creating a second drive for those.  Then if something is messed up you can remount that drive somewhere else and get the data.  I also add various options to mount.  I add `noatime` to all since that just creates extra writes.  Then add `nodev` and `noexec` where I can for security.  See below for partition layout.

| mount    | size  | options                |
| -------- | ----- | ---------------------- |
| /boot    | 512MB | `noatime,nodev`        |
| /        | 1GB   | `noatime`              |
| /usr     | 2GB   | `noatime,nodev`        |
| /var     | 1.5GB | `noatime,nodev`        |
| /var/log | 512MB | `noatime,nodev,noexec` |
| /home    | 512MB | `noatime,nodev`        |
| swap     | 2GB   |                        |

- Output from `df -h` and `vgs`

```
Filesystem               Size  Used Avail Use% Mounted on
udev                     365M     0  365M   0% /dev
tmpfs                     75M  1.5M   73M   2% /run
/dev/mapper/vg0-root     945M  138M  742M  16% /
/dev/mapper/vg0-usr      1.9G  426M  1.4G  24% /usr
tmpfs                    371M     0  371M   0% /dev/shm
tmpfs                    5.0M     0  5.0M   0% /run/lock
tmpfs                    371M     0  371M   0% /sys/fs/cgroup
/dev/mapper/vg0-var      1.4G   26M  1.3G   2% /var
/dev/mapper/vg0-home     465M  2.3M  434M   1% /home
/dev/sda1                464M   29M  407M   7% /boot
/dev/mapper/vg0-var_log  465M   18M  420M   4% /var/log
```

```
  VG   #PV #LV #SN Attr   VSize VFree
  vg0    1   6   0 wz--n- 9.52g 2.37g
```

