---
layout:     post
title:      Synology | 群晖DSM7.1_M.2_NVME固态硬盘创建存储池
subtitle:   使用机型DS1621+
date:       2023-04-23
author:     bin
header-img: 
catalog: true
tags:
    - Synology
    - 群晖
    - DSM
    - M.2_NVME
    - 存储池

---

# 前言：

**DS1621+在DSM7.2之前M.2口的ssd官方默认情况下是只能用作缓存使用的，升级到7.2之后支持M.2_NVME硬盘创建存储池了（本次新增支持M.2_NVME硬盘创建存储池的机型：DS1522+、DS1621+、DS1821+、DS1621xs+），但是官方限制了只能使用官方指定的群晖官方推出的M.2_NVME固态硬盘（例如：SNV3410-800G、SNV3410-400G的NVMe PCIe3.0x4 M.2 2280固态硬盘）。**

本文也是参考网上许多成功和失败案例之后，自己动手执行后记录的

开始之前先引用[spacer大佬](https://www.chiphell.com/forum.php?mod=viewthread&tid=2187138)对于群晖1621+，1821+之类的多盘位机型的几点说明和建议：

- **不要犹豫，内存加满：**群晖的内存 swap 既充当了读缓存，所以不要犹豫，加到机型支持的最大内存。一般是 16X2= 32GB，买 ECC 的内存。注意部分内存有兼容性问题，大家可以回帖帮助群友买到合适内存。
- **PCI-e 优先万兆，不要搞 SSD 缓存：**如果你的机型不支持新出的[E10M20-T1](https://www.synology.cn/zh-cn/products/E10M20-T1) SSD&万兆二合一卡，那 pci-e 优先加万兆网卡，为什么？往上看缓存的坑。
- **如果盘位富裕，1号盘位建议用 sata SSD 代替：**群晖的机制是默认从1号盘位开始读系统，然后 app 的缓存都是默认放在存储空间 1 内的。通常情况下，HDD 做的 RAID 随机读写性能很一般。建议如果盘位够的用户（比如 1618+,1819+）之类的，可以考虑放个 SSD 到 1 号盘位，设定为basic存储空间，用于套件及及各类缓存。2-8 号盘位做 HDD RAID（RAID 6），这样的话，在日常使用，能显著提升系统的响应效能，比 nvme 缓存效率要高很多。



# 正文：

**执行以下操作之前如果ssd是正在用做缓存的，记得要在dsm按照步骤删除缓存盘**



## 群晖DSM7.1使用M.2_NVME固态硬盘创建存储池有两种方法

分别为：

- **纯ssh命令创建储存池**

- **使用脚本将现有M.2_NVME固态硬盘添加进群晖的ssd兼容列表后在群晖系统GUI界面直接执行创建储存池**



## 纯ssh命令创建储存池

1. 使用管理员账号临时获得-i权限（`sudo -i`）或直接使用root账号通过SSH登陆群晖

2. 找出nvme设备：

   ```
   ls /dev/nvme*
   ```

   会显示：

   <a href="https://imgbox.com/4TiMImYZ" target="_blank"><img src="https://thumbs2.imgbox.com/03/f7/4TiMImYZ_t.png" alt="image host" width="600px" align="left"/></a> 

   <div style="clear:both;"></div>

   或

   <a href="https://imgbox.com/dk5JPG7d" target="_blank"><img src="https://thumbs2.imgbox.com/1c/6c/dk5JPG7d_t.png" alt="image host" width="600px" align="left"/></a>
   
   <div style="clear:both;"></div>

   区别在于系统是否自动创建过一个后缀为`n1p1`的分区，好像是无所谓

   群晖插入两条NVME一般都会显示`/dev/nvme0n1`和`/dev/nvme1n1`

   这里准备将1号插槽上的NVME作为储存空间传健储存池，所以本次只用到`/dev/nvme0n1`

   

3. 创建分区

   先查看1号插槽上的NVME（`/dev/nvme0n1`）的磁盘信息：

   ```
   fdisk -l /dev/nvme0n1
   ```

   将1号插槽的NVME（`/dev/nvme0n1`）分区：

   ```
   synopartition --part /dev/nvme0n1 12
   ```

   然后会询问是否继续，输入`Y`确认执行分区操作

   分区完之后再检查一下分区布局：

   ```
   fdisk -l /dev/nvme0n1
   ```

   <a href="https://imgbox.com/89ysPTT6" target="_blank"><img src="https://thumbs2.imgbox.com/5c/49/89ysPTT6_t.png" alt="分区布局" width="200px" align="left"/></a>
   
   <div style="clear:both;"></div>
   
   此处应该就可以看到有三个分区，第三个空间最大的分区`/dev/nvme0n1p3`就是要用做储存  空间的分区
   
   
   
4. 创建RAID阵列（储存池）：

   因为这里只用一个SSD做为储存空间，所以这里会创建`Basic`储存池
   首先查看当前RAID：

   ```
   cat /proc/mdstat
   ```

   ```
   Personalities : [raid1] 
   md6 : active raid1 sata5p5[0]
         13661650816 blocks super 1.2 [1/1] [U]
         
   md5 : active raid1 sata4p5[0]
         13661650816 blocks super 1.2 [1/1] [U]
         
   md3 : active raid1 sata3p5[0] sata2p5[1]
         15615154816 blocks super 1.2 [2/2] [UU]
         
   md2 : active raid1 sata1p5[0]
         3896291584 blocks super 1.2 [1/1] [U]
         
   md1 : active raid1 sata1p2[0] sata5p2[4] sata4p2[3] sata3p2[2] sata2p2[1]
         2097088 blocks [6/5] [UUUUU_]
         
   md0 : active raid1 sata1p1[0] sata5p1[4] sata4p1[3] sata3p1[2] sata2p1[1]
         8388544 blocks [6/5] [UUUUU_]
   ```

   其中`md0`是系统空间，`md1`是交换空间（此二项为分布在所有已插入群晖的所有硬盘上以虚拟raid1的形式存在的），从`md2`开始为本人自行创建的储存池，`md2`为第一盘位的sata口ssd（此处显示raid1，实际为shr），`md3`为第二第三盘位的两个hdd组成的shr（此处也显示为raid1），……，此时，正常情况下理应按顺序以`md7`标识为NVME创建新的储存池

   --

   创建新的RAID，使用md7作为标识：

   ```
   mdadm --create /dev/md7 --level=1 --raid-devices=1 --force /dev/nvme0n1p3
   ```

   其中`level=1`在`raid-devices=1`即只有一块硬盘创建raid的时候创建的为`Basic`，两个及以上硬盘创建的为raid1，`/dev/nvme0n1p3`为刚刚检查磁盘分区布局后看到的那个第三个空间最大的分区

   然后会询问是否继续，输入`Y`确认执行创建新的RAID操作

   

5. 创建文件系统

   在刚刚创建的`md7`RAID上建立`btrfs`或`ext4`文件系统（**强烈建议使用btrfs**：[什么是btrfs](https://www.synology.cn/zh-cn/dsm/Btrfs)）：

   ```
   mkfs.btrfs -f /dev/md7
   ```

   或

   ```
   mkfs.ext4 -F /dev/md4
   ```

   

6. 重启群晖

   ```
   reboot
   ```

   

7. 群晖重启完成后在储存管理器中就能看到刚刚创建的储存池了，但是显示错误，需要手动点击**`在线重组`**完成最后的创建，再执行后续创建储存空间的操作



## 使用脚本将现有M.2_NVME固态硬盘添加进群晖的ssd兼容列表

本方法使用脚本为：

<a title="https://github.com/007revad/Synology_M2_volume" href="https://github.com/007revad/Synology_M2_volume"  target="_blank" rel="noopener noreferrer">Synology_M2_volume</a>（https://github.com/007revad/Synology_M2_volume）

此脚本可以非常方便的启用M.2_NVME作为存储，而且是在GUI界面操作不是命令行手动创建，只需运行脚本并键入`yes`和 `1`、`2`、`3` 或 `4` 来回答一些简单的问题。然后重新启动，转到存储管理器、在线组装和创建卷即可

**本脚本注意事项：**

如果您想更新DSM，您的 M.2 硬盘显示为不受支持的硬盘存储池显示为丢失，并且提示存储损毁，您需要重新运行[Synology_HDD_db脚本](https://github.com/007revad/Synology_HDD_db)。Synology_HDD_db脚本应要在每次 DSM 更新后重新运行。

**支持的 RAID 级别：**

| RAID Level      | Drives Required  |
| --------------- | ---------------- |
| Single（basic） | 1 drive          |
| RAID 0          | 2 or more drives |
| RAID 1          | 2 or more drives |
| RAID 5          | 3 or more drives |

--

1. 使用管理员账号临时获得-i权限（`sudo -i`）或直接使用root账号通过SSH登陆群晖

2. 把下载的脚本上传到群晖，执行脚本：

   ```
   sudo -i /volume1/public/script/syno_create_m2_volume.sh
   ```

   `/volume1/public/script/syno_create_m2_volume.sh`为脚本所在目录

   脚本选项

   ```
   -a, --all        List all M.2 drives even if detected as active/列出所有M.2设备，即使检测为活动驱动器
   -s, --steps      Show the steps to do after running this script/显示运行此脚本后要执行的步骤
   -h, --help       Show this help message/显示帮助信息
   -v, --version    Show the script version/显示脚本版本
   ```




3. **运行脚本后做什么：**

   1. 重新启动 Synology NAS。
   2. 转到存储管理器并选择在线组装：
      - 存储池 > 可用池 > 在线组装
   3. 像往常一样创建卷：
      - 选择新的存储池 > 创建 > 创建卷。
      - 设置分配的大小。
        - （可选）输入卷描述。有创意:)
      - 点击下一步。
      - 选择文件系统（Btrfs 或 ext4）并单击下一步。
      - （可选）启用加密此卷并单击下一步。
        - 创建加密密码或输入您现有的加密密码。
      - 确认您的设置并单击应用以完成创建 M.2 卷。
   4. 可选择启用和安排 TRIM：
      - 存储池 > ... > 设置 > SSD TRIM
      - **注意：DSM 7.1.1。没有针对 M.2 存储池的 SSD TRIM 设置**
      - **注意：DSM 7.2 Beta 没有针对 M.2 RAID 0 或 RAID 5 的 SSD TRIM 设置**











<br>

<br>

---

参考文章：

<a href="https://www.chiphell.com/thread-2395056-1-1.html" target="_blank">Chiphell - neomelb - 继续折腾群晖DS1821+，NVMe SSD除了做缓存竟然还能这么玩！</a>

<a href="https://www.chiphell.com/thread-2187138-1-1.html" target="_blank">Chiphell - spacer - 专门开一贴，群晖硬软件的的各种坑及解决方案</a>

<a href="https://www.bilibili.com/video/BV1za411S7x6/?vd_source=ed256bd486922278484767bd6cbe80c9" target="_blank">bilibili - VedioTalk - 直通NVME硬盘给群晖7.1当存储，黑白群晖均适用</a>



---

<br>


---

## END

