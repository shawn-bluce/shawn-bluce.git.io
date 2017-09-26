---
title: Linux的LVM 逻辑卷管理 分区 划分 重划
date: 2016-05-17 23:15
tags:
  - Linux
  - LVM
  - 分区
  - 格式化
---

# 0X00 LVM是什么，有什么用
LVM的全称是Logical Volume Manager（逻辑卷管理）。是Linux下的一种磁盘分区管理机制，方便给分区（逻辑分区）扩容和压缩。最简单的可以理解成原始的磁盘分区管理是单纯的给每个独立的磁盘进行分区，然后对每个分区进行管理，这样的话每次扩容和压缩空间都会很麻烦。LVM就相当于把所有磁盘的分区都揉到一起，揉成一个大磁盘或者说是大分区，然后从大的中分出小的，这样的话扩容和压缩都会变得方便。
![test](http://o7bn7vqpt.bkt.clouddn.com//article/image/lvm.jpg)
**版权声明：图片来自<a href="https://linux.cn/article-3218-2.html">Linux.cn</a>**

# 0X01 基础术语解释

PV 是*Physical Volume* **物理卷**---也就是真实的磁盘分区
VG 是*Volume Group* **卷组**---也就是好多PV组成的一个组
LV 是*Logical Volume* **逻辑卷**---就是从VG中分出来的分区
PE 是*Physical Extent* **物理区域**---是PV中最小的存储单元
LE 是*Logical Extent* **逻辑区域**---是LV中做小的存储单元

# 0X02 测试环境
V-Box 中的 CentOS 7.x 64bit
有两块或者以上数量的虚拟磁盘
磁盘大小在1GB以上
我这里/dev/sdb和/dev/sdc是刚刚添加的磁盘
root用户的~/lvm-mount用来挂载逻辑卷
使用root登陆(单纯的因为每次sudo太麻烦)

# 0X03 准备分区
使用fdisk为磁盘分区
**不会使用fdisk的可以直接按着我说的敲**
**还是建议学LVM之前掌握最基础的fdisk分区和格式化**
```bash
fdisk /dev/sdb # 使用fdisk给/dev/sdb分区
按n 回车 新建一个分区
按p 回车 选择新建分区为主分区
按 回车 选择默认分区号
按 回车 默认选择开始位置
输入 +100M 回车 选择使用100M为新分区的大小
输入 t 回车 设置分区类型
按 回车 默认选择刚才创建的分区
输入 8e 设置刚才创建的分区为 LVM 类型
```
重复上面的步骤，给/dev/sdb分出来三个区

# 0X04 创建物理卷 PV
创建物理卷的时候，可以大小不同，也可以是不同磁盘的分区，只要是 8e 类型的分区都是可以创建到物理卷中的，这里只是为了做示范
```bash
# 创建
pvcreate /dev/sdb1
pvcreate /dev/sdb2
pvcreate /dev/ddb3

# 检查
pvdisplay

# 删除 (这步不要跟着做)
pvremove /dev/sdb1
```

# 0X05 准备卷组 VG
创建一个包括/dev/sdb1 /dev/sdb2 /dev/sdb3 物理卷的卷组
命名为 volme-group1
```bash
# 创建
vgcreate volume-group1 /dev/sdb1 /dev/sdb2 /dev/sdb3

# 检查
vgdisplay

# 删除 (这步不要跟着做)
vgremove volume-group1
```

# 0X06 创建逻辑卷 LV
创建逻辑卷的时候要指定名称、大小和所属VG
```bash
# 创建
lvcreate -L 100M -n LV1 volume-group1

# 检查
lvdisplay

# 格式化 格式化成ext4类型
mkfs.ext4 /dev/volume-group1/LV1

# 挂载
mkdir ~/lvm-mount #设置一个挂载点
mount /dev/volume-group1/LV1 ~/lvm-mount # 挂载

# 删除
lvremove /dev/volume-group1/LV1
```

# 0X07 扩展LVM逻辑卷
调整逻辑卷大小是LVM最重要最有用的功能。
比如之前创建的100MB的分区不够用了，所以我们需要扩展一下那个分区的大小。虽然LVM很强大，但是扩展的时候还是需要卸载LV
```bash
# 卸载LV
umount ~/lvm-mount/

# 调整大小
lvresize -L 200M /dev/volume-group1/LV1

# 检查磁盘错误（非必须）
e2fsck -f /dev/volume-group1/LV1

# 扩展文件系统
resize2fs /dev/volume-group1/LV1

# 验证
lvdisplay
```

# 0X08 压缩LVM逻辑卷
比如你发现有一个分区给了很大，但是完全用不到，那么就可以压缩它的空间，把空余的空间用在有用的地方。
```bash
# 同样，先卸载
umount /dev/volume-group1/LV1

# 检查错误
e2fsck -f /dev/volume-group1/LV1

# 更新文件系统信息
resize2fs /dev/volume-group1/LV1 100M

# 压缩空间
lvresize -L 100M /dev/volume-group1/LV1
```
这里会弹出警告，告诉你这项操作可能会导致数据丢失，当然，一般是没有问题的

# 0X09 扩展卷组
有一天服务器的磁盘塞满了，你就新买了一块3TB的硬盘插到了电脑上，那么如何让这个3TB和之前的空间一起工作呢？我们可以把这个磁盘分区然后也放到之前的VG（卷组）中，这样通过之前的扩容功能就可以让新的3TB运用到系统中了。

```bash
# 先给新磁盘分区（参考0X03步骤）
fdisk /dev/sdc

# 然后创建PV(物理卷)
pvcreate /dev/sdc1

# 将新PV添加到VG
vgextend volume-group1 /dev/sdc1

# 验证一下
vgdisplay
```
