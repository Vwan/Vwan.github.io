---
title: 批量导出微信语音文件 转成mp3
date: 2017-11-18 21:05:13 +0800
layout: post
categories:

- python

tags:

- python

---

# 批量导出微信语音文件 转成mp3

## 需求

手机内存不够，近期的微信语音太多，而且需要反复听，所以需要导出到电脑并转成mp3格式

## 导出微信语音文件到电脑

android手机上，微信的语音文件存放位置在：

```
...\tencent\weixin\micromsg\<一个很长的文件名>\voice2
```

将voice2 folder复制到本地电脑，由于amr文件分散在各个子目录里，故写了一段python脚本，将所有amr文件提取出来，放到一个文件夹中，并按年份和月份进行分组

```
import os, shutil, datetime
"""
copy wechat .amr files and group by year and month
"""

def copy_files(src, target, filetype=None):
    id = 1
    for dirname, subdirlist, filelist in os.walk(src):
        for fname in filelist:
            if filetype:
                if filetype in fname:
                    mtime = os.stat(os.path.join(dirname, fname)).st_mtime
                    dtime = datetime.datetime.fromtimestamp(mtime)
                    month = str(dtime.month)
                    year = str(dtime.year)
                    if not os.path.exists(os.path.join(target, year, month)):
                        os.makedirs(os.path.join(target, year, month))
                    newtime = str(dtime).replace(":", "-")
                    newfilename = newtime + "_" + str(id) + "_" + fname
                    print("src----", os.path.join(dirname, fname))
                    print("target---", os.path.join(os.path.join(target, year, month), newfilename))
                    shutil.copy(os.path.join(dirname, fname), os.path.join(os.path.join(target, year, month), newfilename))
                    id += 1

src = r"c:\temp\voice2"
target = r"c:\temp\amrfiles_20171118_by_month"
if not os.path.exists(target):
    os.mkdir(target)
copy_files(src, target, '.amr')
```



## 将amr文件转成mp3

试了ffmpeg, 狸窝，都报错，说amr数据不对。

后来终于通过知乎搜索找到了silk2mp3, 顺利转成功。

https://kn007.net/topics/batch-convert-silk-v3-audio-files-to-mp3-in-windows/

不足的一点是不能直接选择一个folder添加所有文件。

## 将所有零散的mp3文件合并成一个mp3文件

安装winrar, 选中所有文件后，右键选择“压缩为”，

在弹出的“压缩文件名和参数”对话框中，

- 压缩方式：存储

- 压缩文件名：将.rar换成“.mp3"

  压缩后即可。