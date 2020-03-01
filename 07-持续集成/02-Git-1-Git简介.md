## 一 git 简介

版本管理系统的演变：
- 本地版本控制系统：能够进行版本切换，但是很难实现多人协同开发，已淘汰；
- 集中式版本控制系统：所有人向同一服务器提交更新，例如软件SVN；
- 分布式版本控制系统：无需中央服务器，每个人都拥有完整的版本库，由共享服务								器来同步、更新数据，代表软件Git

## 二 git 安装

### 2.1 win与mac安装
```
Window安装：https://git-scm.com/直接下载安装
Mac安装：命令行输入命令 git  
```
安装完毕后：任意目录（建议开发根目录）右键 > Git Bash Here	

### 2.2 centOS源码安装git

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
