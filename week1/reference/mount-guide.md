# How to mount a device in Linux

Before installing and loading the database, you can mount devices to store the database files or log files. In this guide, we will *separate data and log files on separate devices*. Placing both DATA AND (transaction) LOG files on the same device can cause contention for that device, resulting in poor performance. Also, placing the log files on a separate device ensures full recovery when the data device crashes.

1. First, list available partitions on your system. And check the device names (e.g., `/dev/nvme0n1`, `/dev/sda`) to mount:

```bash
$ sudo fdisk -l
Disk /dev/nvme0n1: 953.9 GiB, 1024209543168 bytes, 2000409264 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x0b0f2e65

Device         Boot Start        End    Sectors   Size Id Type
/dev/nvme0n1p1       2048 2000409263 2000407216 953.9G 83 Linux


Disk /dev/sda: 238.5 GiB, 256060514304 bytes, 500118192 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x55525fc3
...
```

2. If you need to create a partition, enter command mode:

```bash
$ sudo fdisk /dev/sda

Welcome to fdisk (util-linux 2.27.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help):
```

3. Enter `n` to create a new partition:

```bash
Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): 

Using default response p.
Partition number (1-4, default 1): 
First sector (2048-500118191, default 2048): 
Last sector, +sectors or +size{K,M,G,T,P} (2048-500118191, default 500118191): 

Created a new partition 1 of type 'Linux' and of size 238.5 GiB.
```

4. Then, enter `w` to write the changes you've made to disk:

```bash
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

5. Now, we need to create a filesystem (e.g., EXT4) on the partition:

```bash
$ sudo mkfs.ext4 /dev/sda1
mke2fs 1.42.13 (17-May-2015)
/dev/sda1 contains a ext4 file system
        last mounted on /home/mijin/test_data/pg_xlog on Wed Jul 31 11:51:22 2019
Proceed anyway? (y,n) y
Discarding device blocks: done                            
Creating filesystem with 62514518 4k blocks and 15630336 inodes
Filesystem UUID: 085b8c6a-8dec-4fba-88f1-dabd18527a2e
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
        4096000, 7962624, 11239424, 20480000, 23887872

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```

6. In this example, we will use `/dev/nvme0n1` for the data device, and `/dev/sda` for the log device. So, repeat *STEP 2 ~ 5* for `/dev/nvme0n1`. Then, let's mount the date device first:

```bash
$ mkdir test_data
$ sudo mount /dev/nvme0n1p1 test_data
$ sudo chown -R yourUsername:yourUsername test_data
```

You need to change `/dev/nvme0n1p1` to the partition name of your data device and `yourUsername` to your user name.

7. Then, mount the log device in the same way:

```bash
$ mkdir test_log
$ sudo mount /dev/sda1 -o nobarrier test_log
$ sudo chown -R yourUsername:yourUsername test_log
```

Likewise, you need to change `/dev/sda1` to the partition name of your log device and `yourUsername` to your user name. In the case of the log device, we turned off the *write barrier* option to mitigate the overhead of `fsync()`. The detailed reasons are as follows:

> A **write barrier** is a kernel mechanism used to ensure that file system metadata is correctly written and ordered on persistent storage, even when storage devices with volatile write caches lose power. File systems with write barriers enabled also ensure that data transmitted via `fsync()` is persistent throughout a power loss.
> However, enabling write barriers incurs a substantial performance penalty for some applications. Specifically, applications that use `fsync()` heavily or create and delete many small files will likely run much slower.
> For devices with non-volatile, battery-backed write caches and those with write-caching disabled, you can safely disable write barriers at mount time using the `-o nobarrier` option for mount.

8. You can check mounted devices with the below command:

```bash
$ mount
...
/dev/nvme0n1p1 on /home/mijin/test_data type ext4 (rw,relatime,data=ordered)
/dev/sda1 on /home/mijin/test_log type ext4 (rw,relatime,nobarrier,data=ordered)
...
```