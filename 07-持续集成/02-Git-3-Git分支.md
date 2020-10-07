## 一 git 分支

### 1.1 理解分支

分支用来解决项目的不同进度保存问题。在Git的使用过程中一次提交称为历史记录（版本），并且会生成一个唯一的字符串，即sha值（代表某一历史版本）。  

在Git中所有的提交（commit）实际上都是在分支（branch）的基础上进行的。  

在产生第1次提交时，Git会默认创建了一个名为master的分支，并且有指针（HEAD）指到了末端。指针（HEAD）用来标明当前处于哪个分支的哪个版本。

### 1.2 使用分支
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

### 1.3 分支使用步骤示例

```txt
# 查看当前分支
git branch

# 创建一个新分支 login
git checkout -b login

# 提交完成的代码到 login分支
git add .
git commit -m "完成 login"

# 提交分支到远程
git branch      # 查看本地分支，多出一个 login，而远程没有
git push -u origin login

# 合并本地分支到master
git checkout master
git merge login

# 提交master到远程
git push
```
