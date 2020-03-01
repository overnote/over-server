## 一 git工作原理与流程

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

## 二 Git基础使用

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

## 三 Git使用问题

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