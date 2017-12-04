---
title: Docker 配置机器学习环境
date: 2017-12-04 21:05:13 +0800
layout: post
categories:
- Python
tags:
- Python
---

# Docker 配置机器学习环境

## 安装Docker

### Installer

[Install Docker Toolbox on Windows | Docker Documentation](https://docs.docker.com/toolbox/toolbox_install_windows/)

如果是legacy windows 比如windows Home, 由于docker需要enable Hyper-V模块，但微软官方说：

> The Hyper-V role **cannot** be installed on Windows 10 Home.

所以上面的installer不能用在legacy windows上，需下载 [Install Docker Toolbox on Windows | Docker Documentation](https://docs.docker.com/toolbox/toolbox_install_windows/)

## 安装

傻瓜式。

登录docker vm:

```
docker-machine ssh default
```



## 启动docker service(server)

Windows, run "Docker for Windows" from start menu

Mac, run docker as you run other apps

## 启动docker client

打开一个命令行窗口，输入command "docker"，可以看到很多子命令

```
docker
```

### docker commands

显示当前运行的容器

```
docker ps
```

显示全部容器

```
docker ps -a
```

重新启动已退出的容器

```
docker start -i <container id的前几位数>
-i: 运行命令行和程序交互
```

删除容器(be careful)

```
docker rm <container id的前几位数>
```

## Docker Volume Mounting

suppose you want to expose your local folder to the docker container, you can do a folder mapping as follows:

1. Go to Docker Settings dialog, add your local folder to shared driver. (Non-Legacy Windows)

2. For Legacy Windows:

   - go to VirtualBox Manager -> Share Drive, add c:\Users

   - run command

     ```
     docker run -it -p 8888:8888 -v //c/Users/vivia/OneDrive/Projects/DeepLearning/test:/home/tensorflow tensorflow/tensorflow
     ```

     Reference as below:

   >  On Windows, you can not directly map Windows directory to your container. Because your containers are reside inside a VirtualBox VM. So your `docker -v` command actually maps the directory between the VM and the container.
   >
   > So you have to do it in two steps:
   >
   > - Map a Windows directory to the VM through VirtualBox manager
   > - Map a directory in your container to the directory in your VM
   >
   > You better use the Kitematic UI to help you. It is much eaiser.

   [boot2docker - Shared folder in Docker. With Windows. Not only "C/user/" path - Stack Overflow](https://stackoverflow.com/questions/33966225/shared-folder-in-docker-with-windows-not-only-c-user-path)

   > > I know that Docker only shares the /c/User/folder, is that right?
   >
   > It does, and it is able to do so because the VirtualBox VM used for providing a Linux host for docker is sharing C:\Users.
   >
   > For docker to see another folder, you would need to:
   >
   > - use `VBoxmanage sharedfolder add "VM name" --name "sharename" --hostpath "D:\Works"`
   >
   > - then mount `/D/Works` within a VM session, as mentioned in "[share windows folder (other than c/Users/) with docker container (using docker windows client)](https://stackoverflow.com/a/33935328/6309)", and [mentioned in boot2docker](https://github.com/boot2docker/boot2docker/blob/master/README.md#virtualbox-guest-additions):
   >
   >   ```
   >   mount -t vboxsf -o uid=1000,gid=50 sharename /some/mount/location
   >
   >   ```
   >
   > ------

## Docker Hub

[Docker Hub](https://hub.docker.com/) 上查找你需要的别人开发好的repository

### Docker 

## 配置机器学习环境

### Tensorflow

在docker hub上搜索tensorflow, 找到后，复制其运行命令到本地，运行之

```
 docker-machine ip default
docker run -it -p 8888:8888 -v x:\\vboxsvr\DeepLearning:/tensorflow tensorflow/tensorflow
```

* -t 参数：可在命令行与docker容器内的程序进行交互
* -v: 将本地的目录映射到容器内
* -p: 端口。第一个8888是docker容器运行的端口；第二个8888是容器内app运行的端口

运行成功后，你便以该tensorflow Image 为模板构建了你自己的容器

### Jupyter Notebook

打开浏览器，输入“http://localhost:8888". on Windows however I failed to access it, had to replace Localhost with the default_ip (run "docker-machine ip default" to get it): http://192.168.99.100:8888

* shift + enter to run codes in notebook

* ! ls /tensorflow: 输入感叹号+终端命令，可以直接运行终端命令

  ```
  !python test.py
  !pip install jieba
  ```

  ​

### Jieba and collections

pip install jieba

```
import jieba
from collections import Counter
words = open('/home/tensorflow/slqm.txt').read().strip()
data = Counter()
for s in jieba.cut(words):
    data[s.encode('utf-8')] += 1
print(data)
```

