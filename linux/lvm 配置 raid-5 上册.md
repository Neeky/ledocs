## 概要
最近遇到几次磁盘损坏了，每次损坏都要重新从备份中还原数据再配置复制，感觉再用 raid-0 我就要 GG 了。 我现在开始想用 raid-5 了。 这个博客记录下怎么用 lvm 配置 raid-5。

![lvm-raid-5](static/2021-01/lvm-raid-5.jpg)

---


## 环境介绍
主机上有三块 10G 的盘，现在打算用这三块盘做一个 raid-5 的逻辑卷。
```bash
lsblk
NAME                       MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                          8:0    0   20G  0 disk 
├─sda1                       8:1    0    1G  0 part /boot
└─sda2                       8:2    0   19G  0 part 
  ├─cl_centos--studio-root 253:0    0   17G  0 lvm  /
  └─cl_centos--studio-swap 253:1    0    2G  0 lvm  [SWAP]
sdb                          8:16   0   10G  0 disk 
sdc                          8:32   0   10G  0 disk 
sdd                          8:48   0   10G  0 disk
```

---

## 第一步 做物理卷
1、把三块盘做成物理卷。
```bash
pvcreate /dev/sdb /dev/sdc /dev/sdd
  Physical volume "/dev/sdb" successfully created.
  Physical volume "/dev/sdc" successfully created.
  Physical volume "/dev/sdd" successfully created.
```
2、检查。
```bash
pvscan 
  PV /dev/sdb                          lvm2 [10.00 GiB]
  PV /dev/sdc                          lvm2 [10.00 GiB]
  PV /dev/sdd                          lvm2 [10.00 GiB
```
---

## 第二步 做卷组
1、创建一个叫 vg_mysql 的卷组。
```bash
vgcreate vg_mysql /dev/sdb /dev/sdc /dev/sdd
  Volume group "vg_mysql" successfully created
```
2、检查。
```bash
vgdisplay vg_mysql
  --- Volume group ---
  VG Name               vg_mysql
  System ID             
  Format                lvm2
  Metadata Areas        3
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                3
  Act PV                3
  VG Size               <29.99 GiB
  PE Size               4.00 MiB
  Total PE              7677
  Alloc PE / Size       0 / 0   
  Free  PE / Size       7677 / <29.99 GiB
  VG UUID               iDMfbd-dih1-etdH-VZ5d-eO20-HhqB-3SYQPb
```
---

## 第三步 做逻辑卷
1、要创建一个 raid-5 的逻辑卷，也非常简单直接加上 --type=raid5 就行了。
```bash
lvcreate -n lv_mysql -l 100%FREE --type=raid5 vg_mysql 
  Using default stripesize 64.00 KiB.
  Rounding size (7677 extents) down to stripe boundary size (7676 extents)
  Logical volume "lv_mysql" created.
```
2、检查。
```bash
lvdisplay vg_mysql/lv_mysql
  --- Logical volume ---
  LV Path                /dev/vg_mysql/lv_mysql
  LV Name                lv_mysql
  VG Name                vg_mysql
  LV UUID                Tlcvqv-9LcY-QgXH-jtZ1-FA2B-PGRz-zeQLnj
  LV Write Access        read/write
  LV Creation host, time centos-studio, 2021-03-14 23:31:53 +0800
  LV Status              available
  #open                  0
  LV Size                19.98 GiB
  Current LE             5116
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     768
  Block device           253:8
```

---

## 第四步 格式化逻辑卷与挂载
1、格式化为 XFS 文件系统。
```bash
mkfs.xfs /dev/vg_mysql/lv_mysql 
meta-data=/dev/vg_mysql/lv_mysql isize=512    agcount=16, agsize=327408 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1
data     =                       bsize=4096   blocks=5238528, imaxpct=25
         =                       sunit=16     swidth=32 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=16 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```
2、挂载。
```bash
mount /dev/vg_mysql/lv_mysql /mnt
```
3、检查。
```bash
df -h
文件系统                            容量  已用  可用 已用% 挂载点
devtmpfs                            379M     0  379M    0% /dev
tmpfs                               396M     0  396M    0% /dev/shm
tmpfs                               396M  5.7M  391M    2% /run
tmpfs                               396M     0  396M    0% /sys/fs/cgroup
/dev/mapper/cl_centos--studio-root   17G  1.6G   16G    9% /
/dev/sda1                           976M  128M  782M   15% /boot
tmpfs                                80M     0   80M    0% /run/user/1000
/dev/mapper/vg_mysql-lv_mysql        20G  176M   20G    1% /mnt
```
---

## 总结
1、由于我们使用的是 raid-5 所以会损失 1/3 的空间。这个也就是为什么 3 个物理卷总共 30G ，我们做出来的逻辑卷只有 20G 的原因。

2、raid-5 模式下支持坏一块盘(数据不丢)。

---