title: AWS EC2 挂载磁盘
tags:
- AWS
---

## 挂载磁盘
在创建 `AWS` 的 `EC2` 实例时如果添加了额外的磁盘则需要手动挂载到系统中。

首先运行 `lsblk` 来查看可用的块设备：

```sh
NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0          7:0    0 22.2M  1 loop /snap/amazon-ssm-agent/5657
loop1          7:1    0   49M  1 loop /snap/core18/2406
loop2          7:2    0 57.8M  1 loop /snap/core20/1498
loop3          7:3    0 38.7M  1 loop /snap/snapd/15909
loop4          7:4    0 71.8M  1 loop /snap/lxd/22927
nvme1n1      259:0    0    8G  0 disk
nvme0n1      259:1    0    8G  0 disk
├─nvme0n1p1  259:2    0  7.9G  0 part /
└─nvme0n1p15 259:3    0   99M  0 part /boot/efi
```

其中的 `nvme1n1` 是本次新添加的磁盘，目前还未挂载到系统中，而 `nvme0n1` 则是根设备并且有两个分区。

> `lsblk` 的输出结果会移除设备路径中的 `/dev/` 前缀，所以设备 `nvme1n1` 的完整路径为 `/dev/nvme1n1`。

然后，我们需要在 `nvme1n1` 之上创建文件系统才能使用，执行 `sudo file -s /dev/nvme1n1` 显示 `nvme1n1` 还没有文件系统：

```sh
/dev/nvme1n1: data
```

而如果我们查看 `sudo file -s /dev/nvme0n1` 则会显示：

```sh
/dev/nvme0n1: DOS/MBR boot sector; partition 1 : ID=0xee, start-CHS (0x0,0,2), end-CHS (0x3ff,255,63), startsector 1, 16777215 sectors, extended partition table (last)
```

执行 `sudo mkfs -t xfs /dev/nvme1n1` 来为 `nvme1n1` 创建文件系统，其中 `xfs` 表示文件系统的类型：

```sh
meta-data=/dev/nvme1n1           isize=512    agcount=8, agsize=262144 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=0 inobtcount=0
data     =                       bsize=4096   blocks=2097152, imaxpct=25
         =                       sunit=1      swidth=1 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=1 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

接着，我们就可以创建一个文件夹用来挂载磁盘，例如 `sudo mkdir /data`。最后将 `/dev/nvme1n1` 挂载到 `/data` 上：

```sh
sudo mount /dev/nvme1n1 /data
```

此时如果查看 `df -h` 就会包含 `/dev/nvme1n1`：

```sh
Filesystem       Size  Used Avail Use% Mounted on
/dev/root        7.6G  1.6G  6.1G  21% /
tmpfs            926M     0  926M   0% /dev/shm
tmpfs            371M 1000K  370M   1% /run
tmpfs            5.0M     0  5.0M   0% /run/lock
/dev/nvme0n1p15   98M  5.1M   93M   6% /boot/efi
tmpfs            186M  4.0K  186M   1% /run/user/1000
/dev/nvme1n1     8.0G   90M  8.0G   2% /data
```

## 系统启动自动挂载磁盘
当前的磁盘挂载信息会在系统启动后丢失，如果希望系统启动后自动挂载磁盘则需要向 `/etc/fstab` 中添加一条记录。

安全起见先备份下 `/etc/fstab`：

```sh
sudo cp /etc/fstab /etc/fstab.orig
```

然后运行 `sudo blkid` 来查看设备 `/dev/nvme1n1` 的 `UUID`：

```sh
/dev/nvme0n1p1: LABEL="cloudimg-rootfs" UUID="15ea47e1-ef7d-4928-9dbe-ffaf0e743653" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="1957f80e-a338-441c-a0e0-ed1575eefda3"
/dev/nvme0n1p15: LABEL_FATBOOT="UEFI" LABEL="UEFI" UUID="68E7-1A63" BLOCK_SIZE="512" TYPE="vfat" PARTUUID="1eeb08ab-0afd-4477-bd53-4389a42db8f6"
/dev/loop1: TYPE="squashfs"
/dev/loop4: TYPE="squashfs"
/dev/loop2: TYPE="squashfs"
/dev/loop0: TYPE="squashfs"
/dev/nvme1n1: UUID="aa81c000-325c-40b7-ba4c-598ec2c824e0" BLOCK_SIZE="512" TYPE="xfs"
/dev/loop3: TYPE="squashfs"
```

最后向 `/etc/fstab` 添加一条记录：

```sh
UUID=aa81c000-325c-40b7-ba4c-598ec2c824e0  /data  xfs  defaults,nofail  0  2
```

可以通过先取消挂载 `/data` 即 `sudo umount /data` 然后再执行 `sudo mount -a` 来验证自动挂载是否生效。

## 参考
* [Make an Amazon EBS volume available for use on Linux](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-using-volumes.html)