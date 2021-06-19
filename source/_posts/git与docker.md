---
title: git与docker
date: 2020-12-14 16:56:33
tags: [git,docker]
---

## 一、git

### 1、什么是git

​	git 是一种常的版本管理工具，与之类似的有SVN

git与SVN的区别

- git
  - git是分布式的，每一个安装git环境的机器都有线下的版本仓库
  - 开发者无需把代码提交到线上仓库（GetHub、getee、gitlab）
- SVN
  - SVN是一种集中的版本管理工具
  - 开发者必须把代码提交到SVN服务器，本地没有版本仓库



### 2、git原理

- 工作区：就是在电脑上看到的目录，比如目录下testgit里的文件（.git隐藏目录版本库除外）
- 暂存区：暂存区就是文件夹.git 中的一小部分（.git就是版本库）
- 版本库：工作区有一个隐藏目录.git，这个不属于工作区，就是版本库



### 3、git安装

​	直接从官网下载---->下一步到底安装完成（到桌面右键查看是否有git选项）



### 4、git的使用

```shell
# Git 全局设置（新创建仓库都会有提示）
git config --global 用户名
git config --global 用户邮件
# 创建Git仓库
mkdir 仓库名  # 本地创建仓库
cd 仓库名  # 进入仓库
git init  # 初始化
touch 文件名  # 创建文件
git add 文件名/./-A  # 将工作区文件提交到暂存区
git commit -m "注释"  # 将暂存区代码提交到版本库
git remote add origin 地址  # 将地址以origin的形式存放
git push -u origin 分支名  # 将代码放到分支后推送到网上

# 已经有的仓库
cd 已有仓库名  # 进入以前的仓库
git remote add origin 地址  # 将地址以origin的形式存放
git push -u origin 分支名  # 将代码放到分支后推送到网上
```



### 5、git常见命令

>1、简单提交

```shell
git add 文件名/./-A  # 将工作区文件提交到暂存区
git commit -m "注释"  # 将暂存区代码提交到版本库
git status  # 查看未提交到仓库的文件（工作区、暂存区）
```

>2、回滚

```shell
git log  # 查看所有提交到仓库的版本记录
git reflog  # 查看所有操作记录（状态的md5值和改变的值）
git reset --hard 创建版本的md5值的前4-6位  # 回到指定版本
git reset --hard HEAD^  #回到上一版本
```

>3、撤销修改

```shell
vim Readme  # 我们在Readme文件中写了一些错误的代码
git add .  # 然后又一不小心将文件从工作区提交到了 stage区
git reset HEAD Readme  # 将Readme中刚提交到 stage区 的代码撤回到工作区
git status  # 查看目前工作区状态
git checkout -- Readme  # 将Readme在工作区错误的代码丢弃
```

>4、强制使用master覆盖本地代码

```shell
git fetch --all
git reset --hard origin/master
git pull
```



### 6、git分支管理

分支管理的概念：

- master主分支，稳定代码，未生产环境做准备
- develop开发分支，为开发服务做准备

分支的常用操作：

```shell
# 一般上master分支不被开发，仅允许在develop分支开发

'''1. 从master转到develop分支开发'''
git checkout -b dev master  # 从master分支创建dev开发分支
git branch  # 可以看到现在已经切换到dev分支了
vim Readme  # 模拟在项目中修改代码
git add .  # 把工作区中所有变更全部提交到（暂存区）
git commit -m 'in dev'  # 把暂存区代码提交到本地git仓库（本地git仓库）
git push origin dev  # 把代码先推倒dev分支，让测试人员测试

'''2. 当测试人员测试完成后将dev分支代码合并到master形成文档版本'''
git checkout master  # 开发完成后，需要合并回master分支，先切换到master分支
cat Readme  # 可以看到master分支的内容没有修改
git merge --no-ff dev  # 将刚刚dev中修改的代码合并到master
git push origin master  # 将本地仓库的代码推送到线上仓库（线上git仓库）
```



## 二、docker

### 1、docker是什么

docker是全球范围内最受欢迎的容器技术

- 优点：

​		把自己或三方的软件或服务放到容器里，能够很方便的帮你运行，不需要安装，不需要配置，并且可以以毫秒级的运算速度直接启动

- 缺点：

  对win系统不友好



### 2、docker安装（Linux）

```shell
# 1.卸载旧版本
sudo apt-get remove docker docker-engine docker.io containerd runc

# 2.更新ubuntu的apt源索引
# 修改apt国内源为中科大源
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
sudo sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/' /etc/apt/sources.list
sudo apt update

#3.安装包允许apt通过HTTPS使用仓库
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common

#4.添加Docker官方GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

#5.设置Docker稳定版仓库
#5.1 设置使用官方，很慢
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
#5.2 设置使用阿里云
add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
    
#6.添加仓库后，更新apt源索引
sudo apt-get update

#7.安装最新版Docker CE（社区版）
sudo apt-get install docker-ce

#8.检查Docker CE是否安装正确
sudo docker run hello-world
```

- docker 默认是国外源可以设置成国内镜像源

```shell
root@linux-node1 django-docker]# vim /etc/docker/deamon.json    # 设置docker镜像源
{
    "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
[root@linux-node2 ~]# systemctl restart docker                  # 重启docker生效
```

- docker启动设置

```shell
# 启动Docker服务并设置开机启动
systemctl start docker
systemctl enable docker
```

- docker 简单使用

```shell
# 1、创建一个nginx容器
 docker run -it nginx
 
 # 2、查看docker运行的容器(可以获取到这个容器的id)
 docker ps
 
 # 3、访问这个容器
 # 进入这个nginx容器（进入的文件系统和宿主机是完全隔离的，有自己独立的文件系统）
 docker exec -it 73877e65c07d bash
 
 # 4、查看当前容器的 IP
 docker inspect 73877e65c07d   # 73877e65c07d是通过docekr ps查看到的容器ID
 curl 172.17.0.2               # 测试这个nginx容器是否可以访问
```



### 3、基本使用

- #### 常用命令

  - 镜像常用命令

    ```shell
    [root@linux-node4 diff]# docker help                           # 查看docker帮助
    [root@linux-node4 diff]# docker image --help                   # 查看 docker中 镜像相关帮助
    [root@linux-node4 diff]# docker image ls                       # 查看当前所有镜像
    [root@linux-node4 diff]# docker image inspect nginx            # 查看指定镜像（nginx镜像）详细信息
    [root@linux-node4 diff]# docker pull nginx:1.14                # 下载指定版本镜像 nginx
    [root@linux-node4 diff]# docker image rm nginx:1.14            # 删除nginx 1.14版本
    ```

  - docker创建容器常用命令

    - docker run 的基本使用

    ```shell
    root@dev:~# docker run -itd nginx
    root@dev:~# docker ps
    root@dev:~# docker rm -f e182a69f841d
    ```

    - docker run 的常用参数

    ```shell
    -d   # 后台运行容器，并返回容器ID；
    -i   # 以交互模式运行容器，通常与 -t 同时使用；
    -t   # 为容器重新分配一个伪输入终端，通常与 -i 同时使用；
    -P   # 随机端口映射，容器内部端口随机映射到主机的高端口
    -p   # 指定端口映射，格式为：主机(宿主)端口:容器端口
    --name="nginx-lb"   # 为容器指定一个名称；
    --dns 8.8.8.8       # 指定容器使用的DNS服务器，默认和宿主一致；
    ```

    - docker 使用

    ```shell
    [root@linux-node4 diff]# docker container run -d --name web3 -e test=123456 -p 8800:80 -h webhostname --restart always nginx
    -d                   # 后台启动nginx容器
    --name web3          # 自定义容器名字(默认会是一段随机字符串)
    -e test=123456       # 启动容器添加变量 test=123456 (echo $test)
    -p 8800:80           # 宿主机的8800端口映射到docker容器的80端口中
    -h webhostname       # docker容器主机名 (a300f394af88)
    --restart always     # 宿主机重启自动拉起这个docker容器
    nginx                # 使用这个nginx镜像启动容器
    注：http://192.168.56.12:8800/     访问这个docker  nginx
    [root@linux-node4 diff]# docker logs web                 # 查看上面启动的web容器的日志
    [root@linux-node4 diff]# docker exec -it web bash        # 进入容器web
    ```



## 三、docker 安装ES

```shell
docker pull docker.elastic.co/elasticsearch/elasticsearch:7.3.0
```



## 四、windows  docker配置国内源

```http
https://blog.csdn.net/qq_37475168/article/details/104711738
```

