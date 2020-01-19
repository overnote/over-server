## 一 git 简介

版本管理系统的演变：
- 本地版本控制系统：能够进行版本切换，但是很难实现多人协同开发，已淘汰；
- 集中式版本控制系统：所有人向同一服务器提交更新，例如软件SVN；
- 分布式版本控制系统：无需中央服务器，每个人都拥有完整的版本库，由共享服务								器来同步、更新数据，代表软件Git

## 二 git 安装

#### 2.1 win与mac安装
```
Window安装：https://git-scm.com/直接下载安装
Mac安装：命令行输入命令 git  
```
安装完毕后：任意目录（建议开发根目录）右键 > Git Bash Here	

#### 2.2 centOS源码安装git

```
1 查看当前系统是否已经安装git
git --version              

2 如果没有安装git，那么此时需要预先安装git依赖包
yum -y update
yum -y install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker

3 下载git并解压源码
wget https://github.com/git/git/archive/v2.3.0.zip
unzip v2.3.0.zip                                # 如果提示unzip不存在，则 yum install -y unzip zip

3 编译安装到/usr/local/git目录下
cd git-2.3.0
make prefix=/usr/local/git all
make prefix=/usr/local/git install

注意：因为服务器时间不对编译的过程中报错如下图，使用ntpdate自动校正系统时间。报错“Writing perl.mak for Git make[2]: *** [perl.mak] Error 1”，请重启apache服务，service httpd restart。

4 定义git路径
git --version                                   # 查看版本的结果是1.8，不是2.3，因为默认使用了"/usr/bin"下的git
vim /etc/profile                                # 或者 /etc/bashrc
export PATH=/usr/local/git/bin:$PATH            # 加入该句
source /etc/profile
git --version                                   # 版本号为2.3，安装成功
```

## 三 git 使用

#### 3.0 git工作原理与流程

git工作原理：
- Git管理文件的三种状态：committed（提交）、modified（修改）、staged（暂存）；
- Git对应的三个工作区域：Git仓库、工作区、暂存区。

```
Git仓库：	Git用来保存项目的元数据和对象数据库的地方。
工作目录：对项目的某个版本独立提取出来的内容，放在磁盘上以使用或修改。
暂存区域：是一个文件，保存了下次将提交的文件列表信息，一般在Git仓库目录中。
```
Git本地仓库指的是开发者开发设备中的仓库。  

基本的Git工作流程如下：
- 1 在工作目录中修改文件。
- 2 暂存文件，将文件的快照放入暂存区域。
- 3 提交文件，找到暂存区域的文件，将快照永久性存储到Git仓库目录。

#### 3.1 基础使用

第一步：配置git开发者信息
```
命令：
git config --global user.name "开发者名字"
git config --global user.email "开发者的邮箱"

简介：
--global:配置当前用户所有仓库
--system:配置当前计算机上所有用户的所有仓库

注意：
配置用户只需要执行1次，可以重复使用。通过 git config --list  可以查看配置信息，
当然也可以在当前用户的根目录下查找到隐藏文件：.gitconfig。
```

第二步：初始化仓库
```
初始化命令：如果要自己从新开始新建一个项目，并让Git管理，需要先进行初始化
git init        # 在项目文件夹下cmd中输入该命令生成.git隐藏目录控制项目。

克隆命令：
git clone ‘https://....’	# 由于clone下来的项目一般直接已经被Git控制了，无需初始化，直接就可以进入开发。
```

第三步： add 添加文件到暂存区
```
git status          # 此时会列出当前目录下的文件，红色标识表示文件并未add到暂存区。
add 文件名		    # 添加文件到暂存区,添加后，查询状态会发现添加过的文件为绿色。

注意：
add . 或者 add * 或者 add -A 代表添加所有文件。
```

第四步：commit 提交文件到本地仓库
```
git commit -m '备注信息'    # 将暂存区被标记成绿色的文件，全部提交到本地仓库存储。提交的时候如果没有写提交说明

注意：
在提交时往往需要pull一次：git pull
```

第四步：推送到远程仓库
```
git push
```

其他：版本回退
```
git log		可以查看所有的提交历史，并能看到提交记录的sha值；

通过sha值可以回到某一次的提交：
git reset --hard sha值

当然也可以直接让某个文件回复到某一次提交：
git checkout sha值	文件名

注意：sha值可以只写前几位即可。

还可以按照提交次数回退
git reset --hard HEAD^  回退一次
git reset --hard HEAD~3 回退3次

git reflog 查看最近的几次git操作

git reset 参数的解释
--hard 		工作区会变、历史(HEAD)会变， 暂存区也变
--soft 		只会变历史(HEAD)
--mixed		默认：历史(HEAD)会变、暂存区也变，工作区不变

git reset 和git checkout区别：reset 重写了历史，checkout 则没有。
```



#### 3.2 使用问题

问题一：commit时，填错了-m内容，该如何撤销提交？ 
``` 
此时，可以重新add后再次commit，但是使用的命令为：`git commit -m "****" --amend`  
```

问题二：如何清空历史提交记录，成为一个干净的新仓库？  

即把旧项目提交到Git上，但是会有一些历史记录，这些历史记录中可能会有项目密码等敏感信息。如何删除这些历史记录，形成一个全新的仓库，并且保持代码不变呢？
```
git checkout --orphan latest_branch         # 1. Checkout
git add -A                                  # 2. Add all the files
git commit -am "commit message"             # 3. Commit the changes
git branch -D master                        # 4. Delete the branch
git branch -m master                        # 5.Rename the current branch to master
git push -f origin master                   # 6.Finally, force update your repository
```

## 四 git 分支

### 4.1 理解分支

分支用来解决项目的不同进度保存问题。在Git的使用过程中一次提交称为历史记录（版本），并且会生成一个唯一的字符串，即sha值（代表某一历史版本）。  

在Git中所有的提交（commit）实际上都是在分支（branch）的基础上进行的。  

在产生第1次提交时，Git会默认创建了一个名为master的分支，并且有指针（HEAD）指到了末端。指针（HEAD）用来标明当前处于哪个分支的哪个版本。

#### 4.2使用分支
```
git branch                     # 查看分支，查询结果中 *表示当前所在分支
git branch ‘分支名称’	 	    # 创建一个新的分支
git checkout ‘分支名称’	 	    # 切换分支
git checkout -b deeveloper 	    # 创建并切到developer分支
git merge ‘分支名称’ 		    # 合并分支
git branch -d ‘分支名称’ 	    # 删除分支
git branch -a				    # 查看远程主机分支个数
```

新的分支会在当前分支原有历史版本的结点上进行创建，我称其为子分支,新建的子分支会继承父分支的所有提交历史。

```
git merge ‘说明’		    # 合并分支时需要告诉Git合并到哪个分支
git merge master            # 即将master分支的修改内容加入到了hotfix下。
git push origin “本地分支名称:远程分支名称”将本地分支推送至远程仓库，
git push origin hotfix（通常的写法）相当于
git push origin hotfix:hotfix
git push origin hotfix:newfeature
```

本地仓库分支名称和远程仓库分支名称一样的情况下可以简写成一个，即git push “仓库地址” “分支名称”，如果远程仓库没有对应分支，将会自动创建:
```
删除远程分支git push origin --delete 分支名称
删除远程分支git purigin sh o:分支名称
```

