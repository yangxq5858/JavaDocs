# JavaDocs
2018年Java体系相关技术笔记



## 迅雷账号

迅雷1年的非官方购买渠道买的。

卡号：请加客服QQ757986669,有问题联系客服.微博图标登录17059457149密码mianfeivip.com2w不要删除别人文件，登录方法xinjipin.com/p



点击网盘上的微博图标登录17173493459密码mianfeivip.com5e



## 将本地文件上传到github

git命令工具Git Bash here 。 

首先在Git Bash中使用cd命令进入对应的本地项目路录，按照下面的命令操作： 
1、**git init** 表示在当前的项目目录中生成本地的git管理。
2、**git add .** 表示你要提交到github上的文件，如果你要将所有文件都添加上去的话，使用git add . “.”表示添加当前目录中的所有文件。

3、**git commit -m “first commit”**,表示你对这次提交的注释。

git commit -m “提交的描述信息” 
如果我们这里不用-m参数的话，git将调到一个文本编译器（通常是vim）来让你输入提交的描述信息

git commit -a -m “提交的描述信息” 
git commit 命令的-a 选项可只将所有被修改或者已删除的且已经被git管理的文档提交倒仓库中。如果只是修改或者删除了已被Git 管理的文档，是没必要使用git add 命令的。

git commit –-amend 对于已经修改提交过的注释，如果需要修改，可以借助 git commit –-amend 来进行。

4、**在github上建立一个仓库**，比如仓库名为micro-weather

5、**git remote add origin https://github.com/yangxq5858/micro-weather**

6、**git push -u origin master**



如果是第一次使用，还需要执行下面的命令**实现登陆**

（ git config --global user.email "you@example.com"
​     git config --global user.name "Your Name"）