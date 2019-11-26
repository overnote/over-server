## Git本地多用户配置

场景：一台开发电脑上拥有2个git账户，对不同的仓库使用不同的账户提交。  

第一步：清空默认用户名
```
git config --list                           # 查看当前用户列表
git config --global --unset user.name
git config --global --unset user.email
```

第二步：添加多个用户
```
ssh-keygen -t rsa -f ~/.ssh/id_rsa_user1 -C "user1@gmai.com" 
ssh-keygen -t rsa -f ~/.ssh/id_rsa_user2 -C "user2@gmai.com" 
```

第三步：信任添加的用户
```
ssh-agent bash
ssh-add ~/.ssh/id_rsa_user1         # 返回 Identitiy added成功
ssh-add ~/.ssh/id_rsa_user2         # 返回 Identitiy added成功
```

第四步：将上述公钥分别加入到对应的git账户中  

第五步：创建或修改.ssh目录下的config文件，内容为：
```
#user1
Host github.com
Hostname github.com
IdentityFile ~/.ssh/id_rsa_user1
User user1
  
#user2
Host github.com
Hostname github.com
IdentityFile ~/.ssh/id_rsa_user2
User user1
```

第六步：使用git clone 项目后，需要设置git用户与邮箱
```
# 方式一：进入该项目根目录进行如下命令
git config --local user.name "user1"
git config --local user.email "user1@gmail.com"

# 方式二：进入该项目后找到 .git/config 文件，添加如下配置
[user]
	name = user1
	email = user1@gmail.com
```

测试连接
```
ssh -T user1@github.com
ssh -T user2@github.com
```