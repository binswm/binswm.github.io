---
layout:     post
title:      Synology | 群晖PLEX中文拼音排序
subtitle:   群晖PLEX中文拼音排序并定时自动运行
date:       2023-04-29
author:     bin
header-img: 
catalog: true
tags:
    - Synology
    - 群晖
    - PLEX


---

# 前言：

PLEX官方一直都不支持中文使用拼音首字母排序，所以有很多人都误以为中文标题排序就是没有顺序、是乱序的，其实不然，Plex 的标题是按照首字符的 Unicode 编码顺序排列的，所有语言默认使用的都是这种排序方式，对于大部分西方语言文字或者拼音文字来说，他们的文字是由字母组成的，日常就是使用字母顺序，而他们的的字母顺序和 Unicode 编码顺序是一致的，所以这样排序很正常。但是 Unicode 编码顺序对中文使用者来说基本上是毫无意义的，下图就是 GB2312 的中文字符 Unicode 排序，也就是你在 Plex 中看到的中文标题排序标准，这个顺序对中文使用者是完全起不到任何作用的。

国内的大佬<a href="https://github.com/timmy0209" target="_blank">timmy0209</a>通过自己写的脚本把上面问题都解决了，timmy0209先后发布了<a href="https://github.com/timmy0209/plex-chinese-genre" target="_blank">`plex-chinese-genre`</a>和<a href="https://github.com/timmy0209/plex-pinyin-sort" target="_blank">`plex-pinyin-sort`</a>两个脚本

之后<a href="https://github.com/sqkkyzx" target="_blank">**sqkkyzx**</a>在timmy0209的基础上把两个脚本合二为一制作了<a href="https://github.com/sqkkyzx/plex_localization_zhcn" target="_blank">`plex_localization_zhcn`</a>脚本

<blockquote >
    <p><b>PLEX 中文本地化</b></p>
    <p>实现除照片外，所有类型媒体库按标题拼音首字母排序。</p>
    <p>实现除照片外，所有类型媒体库类别标签更改中文。</p>
    <p>实现保存配置文件简化下次运行流程。</p>
    <p>plex 的中文电影默认排序是按照笔画数排的，对检索中文电影十分不友好，运行此脚本后可对指定电影库进行拼音排序，并可使用拼音首字母检索电影。</p>
</blockquote>



**所以本次使用<a href="https://github.com/sqkkyzx/plex_localization_zhcn" target="_blank">`plex_localization_zhcn`</a>脚本**



# 正文：



## 可在电脑执行脚本或者在群晖设置定时自动执行脚本



## 电脑执行

1. 下载<a href="https://www.python.org/downloads/" target="_blank">python</a>并安装

2. 安装pip（pip 是Python 的[包安装程序](https://packaging.python.org/guides/tool-recommendations/)。可以使用 pip 从[Python包索引](https://pypi.org/)和其他索引安装包。<a href="https://pip.pypa.io/en/stable/installation/" target="_blank">官方安装文档</a>）

   注意！下载及安装该脚本需要国外网络，好像也有人修改过官方安装脚本使其完全离线化安装

   下载`get-pip.py`脚本（这是一个 Python 脚本，它使用一些引导逻辑来安装 pip）：

   ```
   wget https://bootstrap.pypa.io/get-pip.py
   ```

   ps. 此时脚本是下载到临时目录（`./`）中了，如要删除需要cd到该目录删除

   使用python运行脚本以安装pip（以下使用mac示例）：

   ```
   //linux或mac：
   python get-pip.py
   
   //Windows：
   py get-pip.py
   ```

   

   

3. 使用pip安装`pypinyin`和`plexapi`（好像是最新版脚本已经去除依赖plexapi模块了，也就是说不需要这个了）模块

   ```
   pip install pypinyin
   ```

   

   

4. 运行脚本

   双击运行脚本

   根据提示输入信息即可

   也可将信息直接写入脚本文件，直接运行脚本就执行：

   脚本最后有这一段：

   ```
       URL = input('请输入你的 PLEX 服务器地址 ( 例如 http://127.0.0.1:32400 )：') or "http://127.0.0.1:32400"
       TOKEN = input('请输入你的 TOKEN：')
       TYPE = input('请输入操作的库类型，1为电影，2为电视剧：') or 1
   
       server = PLEX(URL, TOKEN, TYPE)
       print(server.listLibrary())
       sectionId = input("选择要操作的库的 ID 数字:")
   
   ```

   将其中的`input`改为你的信息（以下信息为随机字符以作示意）：

   ```
       URL = "http://192.168.1.1:32400"
       """*/('请输入你的 PLEX 服务器地址 ( 例如 http://192.168.1.1:32400 )：') or "http://192.168.1.1:32400"/*"""
       
       TOKEN = "0ByC9zs2H8888Epx21Gm"
       """*/请输入你的 TOKEN：(0ByC9zs2H8888Epx21Gm)/*"""
       
       TYPE = "2"
       """*/('请输入操作的库类型，我的库为：1为电影，2为电视剧：') or 1/*"""
   
       server = PLEX(URL, TOKEN, TYPE)
       print(server.listLibrary())
       sectionId = input("选择要操作的库的 ID 数字:")
       """*/我的库为：[<ShowSection:1:剧集>, <ShowSection:4:动画>, <ShowSection:3:综艺>]/*"""
   
   ```

   




## 群晖执行



1. 使用管理员账号临时获得-i权限（`sudo -i`）或直接使用root账号通过SSH登陆群晖

2. 按照上述步骤安装pip和pypinyin模块

   ```
   wget https://bootstrap.pypa.io/get-pip.py
   
   python get-pip.py
   
   pip install pypinyin
   ```

   ps. 此时脚本是下载到临时目录（`./`）中了，如要删除需要cd到该目录删除
   
   


3. 群晖添加脚本任务

   群晖DSM进入：控制面板 --> 任务计划 --> 新增 --> 计划的任务 --> 用户定义的脚本
   
   用户账号选择root，任务设置 -- 运行命令：
   
   ```
   python /volume1/public/script/plex_localization_zhcn_tvs.py
   python /volume1/public/script/plex_localization_zhcn_movie.py
   ```
   
   我此处试运行两份脚本，分别为电视剧和电影的脚本
   
   再选择合适自己的运行计划，运行频率就好了









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

