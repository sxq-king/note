# git是什么
## 集中式版本控制工具
> CVS、SVN(Subversion)、VSS....
> 集中化的版本控制系统，都有一个单一的集中管理的服务器，保存所有文件的修订版本，协同工作的人们都通过客户端连到这台服务器，取出最新的文件或者提交更新

优点
> 每个人都可以在一定程度上看到项目其他人正在做些什么，管理员也可以轻松掌控每个开发者的权限，并且管理一个集中化的版本控制系统要更加容易

缺点
> 中央服务器的单点故障。如果服务器宕机一小时，那么在这一小时内，谁都无法提交更新，就无法协同工作

## 分布式版本控制工具
> ==Git==、Mereurial、Bazaar、Darcs...
> 像Git这种分布式版本控制工具，客户端提取的不是最新版本的文件快照，而是把代码仓库完整的镜像下来(本地库)。这样任何一处协同工作用的文件发生故障，事后都可以用其他客户端的本地库进行恢复。因为每个哭护短的每一次文件提取操作，实际上都是一次对整个文件仓库的完整备份。

分布式的版本控制系统出现之后，解决了集中式版本控制系统的缺陷

1. 服务器断网的情况下也可以进行开发(因为版本控制式在本地进行的)
2. 每个客户端八寸的也都是整个完整的项目(包含历史记录，更加安全)


## Git历史

L大神花了两周开发出Git，花了两周测试，共一个月...。

## Git工作机制
Git有三块
1. 工作区 写代码，代码的存放目录(可以删除)
2. 暂存区 临时存储(可以删除)
3. 本地库 历史版本,无法删除 (只能删库跑路...)

## Git和代码托管中心
> 代码托管中心是基于网络服务器的远程代码仓库，一般我们简单称为远程库

### 局域网
    GitLab

### 互联网
    GitHub(外网)
    Gitee(国内网站)

## Git常用命令
命令名称 | 作用
---|---
git config --global user.name 用户名 | 设置用户签名
git config --global user.email 邮箱 | 设置用户签名
git init | 初始化本地库
git status | 查看本地库状态
git add 文件名 | 添加到暂存区
git commit -m "日志信息" 文件名 | 提交到本地库
git reflog | 查看版本记录
git log | 查看详细日志
git reset --hard 版本号 | 版本穿梭

## 分支的操作

命令名称 | 作用
---|---
git branch 分支名 | 创建分支
git branch -v | 查看分支
git checkout 分支名 | 切换分支
git merge 分支名 | 把指定的分支合并到当前分支上


## 团队协作
注册github

点击右上角加号New repository

选择public就行

其他不用管直接点击创建


## 远程库操作
命令 | 作用
---|---
git remote -v | 查看当前所有远程地址别名
git remote add | 起别名
git push 别名 分支 | 推送本地分支上的内容到远程仓库
git clone 远程地址 | 将远程仓库的内容克隆到本地
git pull 远程库地址别名 远程分支名 | 将远程仓库对于分支最新内容拉下来后于当前本地分支直接合并


邀请团队协作


## Git连接GitHub

> 因为Git是分布式版本控制系统，所以需要填写用户名和邮箱作为一个标识

Github支持两种同步方式`“https”`和`“ssh”`。

如果使用`https`很简单基本不需要配置就可以使用，但是每次提交代码和下载代码时都需要输入用户名和密码。

如果使用`ssh`方式就需要客户端先生成一个密钥对，即一个公钥一个私钥。然后还需要把公钥放到githib的服务器上。

- 如果配置过git忘记配置信息

```.java
$ git config --list //查看git全局配置
diff.astextplain.textconv=astextplain
filter.lfs.clean=git-lfs clean -- %f
filter.lfs.smudge=git-lfs smudge -- %f
filter.lfs.process=git-lfs filter-process
filter.lfs.required=true
http.sslbackend=openssl
http.sslcainfo=D:/Program Files/Git/mingw64/ssl/certs/ca-bundle.crt
core.autocrlf=true
core.fscache=true
core.symlinks=true
pull.rebase=false
credential.helper=manager-core
credential.https://dev.azure.com.usehttppath=true
init.defaultbranch=master
user.name=sxq	//用户名
user.email=996957240@qq.com  //邮箱

```

- 没有配置过git信息执行以下命令

```.java
git config --global user.name "XXXX"  用户名标识  ---- 实际也可以填写您的github仓库的名称

git config --global user.email "xxxx@xxx.com"  邮箱标识  -------可以填写github仓库的邮箱
```

#### 开始连接关键操作了

1. 配置SSH KEY.

   > 在用户主目录下，看看有没有.ssh目录，如果有，再看看这个目录下有没有`id_rsa`和`id_rsa.pub`这两个文件

```.JAVA
- 用户主目录 C:\Users\当前用户\.ssh
```

![](/images/用户主目录.png)

> 如果有的话，直接跳过此如下命令，如果没有的话，打开命令行，输入如下命令：

```.java
ssh-keygen -t rsa  //--创建秘钥
```

一直回车即可。

- id_rsa 是密匙

- id_rsa.pub是公匙

使用记事本打开复制公匙（ctrl+a，ctrl+c全选）

2. 你的远程仓库配置（github为例）

   > 登录github，github右上角你账号的头像选择settings ==> SSH and GPG keys ==> New SSH key
   >
   > Title 可以任意填写 ==> Key就填你复制的id_rsa.pub ==> 最后 Add New key。配置完成

3.  查看远程仓库

```.java
git remote      //--git查看远程仓库信息
//出现一下信息说明没有远程库
$ git remote
fatal: not a git repository (or any of the parent directories): .git
```

远程仓库创建两种方式

- git本地创建然后提交同时创建
- 通过github直接new repository 创建完成会出现以下信息

![](/images/github创建仓库信息.png)

第一种是没有本地库初始化本地库直接提交并合并到远程仓库。

```java
sxq@DESKTOP-ILEV43G MINGW64 ~/Desktop
$ mkdir test //我测试直接创建一个test空文件夹

sxq@DESKTOP-ILEV43G MINGW64 ~/Desktop
$ cd test

sxq@DESKTOP-ILEV43G MINGW64 ~/Desktop/test
$ echo "# test" >> README.md //写一些你仓库的介绍信息

sxq@DESKTOP-ILEV43G MINGW64 ~/Desktop/test
$ ll
total 1
-rw-r--r-- 1 sxq 197609 7 Nov 18 11:33 README.md

sxq@DESKTOP-ILEV43G MINGW64 ~/Desktop/test
$ git init
git commit -m "first commit"
git branch -M master
git remote add origin https://github.com/sxq-king/test.git
git push -u origin masterInitialized empty Git repository in C:/Users/sxq/Desktop/test/.git/
//以上五个个命令直接复制github提供的
sxq@DESKTOP-ILEV43G MINGW64 ~/Desktop/test (master)
$ git add README.md
warning: LF will be replaced by CRLF in README.md.
The file will have its original line endings in your working directory

sxq@DESKTOP-ILEV43G MINGW64 ~/Desktop/test (master)
$ git commit -m "first commit"
[master (root-commit) f63d112] first commit
 1 file changed, 1 insertion(+)
 create mode 100644 README.md

sxq@DESKTOP-ILEV43G MINGW64 ~/Desktop/test (master)
$ git branch -M master

sxq@DESKTOP-ILEV43G MINGW64 ~/Desktop/test (master)
$ git remote add origin https://github.com/sxq-king/test.git

sxq@DESKTOP-ILEV43G MINGW64 ~/Desktop/test (master)
$ git push -u origin master
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Writing objects: 100% (3/3), 210 bytes | 210.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
To https://github.com/sxq-king/test.git
 * [new branch]      master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.

sxq@DESKTOP-ILEV43G MINGW64 ~/Desktop/test (master)

```

第二种是本地已经有仓库连接到远程。

```.java
git remote add origin https://github.com/sxq-king/test.git //添加远程库
git branch -M master	//创建master分支
git push -u origin master //推送到master分支
```

over完成

## 工作流常用

1.git clone // 到本地
2.git checkout -b xxx 切换至新分支xxx
（相当于复制了remote的仓库到本地的xxx分支上
3.修改或者添加本地代码（部署在硬盘的源文件上）
4.git diff 查看自己对代码做出的改变
5.git add 上传更新后的代码至暂存区
6.git commit 可以将暂存区里更新后的代码更新到本地git
7.git push origin xxx 将本地的xxxgit分支上传至github上的git

------
（如果在写自己的代码过程中发现远端GitHub上代码出现改变）
1.git checkout main 切换回main分支
2.git pull origin master(main) 将远端修改过的代码再更新到本地
3.git checkout xxx 回到xxx分支
4.git rebase main 我在xxx分支上，先把main移过来，然后根据我的commit来修改成新的内容
（中途可能会出现，rebase conflict -----》手动选择保留哪段代码）
5.git push -f origin xxx 把rebase后并且更新过的代码再push到远端github上
（-f ---》强行）

6.原项目主人采用pull request 中的 squash and merge 合并所有不同的commit

------

远端完成更新后
1.git branch -d xxx 删除本地的git分支
2.git pull origin master 再把远端的最新代码拉至本地