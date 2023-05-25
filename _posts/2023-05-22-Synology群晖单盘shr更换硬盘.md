---
layout:     post
title:      Synology | 群晖单盘shr更换硬盘
subtitle:   群晖单盘shr更换硬盘
date:       2023-05-22
author:     bin
header-img: 
catalog: true
tags:
    - Synology
    - 群晖
    - 硬盘


---



开始之前建议将储存池进行一次**数据清理**



## 正文



先把新硬盘插入空余盘位，进入dsm，将新硬盘添加进要更换的单盘shr中（储存池三个点-添加硬盘），等待完成添加硬盘，一天左右大概，

添加完成后关机，

将要更换的旧硬盘取下（新硬盘插入到旧硬盘原先的位置，此操作可有可无好像）

开机，控制面板停止哔声，进入ssh使用root登陆

查看所有已创建raid：

```
cat /proc/mdstat
```

raid序号是按顺序排列的：

```
md6 : active raid1 sata5p5[1]
      13661650816 blocks super 1.2 [2/1] [_U]
```

其中`sata5p5`即为包含第五盘位硬盘的raid，或其后的[_U] 和[2/1]辨别，下划线就是缺少一硬盘

例如将md6从双盘shr设置为单盘shr：

```
mdadm --grow --raid-devices=1 --force /dev/md6
```

dsm储存管理器中会实时更新。



#### 更改分区表以扩容



此时虽然已更换硬盘成功且恢复为单盘shr，但是如果更换的硬盘容量与之前的不同（更大），需要手动更改分区表以扩容

查看磁盘的名字：

```
ls
```

或

```
ls/dev/sata*
```





此时可能会有红色的报错文本：

```
GPT PMBR size mismatch will be corrected by write
```

需要修复：

```
parted -l
```

```
Fix
```

此时再次执行`ls`或`fdisk -l`应该就可以看到问题解决不会报错了





查看需要更改的硬盘的分区使用情况：

```
parted /dev/sata5
```

再输入`p`查看，显示分区表后会看到，系统可能自动把最后一部分（比之前硬盘大的那部分容量空间）自动创建一个分区，，此时需要把这个最后无用的分区删除掉以释放空间：

```
rm 5
```

其中`5`为需要删除的分区编码



调整分区大小：

```
parted /dev/sata5 resizepart 5 100%
```























<br>

<br>


---

## END

