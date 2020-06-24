## pt-align 要解决问题
pt-align 用于把文本按列对齐之后输出，还能过滤掉注释。

![sqlpy](static/2020-26/sqlpy-pt-align.jpg)

---


## 使用效果
我电脑上的 `/etc/fstab` 的内容如下。
```bash
cat /etc/fstab 

#
# /etc/fstab
# Created by anaconda on Wed Jun 24 13:43:55 2020
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=95b2c407-6d89-411a-96e3-b92fd5cdeb1b /boot                   xfs     defaults        0 0
/dev/mapper/centos-home /home                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0
```

可以看到并没有对齐，并且还有若干行注释。

```bash
cat /etc/fstab  | pt-align 
/dev/mapper/centos-root                   /     xfs  defaults 0 0
UUID=95b2c407-6d89-411a-96e3-b92fd5cdeb1b /boot xfs  defaults 0 0
/dev/mapper/centos-home                   /home xfs  defaults 0 0
/dev/mapper/centos-swap                   swap  swap defaults 0 0
```

---