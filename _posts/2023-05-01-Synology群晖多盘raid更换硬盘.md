---
layout:     post
title:      Synology | 群晖多盘raid更换硬盘
subtitle:   群晖多盘raid更换硬盘
date:       2023-05-01
author:     bin
header-img: 
catalog: true
tags:
    - Synology
    - 群晖
    - 硬盘


---



## 视频版：



<video width="100%" height="auto" controls>
    <source src="https://file.notion.so/f/s/ff8be3f8-0cbb-49f9-aafd-45f5fc6004ca/NAS%E5%AE%B9%E9%87%8F%E5%91%8A%E6%80%A5%EF%BC%9F%E8%B7%9F%E7%9D%80%E8%A7%86%E9%A2%91%E6%95%99%E7%A8%8B%E8%B5%B0%EF%BC%8C%E5%87%A0%E6%AD%A5%E6%8D%A2%E4%B8%8A%E6%96%B0%E7%A1%AC%E7%9B%98%EF%BC%81.mp4?id=971b6b88-c57e-42f1-866b-8c5506e7eb8e&table=block&spaceId=41eb4144-a3d5-46c2-84db-61dfb5c17755&expirationTimestamp=1683040327991&signature=eqHtZHXf40MN2u3ZHW4vIlWeVv89H_aD2InzhHRSnhs&downloadName=NAS%E5%AE%B9%E9%87%8F%E5%91%8A%E6%80%A5%EF%BC%9F%E8%B7%9F%E7%9D%80%E8%A7%86%E9%A2%91%E6%95%99%E7%A8%8B%E8%B5%B0%EF%BC%8C%E5%87%A0%E6%AD%A5%E6%8D%A2%E4%B8%8A%E6%96%B0%E7%A1%AC%E7%9B%98%EF%BC%81.mp4" type="video/mp4">
</video>



## 文字版：



开始之前建议将储存池进行一次**数据清理**



<center><b><i><font size="6.5">01</font></i>&nbsp;<font face="SimSun" size="4.5">有空余硬盘位时raid更换硬盘</font></b></center>



###### **有空余硬盘位时**

-将**新硬盘**直接插入空余盘位中。

-前往 DSM 中的**存储管理器**页面，单击要扩充的存储池右上角的三点图标，选择**更换硬盘**。

<a href="https://imgbox.com/4RgerSse" target="_blank"><img src="https://images2.imgbox.com/21/60/4RgerSse_o.png" alt="image host"/></a>



-选择 RAID 中**容量最小的硬盘**，然后将其从存储池中移除。

<a href="https://imgbox.com/tgCUT5UY" target="_blank"><img src="https://images2.imgbox.com/84/ff/tgCUT5UY_o.png" alt="image host"/></a> 



-选择要添加到存储池的新硬盘，勾选**扩充存储空间容量**。（这一步会将新硬盘中的所有数据擦除，请在知晓后单击确定。）

<a href="https://imgbox.com/FkF6mdRE" target="_blank"><img src="https://images2.imgbox.com/f5/68/FkF6mdRE_o.png" alt="image host"/></a> 

<a href="https://imgbox.com/fH497dEd" target="_blank"><img src="https://images2.imgbox.com/74/8c/fH497dEd_o.png" alt="image host"/></a>



-等待添加完成即可。



<center><b><i><font size="6.5">02</font></i>&nbsp;<font face="SimSun" size="4.5">没有空余硬盘位时</font></b></center>



###### **没有空余硬盘位时**

-在 DSM 中前往**存储管理器**页面，点击 **HDD**，选择 RAID 中**容量最小的硬盘**，单击**动作** > **停用硬盘**。

-将 NAS 关机，移除第一步中停用的硬盘，然后安装一个**容量相同或更大的新硬盘**。

-重启 NAS后，再次前往**存储管理器** >**HDD**，确认系统能够识别新硬盘。

-该存储池现在应处于已降级状态，单击存储池右上角的更多，然后选择**修复**。
-选择上一步中安装的新硬盘，将其添加到存储池中。这一步修复阵列所需要较长，请耐心等待。（请注意硬盘中的所有数据都会被擦除。）

-重复上述过程直至所有需要的硬盘均更换为更大容量的硬盘即可。



<center><b><i><font size="6.5">03</font></i>&nbsp;<font face="SimSun" size="4.5">单盘位机型</font></b></center>



###### ***\*单盘位机型\****

-准备一块**容量不小于需要转移数据总量的移动硬盘**，并连接到 NAS。

-安装并打开 **Hyper Backup**，选择备份目标**本地文件夹和 USB**，创建备份目的地为移动硬盘，选择需要转移的共享文件夹与应用程序，其他设置保持默认即可。耐心等待程序备份完成。

-将 NAS 关机后，拔出旧硬盘并插入容量更大的新硬盘。

-启动 NAS 并安装 DSM 操作系统，进入系统后安装并打开 Hyper Backup。

-单击还原图标 ，选择**数据**，选择**从已有存储库还原**，还原来源选择**本地文件夹和 USB**。

-继续按照配置向导选择想要还原的系统配置、文件夹和应用程序。然后耐心等待还原完成即可。









<br>

<br>


---

## END

