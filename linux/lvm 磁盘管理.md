## 背景
业务对磁盘容量的要求比较高，而单块磁解决不了，于是决定使用 lvm 把多块物理磁盘做成一块逻辑盘供业务使用。

太久没有进行这个操作了，手脚有点生疏，现在测试环境上把操作记录下来，方便以后直接用。

```bash
[root@lvmstudio ~]# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                         8:0    0  100G  0 disk 
├─sda1                      8:1    0    1G  0 part /boot
└─sda2                      8:2    0   99G  0 part 
  ├─centos_lvmstudio-root 253:0    0   50G  0 lvm  /
  ├─centos_lvmstudio-swap 253:1    0    2G  0 lvm  [SWAP]
  └─centos_lvmstudio-home 253:2    0   47G  0 lvm  /home
sdb                         8:16   0   20G  0 disk 
sdc                         8:32   0   20G  0 disk
```

![sqlpy](static/2020-29/sqlpy-lvm.jpg)

---

## 第一步 确认 lvmetad 在运行中
通过 ps 查看 lvm 相关的守护进程是否在运行。
```bash
ps -ef | grep lvmetad
root        533      1  0 11:24 ?        00:00:00 /usr/sbin/lvmetad -f
# 说明 lvmetad 在运行中
```
---

## 第二步 配置 lvmetad 开机启动
如果 lvmetad 没有在运行中那么要把它设置为开机启动并运行它。
```bash
# 开机启动
systemctl enable lvm2-lvmetad

# 运行
systemctl start lvm2-lvmetad
```

---

## 第三步 创建 pv
把整块盘做成物事卷。
```bash
pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.

pvcreate /dev/sdc                                                            
  Physical volume "/dev/sdc" successfully created.
```

---

## 第四步 创建 vg
创建卷组。
```bash
vgcreate vg_mysql /dev/sdb /dev/sdc
  Volume group "vg_mysql" successfully created
```

---

## 第五步 创建 lv
创建逻辑卷并分配 100% 的空间。
```bash
lvcreate -n lv_mysql -l 100%FREE vg_mysql
  Logical volume "lv_mysql" created.
```
---


## 第六步 检查
```bash
lvs                                                                          
  LV       VG               Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  home     centos_lvmstudio -wi-ao---- 46.99g                                                    
  root     centos_lvmstudio -wi-ao---- 50.00g                                                    
  swap     centos_lvmstudio -wi-ao----  2.00g                                                    
  lv_mysql vg_mysql         -wi-a----- 39.99g 

ll /dev/vg_mysql/lv_mysql 
lrwxrwxrwx. 1 root root 7 7月   6 11:50 /dev/vg_mysql/lv_mysql -> ../dm-3
```

---

## 第七步 格式化文件系统
在刚才的 lv 上格式化文件系统。
```bash
mkfs.ext4 /dev/vg_mysql/lv_mysql 
mke2fs 1.42.9 (28-Dec-2013)
文件系统标签=
OS type: Linux
块大小=4096 (log=2)
分块大小=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
2621440 inodes, 10483712 blocks
524185 blocks (5.00%) reserved for the super user
第一个数据块=0
Maximum filesystem blocks=2157969408
320 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
        4096000, 7962624

Allocating group tables: 完成                            
正在写入inode表: 完成                            
Creating journal (32768 blocks): 完成
Writing superblocks and filesystem accounting information: 完成
```
---

## 第八步 挂载
```bash
mkdir /database
mount /dev/vg_mysql/lv_mysql /database
```
为了下次开机还能自动挂载，还要加上 fstab 。
```
cat /etc/fstab                                                             
/dev/mapper/centos_lvmstudio-root /                       xfs     defaults        0 0
UUID=84cdb773-75cf-4683-bdb2-455ed48f2c0f /boot                   xfs     defaults        0 0
/dev/mapper/centos_lvmstudio-home /home                   xfs     defaults        0 0
/dev/mapper/centos_lvmstudio-swap swap                    swap    defaults        0 0
/dev/vg_mysql/lv_mysql/         /database                 ext4    defaults        0 0
```

---


