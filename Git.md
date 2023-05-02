# Git部署

![image-20230501233649571](assets/Git/image-20230501233649571.png)

```shell
环境：
    git-server    192.168.246.214  充当中央服务器
    client        192.168.246.213

所有机器关闭防火墙和selinux

安装：所有机器都安装
   [root@git-server ~]# yum install -y git
   [root@git-server ~]# git --version 
   git version 1.8.3.1
   
准备：
    因为Git是分布式版本控制系统，所以，每个机器都必须自报家门：你的名字和Email地址。
    注意git config命令的--global参数，用了这个参数，表示你这台机器上所有的Git仓库都会使用这个配置。

所有的机器都添加，只要邮箱和用户不一样就可以。   
    # git config --global user.email "tian1364548672@gmail.com"     ----设置邮箱
    # git config --global user.name "jack"                   ----加添用户
    # cat /root/.gitconfig
    # git config --global color.ui true		#语法高亮
    # git config --list			#查看全局配置
    user.email=tian1364548672@gmail.com
	user.name=jack
	color.ui=true
```

# git使用

```shell
1、创建一个新目录：在中心服务器上创建
[root@git-server ~]# mkdir /git-test
[root@git-server ~]# useradd git   #创建一个git用户用来运行git
[root@git-server ~]# passwd git  #给用户设置密码
[root@git-server ~]# cd /git-test/
```

```shell
2、通过git init命令把这个目录变成Git可以管理的仓库：
 第1种情况：可以改代码，还能上传到别人的机器，别人也能从你这里下载但是别人不能上传代码到你的机器上。
 第2种情况：只是为了上传代码用，别人从这台机器上下载代码也可以上传代码到这台机器上，经常用于核心代码库。
```

```shell
3、创建裸库：适用于作为远程中心仓库使用
创建裸库才可以从别处push（传）代码过来，使用--bare参数------裸
[root@git-server git-test]# git init --bare testgit
Initialized empty Git repository in /git-test/testgit/
[root@git-server ~]# chown git.git /git-test -R  #修改权限
```

```shell
4、仓库创建完成后查看库目录：
[root@git-server git-test]# cd testgit/
[root@git-server testgit]# ls
branches  config  description  HEAD  hooks  info  objects  refs
```

# 客户端

```shell
1.配置免密登录
[root@client ~]# ssh-keygen    #生成秘钥
[root@client ~]# ssh-copy-id -i git@192.168.246.214   #将秘钥传输到git服务器中的git用户
```

```shell
2.克隆git仓库
[root@client ~]# yum install -y git
[root@client ~]# git clone git@192.168.246.214:/git-test/testgit/
Cloning into 'testgit'...
warning: You appear to have cloned an empty repository.
[root@client ~]# ls  #查看仓库已经克隆下来了
anaconda-ks.cfg    testgit
```

```shell
3、创建文件模拟代码提交到仓库
在testgit目录下创建一个测试文件test.txt
[root@client ~]# cd testgit/
[root@client testgit]# vim test.txt   #随便写点东西


把文件添加到暂存区：使用 "git add" 建立跟踪
[root@client testgit]# git add test.txt
注: 这里可以使用 git add * 或者 git add -A


提交文件到仓库分支：1
[root@client testgit]# git commit -m "test1"
[master (root-commit) 2b51ff9] test1
 1 file changed, 2 insertions(+)
 create mode 100644 test.txt
 -m:描述
 
 
查看git状态：
[root@client testgit]# git status 
# On branch master   #分支位于master


修改文件后再此查看状态：
[root@client testgit]# echo '1122334' >> test.txt
[root@client testgit]# git status
# 位于分支 master
# 尚未暂存以备提交的变更：
#   （使用 "git add <file>..." 更新要提交的内容）
#   （使用 "git checkout -- <file>..." 丢弃工作区的改动）
#
#	修改：      readme.txt
#
修改尚未加入提交（使用 "git add" 和/或 "git commit "


先add
[root@client testgit]# git add -A


再次提交commit：
[root@client testgit]# git commit  -m "add2" test.txt 
[master 73bf688] add2
 1 file changed, 1 insertion(+)
 [root@client testgit]# git status 
# On branch master
nothing to commit, working directory clean
```

# 版本回退

```shell
已经提交了不合适的修改到版本库时，想要撤销本次提交，使用版本回退，不过前提是没有推送到远程库。

查看现在的版本：
[root@client testgit]# git log
显示的哪个版本在第一个就是当前使用的版本。

版本回退(切换)
在Git中，当前版本是HEAD,上一个版本就HEAD^，上上一个版本就是HEAD^^，当然往上100个版本写100个比较容易数不过来，所以写成HEAD~100（一般使用id号来恢复）

回到上一个版本
[root@client testgit]# git reset --hard HEAD^ 
HEAD is now at 0126755 test1

回到指定的版本(根据版本号): 
[root@client testgit]# git reset --hard dd66ff
HEAD is now at dd66ff9 add2
==========================================================
注：消失的ID号：
回到早期的版本后再查看git log会发现最近的版本消失，可以使用reflog查看消失的版本ID，用于回退到消失的版本
[root@vm20 gittest]# git reflog
2a85982 HEAD@{0}: reset: moving to 2a859821a2385e136fe83f3a206b287eb0eb8c18
f5bc8c1 HEAD@{1}: commit: test-version2
2a85982 HEAD@{2}: commit (initial): test-version1

[root@git-client testgit]# git reset --hard f5bc8c1
```

# 删除文件

```shell
从工作区删除test.txt，并且从版本库一起删除
```

```shell
工作区删除：
[root@client testgit]# touch test.txt
[root@client testgit]# git status
# On branch master
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#       as.txt
nothing added to commit but untracked files present (use "git add" to track)
[root@client testgit]# rm -rf test.txt  未添加到暂存区，可直接删除
[root@client testgit]# git status
# On branch master
nothing to commit, working directory clean

```

```shell
已从工作区提交到暂存区：
第一种方法
[root@client testgit]# touch test.txt
[root@client testgit]# git status
# On branch master
#
# Initial commit
#
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#       test.txt
nothing added to commit but untracked files present (use "git add" to track)

[root@client testgit]# git add test.txt
[root@client testgit]# git status
# On branch master
#
# Initial commit
#
# Changes to be committed:
#   (use "git rm --cached <file>..." to unstage)
#
#       new file:   test.txt
#

[root@client testgit]#  git rm --cache test.txt #从暂存区移除
rm 'test.txt'
[root@client testgit]# ls
test.txt
[root@client testgit]# git status
# On branch master
#
# Initial commit
#
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#       test.txt
nothing added to commit but untracked files present (use "git add" to track)
[root@client testgit]# rm -rf test.txt 
[root@client testgit]# git status
# On branch master
#
# Initial commit
#
nothing to commit (create/copy files and use "git add" to track)

第二种方法：
[root@client testgit]# touch  b.txt
[root@client testgit]# git add b.txt 
[root@client testgit]# git status
# On branch master
#
# Initial commit
#
# Changes to be committed:
#   (use "git rm --cached <file>..." to unstage)
#
#       new file:   b.txt
#
[root@client testgit]# git rm -f b.txt  ##这种删除连同工作区一块儿删除
rm 'b.txt'
[root@client testgit]# ls
[root@client testgit]# git status    
# On branch master
#
# Initial commit
#
nothing to commit (create/copy files and use "git add" to track)
```

```shell
直接在暂存区rm掉文件，如何解决
[root@client testgit]# touch c.txt
[root@client testgit]# git add c.txt 
[root@client testgit]# ls
c.txt
[root@client testgit]# git status
# On branch master
#
# Initial commit
#
# Changes to be committed:
#   (use "git rm --cached <file>..." to unstage)
#
#       new file:   c.txt
#
[root@client testgit]# rm -rf c.txt 
[root@client testgit]# git status
# On branch master
#
# Initial commit
#
# Changes to be committed:
#   (use "git rm --cached <file>..." to unstage)
#
#       new file:   c.txt
#
# Changes not staged for commit:
#   (use "git add/rm <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#       deleted:    c.txt
#
[root@client testgit]# git rm --cache c.txt
rm 'c.txt'
[root@client testgit]# ls
[root@client testgit]# git status
# On branch master
#
# Initial commit
#
nothing to commit (create/copy files and use "git add" to track)
[root@client testgit]# 
```

# 修改文件

```shell
暂存区修改名称
[root@client testgit]# touch  a.txt
[root@client testgit]# git status
# On branch master
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#       a.txt
nothing added to commit but untracked files present (use "git add" to track)
[root@client testgit]# git add a.txt 
[root@client testgit]# git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#       new file:   a.txt
#
[root@client testgit]# git mv a.txt  d.txt
[root@client testgit]# git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#       new file:   d.txt
#
[root@client testgit]# ls
d.txt  test.txt
[root@client testgit]# git rm --cache d.txt
[root@client testgit]# rm -rf d.txt
```

# 将代码上传到仓库的master分支

```shell
[root@client testgit]# vi a.txt   #创建一个新文件
hello world
[root@client testgit]# git add a.txt 
[root@client testgit]# git commit -m "add"
[root@client testgit]# git push origin master   #上传到中心仓库master分支
Counting objects: 11, done.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (11/11), 828 bytes | 0 bytes/s, done.
Total 11 (delta 0), reused 0 (delta 0)
To git@192.168.246.214:/git-test/testgit/
 * [new branch]      master -> master
```

```shell
在客户端将仓库删除掉然后在克隆下来查看仓库中是否有文件
[root@client testgit]# cd
[root@client ~]# rm -rf testgit/
[root@client ~]# git clone git@192.168.246.214:/git-test/testgit/
Cloning into 'testgit'...
remote: Counting objects: 11, done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 11 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (11/11), done.
[root@client ~]# cd testgit/
[root@client testgit]# ls
a.txt
[root@client testgit]# cat a.txt 
hello world
```

# 创建分支并合并分支

```shell
每次提交，Git都把它们串成一条时间线，这条时间线就是一个分支。截止到目前，只有一条时间线，在Git里，这个分支叫主分支，即`master`分支。`HEAD`严格来说不是指向提交，而是指向`master`，`master`才是指向提交的，所以，`HEAD`指向的就是当前分支。
```

```shell
客户端操作
[root@client ~]# git clone git@192.168.246.214:/git-test/testgit/
[root@client testgit]# git status 
# On branch master   #当前所在为master分支
#
# Initial commit
#
nothing to commit (create/copy files and use "git add" to track)
注意：刚创建的git仓库默认的master分支要在第一次commit之后才会真正建立。然后先git add .添加所有项目文件到本地仓库缓存，再git commit -m "init commit"提交到本地仓库，之后就可以随心所欲地创建或切换分支了。
创建分支:
[root@client testgit]# git branch dev   #创建分支。
[root@client testgit]# git branch    #查看分支。*在哪里就表示当前是哪个分支
  dev
* master
切换分支:
[root@client testgit]# git checkout dev
Switched to branch 'dev'
[root@client testgit]# git branch 
* dev
  master
在dev分支创建一个文件；
[root@client testgit]# vi test.txt
[root@client testgit]# git add test.txt 
[root@client testgit]# git commit -m "add dev"
[dev f855bdf] add dev
 1 file changed, 1 insertion(+)
 create mode 100644 test.txt
现在，dev分支的工作完成，我们就可以切换回master分支：
 [root@client testgit]# git checkout master
Switched to branch 'master'
```

```shell
切换回`master`分支后，再查看一个`test.txt`文件，刚才添加的内容不见了！因为那个提交是在`dev`分支上，而`master`分支此刻的提交点并没有变：
```

```shell
[root@client testgit]# ls
a.txt
```

```shell
现在，我们把`dev`分支的工作成果合并到`master`分支上：
```

```shell
[root@client testgit]# git merge dev
Updating 40833e0..f855bdf
Fast-forward
 test.txt | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 test.txt
[root@client testgit]# ls
a.txt  test.txt
现在已经将dev分支的内容合并到master上。确认没有问题上传到远程仓库:
[root@client testgit]# git push origin master
```

```shell
`git merge`命令用于合并指定分支到当前分支。合并后，再查看`test.txt`的内容，就可以看到，和`dev`分支的最新提交是完全一样的。
```

```shell
合并完成后，就可以放心地删除`dev`分支了：
```

```shell
[root@client testgit]# git branch -d dev
Deleted branch dev (was f855bdf).
```

```shell
删除后，查看`branch`，就只剩下`master`分支了：
[root@client testgit]# git branch 
* master
```

# 部署gitlab服务

准备环境:  关闭防火墙和selinux

```shell
192.168.246.214  #gitlab服务器
```

1.配置yum源

```shell
[root@git-server ~]# cd /etc/yum.repos.d/
[root@git-server yum.repos.d]# vi gitlab-ce.repo
[gitlab-ce]
name=Gitlab CE Repository
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el$releasever
gpgcheck=0
enabled=1
安装相关依赖
https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/
[root@git-server yum.repos.d]# yum install -y postfix curl policycoreutils-python openssh-server
[root@git-server yum.repos.d]# systemctl enable sshd
[root@git-server yum.repos.d]# systemctl start sshd
安装postfix
[root@git-server yum.repos.d]# yum install postfix  -y   #安装邮箱
[root@git-server yum.repos.d]# systemctl enable postfix
[root@git-server yum.repos.d]# systemctl start postfix
[root@git-server yum.repos.d]# yum install -y gitlab-ce  #将会安装gitlab最新版本
```

配置gitlab登录链接

```shell
[root@git-server ~]# vim /etc/gitlab/gitlab.rb
1.# 添加对外的域名（gitlab.papamk.com请添加A记录指向本服务器的公网IP）：将原来的修改为
external_url 'http://192.168.246.214'
2.设置地区
gitlab_rails['time_zone'] = 'Asia/Shanghai'
```

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672067618611-cac6861d-6352-4267-8eab-fc7c65ab547d.png#averageHue=%2323201f&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=96&id=u10828067&name=image.png&originHeight=172&originWidth=1228&originalType=binary&ratio=1&rotation=0&showTitle=false&size=71191&status=done&style=none&taskId=uf7947867-18c8-439e-bf52-343d09041ca&title=&width=682.2222402949396#averageHue=%2323201f&crop=0&crop=0&crop=1&crop=1&id=bfoyR&originHeight=172&originWidth=1228&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

将数据路径的注释去掉，可以更改

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672067627212-dc28199c-5ae2-4675-be7e-c28f55e58f6c.png#averageHue=%230c0b0a&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=222&id=ue94b65a1&name=image.png&originHeight=399&originWidth=910&originalType=binary&ratio=1&rotation=0&showTitle=false&size=82252&status=done&style=none&taskId=u9dd4b4ca-9fd3-42f2-b111-c51e414b222&title=&width=505.5555689482045#averageHue=%230c0b0a&crop=0&crop=0&crop=1&crop=1&id=eg35E&originHeight=399&originWidth=910&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

开启ssh服务:

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672067643207-29d99b43-32dc-4d0a-a696-7d9361eea219.png#averageHue=%2310100e&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=145&id=u9c295a77&name=image.png&originHeight=261&originWidth=1044&originalType=binary&ratio=1&rotation=0&showTitle=false&size=55993&status=done&style=none&taskId=ue1286fab-9804-4a3f-8e8a-35320473737&title=&width=580.0000153647533#averageHue=%2310100e&crop=0&crop=0&crop=1&crop=1&id=BORgx&originHeight=261&originWidth=1044&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

初始化Gitlab:

```shell
[root@git-server ~]# gitlab-ctl reconfigure   #重新加载，需要等很长时间
```

...

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672067657752-d408bd13-4941-4ebb-9b00-438122aa6820.png#averageHue=%23141312&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=134&id=uafd151c7&name=image.png&originHeight=241&originWidth=1225&originalType=binary&ratio=1&rotation=0&showTitle=false&size=129908&status=done&style=none&taskId=u530d2d6a-35fa-4e7b-a9a7-644972d52f5&title=&width=680.5555735841215#averageHue=%23141312&crop=0&crop=0&crop=1&crop=1&id=zIG3P&originHeight=241&originWidth=1225&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672067669220-be72f39b-6b46-42d8-8501-ea95b9a45881.png#averageHue=%23110f0e&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=317&id=u947e373c&name=image.png&originHeight=570&originWidth=1231&originalType=binary&ratio=1&rotation=0&showTitle=false&size=219202&status=done&style=none&taskId=ub44dcf59-fe7a-41ad-9887-9dbbae5e5b8&title=&width=683.8889070057579#averageHue=%23110f0e&crop=0&crop=0&crop=1&crop=1&id=xKDCe&originHeight=570&originWidth=1231&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

...

启动Gitlab服务:

```shell
[root@git-server ~]# gitlab-ctl start  #启动
```

<img src="https://cdn.nlark.com/yuque/0/2022/png/25576613/1672067681540-8fb6438a-8390-41f1-8f8a-a9a65453ecf8.png#averageHue=%23171412&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=427&id=uc87638da&name=image.png&originHeight=769&originWidth=1007&originalType=binary&ratio=1&rotation=0&showTitle=false&size=251975&status=done&style=none&taskId=u76cec043-3fd7-46b4-ad7d-86e8ea1408d&title=&width=559.4444592646614#averageHue=%23171412&crop=0&crop=0&crop=1&crop=1&id=d25sT&originHeight=769&originWidth=1007&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=" alt="image.png" style="zoom:50%;" />

Gitlab 设置 HTTPS 方式

```
如果想要以上的 https 方式正常生效使用，则需要把 letsencrypt 自动生成证书的配置打开，这样在执行重
新让配置生效命令 (gitlab-ctl reconfigure) 的时候会自动给域名生成免费的证书并自动在 gitlab 自带的
 nginx 中加上相关的跳转配置，都是全自动的，非常方便。
letsencrypt['enable'] = true 
letsencrypt['contact_emails'] = ['caryyu@qq.com']     # 这应该是一组要添加为联系人的电子邮件地址
```

测试访问:[http://192.168.246.214](http://192.168.246.214/)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672067705448-e340d890-34cb-4f46-a421-6439a8f46e55.png#averageHue=%23fcfafa&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=353&id=u1390a6d4&name=image.png&originHeight=636&originWidth=1145&originalType=binary&ratio=1&rotation=0&showTitle=false&size=98170&status=done&style=none&taskId=udfc7e88b-08d7-49b7-bd66-579593a9386&title=&width=636.1111279623012#averageHue=%23fcfafa&crop=0&crop=0&crop=1&crop=1&id=dVPvF&originHeight=636&originWidth=1145&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672067715681-e717bfc9-4df6-4291-b0ff-ce9634b93b6e.png#averageHue=%23fcf9f9&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=328&id=u51f1b99d&name=image.png&originHeight=591&originWidth=1173&originalType=binary&ratio=1&rotation=0&showTitle=false&size=87933&status=done&style=none&taskId=u83b19dd4-d634-44b8-a624-a8467a8afd6&title=&width=651.6666839299384#averageHue=%23fcf9f9&crop=0&crop=0&crop=1&crop=1&id=i534V&originHeight=591&originWidth=1173&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

用户为:root

密码:本人设置的密码是12345678

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672067755384-5860bd96-45ea-4b6e-8346-fbbc7b3bba0f.png#averageHue=%23fdfcfc&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=412&id=ubfd55fc8&name=image.png&originHeight=741&originWidth=1610&originalType=binary&ratio=1&rotation=0&showTitle=false&size=73279&status=done&style=none&taskId=u3a6d3b05-7ec8-42d2-bcd4-77cc4531beb&title=&width=894.444468139131#averageHue=%23fdfcfc&crop=0&crop=0&crop=1&crop=1&id=KBFkC&originHeight=741&originWidth=1610&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672067768866-c91538cd-501d-4d7f-890c-c21d8c0a4dd5.png#averageHue=%23fcfcfc&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=346&id=uc8ec40a2&name=image.png&originHeight=622&originWidth=1218&originalType=binary&ratio=1&rotation=0&showTitle=false&size=86908&status=done&style=none&taskId=u4aa73bb3-a8a4-4b7b-af98-92b48457386&title=&width=676.6666845922122#averageHue=%23fcfcfc&crop=0&crop=0&crop=1&crop=1&id=ah8cK&originHeight=622&originWidth=1218&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672067798757-f03e88ad-41bf-4d2a-8119-c0245a54e580.png#averageHue=%23fdfcfb&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=336&id=ued86df59&name=image.png&originHeight=605&originWidth=1224&originalType=binary&ratio=1&rotation=0&showTitle=false&size=134172&status=done&style=none&taskId=u1d65b28d-7afa-4abf-9c43-03b366890dc&title=&width=680.0000180138487#averageHue=%23fdfcfb&crop=0&crop=0&crop=1&crop=1&id=ZHDC9&originHeight=605&originWidth=1224&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672067824628-69e248a3-3fb0-45c1-8690-eef94597d4f6.png#averageHue=%23fbfafa&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=283&id=u57d38e2f&name=image.png&originHeight=509&originWidth=1183&originalType=binary&ratio=1&rotation=0&showTitle=false&size=103464&status=done&style=none&taskId=u8dfde65a-abbe-464b-beab-ad637834c58&title=&width=657.2222396326658#averageHue=%23fbfafa&crop=0&crop=0&crop=1&crop=1&id=cwIuK&originHeight=509&originWidth=1183&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672067841273-d5b0e6f8-0002-4b8d-a6d0-ab374399b2cb.png#averageHue=%23f9f5ef&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=356&id=ueac9b02f&name=image.png&originHeight=641&originWidth=1216&originalType=binary&ratio=1&rotation=0&showTitle=false&size=145409&status=done&style=none&taskId=u41f43f7c-b3bb-4c43-a882-cc7a5696ecd&title=&width=675.5555734516666#averageHue=%23f9f5ef&crop=0&crop=0&crop=1&crop=1&id=j4fqz&originHeight=641&originWidth=1216&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

需要创建秘钥

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672067854967-b1c45e85-29da-4c98-8ee4-2500966c4cb6.png#averageHue=%23fbfafa&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=362&id=u18fbc698&name=image.png&originHeight=652&originWidth=1042&originalType=binary&ratio=1&rotation=0&showTitle=false&size=47367&status=done&style=none&taskId=ue1a3aded-0b7e-47ef-be0f-417f23eb070&title=&width=578.8889042242077#averageHue=%23fbfafa&crop=0&crop=0&crop=1&crop=1&id=PlSAy&originHeight=652&originWidth=1042&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672067869483-d2d135f7-607b-4247-8b7a-8350142d6d92.png#averageHue=%23fcfbfa&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=519&id=ub0514af4&name=image.png&originHeight=934&originWidth=1830&originalType=binary&ratio=1&rotation=0&showTitle=false&size=121351&status=done&style=none&taskId=ua5ed2b2f-ac39-45ff-86fe-7fdc0b5c27b&title=&width=1016.6666935991365#averageHue=%23fcfbfa&crop=0&crop=0&crop=1&crop=1&id=cJBIt&originHeight=934&originWidth=1830&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

```shell
[root@client ~]# ssh-keygen
[root@client ~]# cd .ssh/
[root@client .ssh]# ls 
[root@client .ssh]# cat id_rsa.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC0HeO8gaua13h9HCJK5RXVc/cjet9DpLYq2dqmQ0LXfP0Vwj6YjKxu7lE1i/4Y2cmu5lDe8bG22aikyaW38Fnz0bkGfEurdVZii/KCiHBz2lXS1ocuAdloJT4wnc2MUjh/gwc4FuNkWdYvpbMLXSSHIVjv8vB9YbHlPMTMy5N89kMwMmta5C87/8fBO5VtGijgGOueywM+xAzovlfoJbprV/ZBKkhiskSKz4fHyoGFGwllX3kMkNR/soGF5XXA+/99iO3UqSaloF0UzfUCgasMfMVB5zDHGIB6uTrMe6ccfKp9gnVyD7m4Zmk7MwouBwAfMLIiHmvekBGXqb1YCTgJ root@client
```

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672067892446-c73ee83d-899a-4b0d-8183-f6269b833082.png#averageHue=%23fcfbfb&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=629&id=u1ebfe36e&name=image.png&originHeight=1133&originWidth=2057&originalType=binary&ratio=1&rotation=0&showTitle=false&size=133816&status=done&style=none&taskId=u75e1f432-84e9-43b8-9394-c772fef86bf&title=&width=1142.7778080510514#averageHue=%23fcfbfb&crop=0&crop=0&crop=1&crop=1&id=rXFB6&originHeight=1133&originWidth=2057&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672067917934-0ed1f7c8-307f-46cb-bbb9-17d29b26a186.png#averageHue=%23fcfcfb&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=377&id=ubb759c6e&name=image.png&originHeight=679&originWidth=2207&originalType=binary&ratio=1&rotation=0&showTitle=false&size=44190&status=done&style=none&taskId=u2f648cba-ebc8-47fa-9c42-489e7aa3017&title=&width=1226.111143591964#averageHue=%23fcfcfb&crop=0&crop=0&crop=1&crop=1&id=JrvTE&originHeight=679&originWidth=2207&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672067938114-09c9ca9a-c92f-4842-9095-624208bea5ab.png#averageHue=%23fcfbfb&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=622&id=u21c0a048&name=image.png&originHeight=1119&originWidth=2218&originalType=binary&ratio=1&rotation=0&showTitle=false&size=102601&status=done&style=none&taskId=u97ac071b-fc57-49fe-bc06-dd79a271ba9&title=&width=1232.2222548649643#averageHue=%23fcfbfb&crop=0&crop=0&crop=1&crop=1&id=nvNf4&originHeight=1119&originWidth=2218&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672067953748-04ef395e-728b-4895-8393-036b6b79805d.png#averageHue=%23eeece9&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=301&id=u848d7ab5&name=image.png&originHeight=541&originWidth=1486&originalType=binary&ratio=1&rotation=0&showTitle=false&size=37364&status=done&style=none&taskId=u1efa251c-6519-42d0-97b8-d293a0d9b6f&title=&width=825.5555774253097#averageHue=%23eeece9&crop=0&crop=0&crop=1&crop=1&id=szBEs&originHeight=541&originWidth=1486&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672067975464-15b99894-a293-4acc-8882-d7d606833e46.png#averageHue=%23f5f4f4&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=487&id=u8e0c5329&name=image.png&originHeight=877&originWidth=1746&originalType=binary&ratio=1&rotation=0&showTitle=false&size=55324&status=done&style=none&taskId=u9e8d7611-0ecc-4ad9-850f-c7c666bc15e&title=&width=970.0000256962253#averageHue=%23f5f4f4&crop=0&crop=0&crop=1&crop=1&id=Melwx&originHeight=877&originWidth=1746&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672067991261-d09cae42-20fa-4b76-b250-7af3e72bd024.png#averageHue=%23f9f8f7&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=479&id=u303d2440&name=image.png&originHeight=862&originWidth=1848&originalType=binary&ratio=1&rotation=0&showTitle=false&size=46445&status=done&style=none&taskId=uc3ca832c-5302-4f3b-a9da-19c98bbdd16&title=&width=1026.666693864046#averageHue=%23f9f8f7&crop=0&crop=0&crop=1&crop=1&id=uXThj&originHeight=862&originWidth=1848&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068008979-5274159b-5bf7-4e2a-abbd-d8c7dabc88c7.png#averageHue=%23f7f6f6&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=612&id=u617ee5cc&name=image.png&originHeight=1102&originWidth=1584&originalType=binary&ratio=1&rotation=0&showTitle=false&size=66036&status=done&style=none&taskId=u67225e51-0ad6-41ed-bfab-31034c1a8e5&title=&width=880.0000233120395#averageHue=%23f7f6f6&crop=0&crop=0&crop=1&crop=1&id=WYqSH&originHeight=1102&originWidth=1584&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068027450-574a9b38-0545-4d4b-bf64-a7eebb7767d7.png#averageHue=%23faf9f9&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=595&id=u2672482b&name=image.png&originHeight=1071&originWidth=1170&originalType=binary&ratio=1&rotation=0&showTitle=false&size=40710&status=done&style=none&taskId=u02d30fe4-4d0e-41e5-bc74-cfafeda3467&title=&width=650.0000172191201#averageHue=%23faf9f9&crop=0&crop=0&crop=1&crop=1&id=CrF1B&originHeight=1071&originWidth=1170&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

创建一个文件:

1.newfile:先新建一个文件。
2.uploadfile:再上传即可。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068042838-e5991d0f-6409-4461-99d4-f5c982365ab5.png#averageHue=%23fdfcfc&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=504&id=ub793064b&name=image.png&originHeight=908&originWidth=2375&originalType=binary&ratio=1&rotation=0&showTitle=false&size=74552&status=done&style=none&taskId=u9269d482-3027-40a5-99fd-8f08cd44739&title=&width=1319.4444793977864#averageHue=%23fdfcfc&crop=0&crop=0&crop=1&crop=1&id=ZLNhY&originHeight=908&originWidth=2375&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068063236-f92fc0be-67c4-4755-88af-8987cab91bba.png#averageHue=%23fdfdfd&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=746&id=u3c6b924f&name=image.png&originHeight=1342&originWidth=2169&originalType=binary&ratio=1&rotation=0&showTitle=false&size=40675&status=done&style=none&taskId=u54441f50-e778-461a-9101-4c7026ef9c9&title=&width=1205.0000319215994#averageHue=%23fdfdfd&crop=0&crop=0&crop=1&crop=1&id=YvlLV&originHeight=1342&originWidth=2169&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068082410-c99e3fcb-e246-4e12-af34-d5f6ec33824f.png#averageHue=%23fbfafa&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=334&id=ufdcd336d&name=image.png&originHeight=601&originWidth=2184&originalType=binary&ratio=1&rotation=0&showTitle=false&size=39349&status=done&style=none&taskId=udd755bdd-d4e4-407a-8726-817832c975c&title=&width=1213.3333654756907#averageHue=%23fbfafa&crop=0&crop=0&crop=1&crop=1&id=Ea6DV&originHeight=601&originWidth=2184&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

上传一个文件：

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068099677-52482bb7-8891-429e-8219-4be8ad4b4275.png#averageHue=%23fbfaf9&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=331&id=u42612afe&name=image.png&originHeight=596&originWidth=1775&originalType=binary&ratio=1&rotation=0&showTitle=false&size=37349&status=done&style=none&taskId=u709b7fb8-d765-4c6a-b695-6ba589acc94&title=&width=986.1111372341352#averageHue=%23fbfaf9&crop=0&crop=0&crop=1&crop=1&id=lXAo3&originHeight=596&originWidth=1775&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068119358-06faa2f6-9b56-46ab-be55-49f8da93a648.png#averageHue=%23fcfbfb&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=512&id=u8c31d6cb&name=image.png&originHeight=921&originWidth=1935&originalType=binary&ratio=1&rotation=0&showTitle=false&size=70369&status=done&style=none&taskId=u17dca5a7-fe82-458d-a3a8-c42ec85630c&title=&width=1075.0000284777755#averageHue=%23fcfbfb&crop=0&crop=0&crop=1&crop=1&id=OHNJw&originHeight=921&originWidth=1935&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068138908-dab45e74-6496-4baa-a85e-e265ce14479a.png#averageHue=%23fcfcfc&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=483&id=ud5c26dbf&name=image.png&originHeight=870&originWidth=1391&originalType=binary&ratio=1&rotation=0&showTitle=false&size=26857&status=done&style=none&taskId=u75941999-6b1c-4d3e-a92f-c4ddc166c9f&title=&width=772.7777982493983#averageHue=%23fcfcfc&crop=0&crop=0&crop=1&crop=1&id=SWo6S&originHeight=870&originWidth=1391&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068163516-f4084ee9-8f7c-4ecb-86e8-d2c8ac62c3f0.png#averageHue=%23dfdedd&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=607&id=u733d1e90&name=image.png&originHeight=1092&originWidth=2127&originalType=binary&ratio=1&rotation=0&showTitle=false&size=145733&status=done&style=none&taskId=ua0c57b92-d926-41ee-a2b7-5327692eec7&title=&width=1181.6666979701438#averageHue=%23dfdedd&crop=0&crop=0&crop=1&crop=1&id=LDFLI&originHeight=1092&originWidth=2127&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068179556-e8e425cc-19fd-427c-802d-8cce41af02de.png#averageHue=%23fcfcfc&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=545&id=u0ea0d347&name=image.png&originHeight=981&originWidth=1536&originalType=binary&ratio=1&rotation=0&showTitle=false&size=27332&status=done&style=none&taskId=u9999d75d-2838-43f8-85e0-9322a416818&title=&width=853.3333559389473#averageHue=%23fcfcfc&crop=0&crop=0&crop=1&crop=1&id=m28SX&originHeight=981&originWidth=1536&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068196396-11fe48cb-636e-4817-b220-4ba88cd4dc69.png#averageHue=%23fbfbfa&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=560&id=ub372bf1e&name=image.png&originHeight=1008&originWidth=2341&originalType=binary&ratio=1&rotation=0&showTitle=false&size=70481&status=done&style=none&taskId=u00e36b61-0340-4e70-8986-f0e25312c92&title=&width=1300.5555900085128#averageHue=%23fbfbfa&crop=0&crop=0&crop=1&crop=1&id=Q8lYQ&originHeight=1008&originWidth=2341&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068212089-51a911ad-632b-42c5-8fbb-c60839407002.png#averageHue=%23fbfbfb&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=362&id=u93355925&name=image.png&originHeight=651&originWidth=1907&originalType=binary&ratio=1&rotation=0&showTitle=false&size=36420&status=done&style=none&taskId=u9110db63-a188-4cf9-b36e-3e6168f3350&title=&width=1059.4444725101384#averageHue=%23fbfbfb&crop=0&crop=0&crop=1&crop=1&id=fGPzF&originHeight=651&originWidth=1907&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068228327-6ffd1130-a21d-48f3-8c5b-10d03e80d408.png#averageHue=%23fbfafa&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=476&id=u2901f154&name=image.png&originHeight=856&originWidth=1890&originalType=binary&ratio=1&rotation=0&showTitle=false&size=61360&status=done&style=none&taskId=u2ca2abb7-1b90-45a0-83a4-f9fca330122&title=&width=1050.0000278155017#averageHue=%23fbfafa&crop=0&crop=0&crop=1&crop=1&id=zi5jX&originHeight=856&originWidth=1890&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068243736-e1b91ebc-96b0-4ce9-92c5-1e16df1bc7ff.png#averageHue=%23fbfbfb&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=319&id=u895a9599&name=image.png&originHeight=575&originWidth=1892&originalType=binary&ratio=1&rotation=0&showTitle=false&size=36518&status=done&style=none&taskId=u2be8d166-611c-4142-a830-fa7fee35a7d&title=&width=1051.1111389560472#averageHue=%23fbfbfb&crop=0&crop=0&crop=1&crop=1&id=OJOUD&originHeight=575&originWidth=1892&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068259357-3ebe7b6c-3ad6-4446-959b-df2cb1815837.png#averageHue=%23faf9f9&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=332&id=u9af2bcb2&name=image.png&originHeight=597&originWidth=2354&originalType=binary&ratio=1&rotation=0&showTitle=false&size=44287&status=done&style=none&taskId=u0b665905-57c9-4d1e-94cf-955c7d99e26&title=&width=1307.7778124220586#averageHue=%23faf9f9&crop=0&crop=0&crop=1&crop=1&id=wURXg&originHeight=597&originWidth=2354&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068274362-39d803ae-25d3-4449-ba6e-277098262725.png#averageHue=%23fafafa&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=281&id=u3d80ccd7&name=image.png&originHeight=505&originWidth=730&originalType=binary&ratio=1&rotation=0&showTitle=false&size=12223&status=done&style=none&taskId=ufb303710-c611-426d-b352-9884e6555a7&title=&width=405.5555662991091#averageHue=%23fafafa&crop=0&crop=0&crop=1&crop=1&id=wDPW6&originHeight=505&originWidth=730&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068289475-f085389b-212b-4ac6-b91b-e34be97769c8.png#averageHue=%23f9f1e9&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=333&id=u75deb4c5&name=image.png&originHeight=600&originWidth=918&originalType=binary&ratio=1&rotation=0&showTitle=false&size=19516&status=done&style=none&taskId=ucf460754-2fae-4dcc-b669-34ec5c154be&title=&width=510.0000135103865#averageHue=%23f9f1e9&crop=0&crop=0&crop=1&crop=1&id=wnOPk&originHeight=600&originWidth=918&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

## 新建普通用户

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068303798-be17659a-5d3f-48d3-9b09-4b6374807c5d.png#averageHue=%23fcfcfc&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=618&id=uf1c19e1c&name=image.png&originHeight=1113&originWidth=1747&originalType=binary&ratio=1&rotation=0&showTitle=false&size=71902&status=done&style=none&taskId=u58fdb145-7f83-4917-b528-44fd1d1f103&title=&width=970.555581266498#averageHue=%23fcfcfc&crop=0&crop=0&crop=1&crop=1&id=PQCbX&originHeight=1113&originWidth=1747&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068319300-2bb3b410-0afa-48e5-afaa-c8466e1dafec.png#averageHue=%23fcfcfb&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=637&id=u2fc59234&name=image.png&originHeight=1146&originWidth=1730&originalType=binary&ratio=1&rotation=0&showTitle=false&size=81714&status=done&style=none&taskId=u8e64e70e-3852-4e43-b2f9-fa08ad9bf9a&title=&width=961.1111365718613#averageHue=%23fcfcfb&crop=0&crop=0&crop=1&crop=1&id=KR4t3&originHeight=1146&originWidth=1730&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068333282-beb24990-fe82-426c-8583-a539632ea903.png#averageHue=%23fcfbfb&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=529&id=u6c117486&name=image.png&originHeight=952&originWidth=1928&originalType=binary&ratio=1&rotation=0&showTitle=false&size=71045&status=done&style=none&taskId=u5391480c-8915-46b4-ba2b-31a46f40e66&title=&width=1071.1111394858663#averageHue=%23fcfbfb&crop=0&crop=0&crop=1&crop=1&id=ovFZw&originHeight=952&originWidth=1928&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068348828-1d00b638-fbb9-44ed-baef-8189609a52c8.png#averageHue=%23fdfdfc&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=579&id=u4209140b&name=image.png&originHeight=1043&originWidth=1882&originalType=binary&ratio=1&rotation=0&showTitle=false&size=65588&status=done&style=none&taskId=uca4d13e9-f478-4a37-9b33-9e8b2cec7a4&title=&width=1045.5555832533196#averageHue=%23fdfdfc&crop=0&crop=0&crop=1&crop=1&id=e1j6Z&originHeight=1043&originWidth=1882&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

新添加的用户创建成功！

## 在git客户端

```shell
[root@client ~]# git clone git@192.168.246.214:root/testapp.git
Cloning into 'testapp'...
remote: Enumerating objects: 6, done.
remote: Counting objects: 100% (6/6), done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 6 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (6/6), done.
[root@client ~]# ls
testapp
[root@client ~]# cd testapp/
[root@client testapp]# ls
test.txt  同步时间.txt
[root@client testapp]#
使用http的
[root@client ~]# rm -rf testgit/
[root@client ~]# git clone http://192.168.246.214/root/testapp.git
Cloning into 'testapp'...
Username for 'http://192.168.246.214': root
Password for 'http://root@192.168.246.214':12345678  #为自己设置的密码
remote: Enumerating objects: 6, done.
remote: Counting objects: 100% (6/6), done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 6 (delta 0), reused 0 (delta 0)
Unpacking objects: 100% (6/6), done.
[root@client ~]# ls
testapp
```

```shell
提交到远程gitlab仓库
[root@client testapp]# vim update.txt
1000phone
[root@client testapp]# git add .
[root@client testapp]# git commit -m "update_version1"
[master 091798d] update_version1
 1 file changed, 2 insertions(+)
 create mode 100644 update.txt
[root@client testapp]# git push origin master
Username for 'http://192.168.62.166': ^C
[root@nginx-server testapp2]# git push origin master
Username for 'http://192.168.62.166': root
Password for 'http://root@192.168.62.166': 
Counting objects: 4, done.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 307 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To http://192.168.62.166/root/testapp2.git
   201f479..091798d  master -> master
```

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068368384-de9e64e5-1b9d-4827-96d3-aedf2793a80a.png#averageHue=%23fbfafa&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=361&id=ud47e048a&name=image.png&originHeight=650&originWidth=1466&originalType=binary&ratio=1&rotation=0&showTitle=false&size=30574&status=done&style=none&taskId=u263fa728-3368-4771-a91c-4c2bb215f13&title=&width=814.4444660198548#averageHue=%23fbfafa&crop=0&crop=0&crop=1&crop=1&id=NSHi7&originHeight=650&originWidth=1466&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

## 调整上传文件的大小

## ![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068384136-4a29c0b5-02f5-4ac3-b9aa-670faa118bb0.png#averageHue=%23f9f8f7&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=449&id=u4a7d00aa&name=image.png&originHeight=808&originWidth=1810&originalType=binary&ratio=1&rotation=0&showTitle=false&size=63809&status=done&style=none&taskId=uc11bfdc2-52fa-4978-8ded-0d01a2d009f&title=&width=1005.5555821936814#averageHue=%23f9f8f7&crop=0&crop=0&crop=1&crop=1&id=AAKzH&originHeight=808&originWidth=1810&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068398668-c1ff72d0-2be1-4abe-b1b9-f4005a531209.png#averageHue=%23f2f1f0&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=636&id=ud4bf920e&name=image.png&originHeight=1144&originWidth=671&originalType=binary&ratio=1&rotation=0&showTitle=false&size=39566&status=done&style=none&taskId=u690755d9-e022-49a2-8a9c-61006da1b51&title=&width=372.77778765301673#averageHue=%23f2f1f0&crop=0&crop=0&crop=1&crop=1&id=ubECe&originHeight=1144&originWidth=671&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

默认是10M，可根据情况调整

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068412573-8fb23255-cfca-456b-9041-1244938600aa.png#averageHue=%23fcfafa&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=383&id=u55df346a&name=image.png&originHeight=690&originWidth=1923&originalType=binary&ratio=1&rotation=0&showTitle=false&size=46585&status=done&style=none&taskId=u38c4dfec-0d68-4fdf-abef-cd83d996133&title=&width=1068.3333616345024#averageHue=%23fcfafa&crop=0&crop=0&crop=1&crop=1&id=JlaV6&originHeight=690&originWidth=1923&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068426805-92367c0d-d96f-459e-bf87-e9cfa9bcbdf8.png#averageHue=%23fdfcfb&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=487&id=u6cbf9af8&name=image.png&originHeight=876&originWidth=1964&originalType=binary&ratio=1&rotation=0&showTitle=false&size=48188&status=done&style=none&taskId=u49b62c65-ef49-40a1-8eff-aae60848417&title=&width=1091.1111400156854#averageHue=%23fdfcfb&crop=0&crop=0&crop=1&crop=1&id=ZCRWj&originHeight=876&originWidth=1964&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

```shell
拓展：
1.cat /proc/swaps 查看swap分区是否启动（无）
2.创建 ：
dd if=/dev/zero of=/data/swap bs=512 count=8388616
创建swap大小为bs*count=4294971392(4G)；
/data/swap目录若无则找/mnt/swap
3.通过mkswap命令将上述空间制作成swap分区：
mkswap /data/swap
4.查看内核参数vm.swappiness中的数值是否为0，如果为0则根据实际需要调		 整成60：
查看： cat /proc/sys/vm/swappiness
设置： sysctl -w vm.swappiness=60
若想永久修改，则编辑/etc/sysctl.conf文件，改文件中有vm.swappiness变量配置，默认为0
5.启用分区：
swapon /data/swap
echo “/data/swap swap swap defaults 0 0” >> /etc/fstab
6.再次使用cat /proc/swaps 查看swap分区是否启动
```

# Gitlab 备份与恢复

## 1、查看系统版本和软件版本

```shell
[root@git-server ~]# cat /etc/redhat-release 
CentOS Linux release 7.4.1708 (Core)

[root@git-server ~]# cat /opt/gitlab/embedded/service/gitlab-rails/VERSION
8.15.4
```

## 2、数据备份

打开/etc/gitlab/gitlab.rb配置文件，查看一个和备份相关的配置项：

```shell
[root@git-server backups]# vim /etc/gitlab/gitlab.rb
gitlab_rails['backup_path'] = "/var/opt/gitlab/backups"	#备份的路径
gitlab_rails['backup_archive_permissions'] = 0644		#备份文件的默认权限
gitlab_rails['backup_keep_time'] = 604800				#保留时长，秒为单位
```

设置备份保留时常，防止每天执行备份，肯定有目录被爆满的风险，打开/etc/gitlab/gitlab.rb配置文件，找到如下配置

该项定义了默认备份出文件的路径，可以通过修改该配置，并执行 **gitlab-ctl reconfigure 或者 gitlab-ctl  restart** 重启服务生效。

```shell
[root@git-server backups]# gitlab-ctl reconfigure
或者
[root@git-server backups]# gitlab-ctl  restart
```

执行备份命令进行备份

```shell
[root@git-server backups]# /opt/gitlab/bin/gitlab-rake gitlab:backup:create
```

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068442914-cd516f5b-cadc-402b-ac52-b332911dff43.png#averageHue=%23102a4f&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=317&id=u4568c064&name=image.png&originHeight=571&originWidth=1220&originalType=binary&ratio=1&rotation=0&showTitle=false&size=188050&status=done&style=none&taskId=udb7f858e-a246-4c3d-8bb7-c1a2d033bcd&title=&width=677.7777957327577#averageHue=%23102a4f&crop=0&crop=0&crop=1&crop=1&id=x5y2W&originHeight=571&originWidth=1220&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

也可以添加到 crontab 中定时执行：

```shell
0 2 * * * /opt/gitlab/bin/gitlab-rake gitlab:backup:create
```

可以到/var/opt/gitlab/backups找到备份包，解压查看，会发现备份的还是比较全面的，数据库、repositories、build、upload等分类还是比较清晰的。

备份完成，会在备份目录中生成一个当天日期的tar包。

```shell
[root@git-server ~]# ll /var/opt/gitlab/backups/
```

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068461187-3642608a-6c3b-4905-ac9e-588b5f183054.png#averageHue=%23172f51&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=86&id=u33fa3b11&name=image.png&originHeight=154&originWidth=1889&originalType=binary&ratio=1&rotation=0&showTitle=false&size=15014&status=done&style=none&taskId=u80617b6d-d41b-40d3-8cde-81c2f509135&title=&width=1049.4444722452288#averageHue=%23172f51&crop=0&crop=0&crop=1&crop=1&id=PaVNN&originHeight=154&originWidth=1889&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

## 3、数据恢复

特别注意：

- 备份目录和gitlab.rb中定义的备份目录必须一致
- GitLab的版本和备份文件中的版本必须一致，否则还原时会报错。

**在恢复之前，可以删除一个文件，以便查看效果**

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068477607-168c1761-dacd-4a06-aa1e-6fd6e1dd59e2.png#averageHue=%23fbfafa&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=311&id=u3160965a&name=image.png&originHeight=560&originWidth=2188&originalType=binary&ratio=1&rotation=0&showTitle=false&size=39042&status=done&style=none&taskId=uf9b4739d-14ac-4644-b021-28beb9f4ee1&title=&width=1215.5555877567817#averageHue=%23fbfafa&crop=0&crop=0&crop=1&crop=1&id=pohO7&originHeight=560&originWidth=2188&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

执行恢复操作：

```shell
[root@git-server ~]# cd /var/opt/gitlab/backups
[root@git-server  backups]# gitlab-rake gitlab:backup:restore BACKUP=1588700546_2020_05_06_12.6.3
注意恢复文件的名称
```

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068486798-52440d8c-67d8-4cbd-bc0e-07eb328974b8.png#averageHue=%230f2a4f&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=143&id=u5d1e4cf7&name=image.png&originHeight=257&originWidth=1223&originalType=binary&ratio=1&rotation=0&showTitle=false&size=101178&status=done&style=none&taskId=u02e43966-1795-4b0e-8d4f-da32d454163&title=&width=679.444462443576#averageHue=%230f2a4f&crop=0&crop=0&crop=1&crop=1&id=ZBQ8N&originHeight=257&originWidth=1223&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068493462-b1aea56d-a6dc-43ca-8f26-b0583bf62e88.png#averageHue=%23132e54&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=348&id=u59defd21&name=image.png&originHeight=627&originWidth=1224&originalType=binary&ratio=1&rotation=0&showTitle=false&size=318039&status=done&style=none&taskId=u0df0a3d2-97d4-4471-a14a-037e2077697&title=&width=680.0000180138487#averageHue=%23132e54&crop=0&crop=0&crop=1&crop=1&id=oP026&originHeight=627&originWidth=1224&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068499860-1f04be55-298f-4a77-a2e0-ec8c66ff65ef.png#averageHue=%230e294e&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=224&id=u8a54047f&name=image.png&originHeight=404&originWidth=1213&originalType=binary&ratio=1&rotation=0&showTitle=false&size=80879&status=done&style=none&taskId=uaa0a6869-c03e-4b82-8aaf-83a58172d34&title=&width=673.8889067408484#averageHue=%230e294e&crop=0&crop=0&crop=1&crop=1&id=JOmgJ&originHeight=404&originWidth=1213&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

恢复完成后，启动刚刚的两个服务，或者重启所有服务，再打开浏览器进行访问，发现数据和之前的一致：

**注意：通过备份文件恢复gitlab必须保证两台主机的gitlab版本一致，否则会提示版本不匹配**

查看gitlab端，可以看到数据恢复成功

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068516549-6fa4cc8b-48f8-4943-801d-a4d6dbe81298.png#averageHue=%23f9f9f8&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=428&id=u900e129f&name=image.png&originHeight=770&originWidth=2042&originalType=binary&ratio=1&rotation=0&showTitle=false&size=74647&status=done&style=none&taskId=ubc083758-ccc4-4d43-9fd5-c12997b0947&title=&width=1134.4444744969599#averageHue=%23f9f9f8&crop=0&crop=0&crop=1&crop=1&id=pfqkh&originHeight=770&originWidth=2042&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

# Github 远程仓库

1、github.com 注册账户

2、在github上创建仓库

本人账户：

用户名：yangge

邮箱：  [850439713@qq.com](mailto:850439713@qq.com)

密码：

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672126436003-5c982918-771d-4272-b905-d96b69f7c833.png#averageHue=%231b1d29&clientId=ub0265ce7-51f5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=669&id=u821f2816&name=image.png&originHeight=1338&originWidth=2337&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1512324&status=done&style=none&taskId=uf4844147-8bd2-4068-bc3a-23a8a3201a8&title=&width=1168.5)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672126415823-e7acdaa4-8216-4ddf-b95d-22fd03ce6654.png#averageHue=%23181b1f&clientId=ub0265ce7-51f5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=751&id=u2cdd92fe&name=image.png&originHeight=1352&originWidth=1448&originalType=binary&ratio=1&rotation=0&showTitle=false&size=147559&status=done&style=none&taskId=u396f62a4-9dfe-424b-a0de-8098ac77060&title=&width=804.4444657549452)

3、客户端生成本地ssh key

```shell
[root@localhost ~]# ssh-keygen -t rsa -C 'meteor@163.com' # 邮箱要与github上注册的相同
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:RiE6UR1BtzV5avyE2uz6TNPsVHa2D2eHprghrJEkd/g meteor@163.com
The key's randomart image is:
+---[RSA 2048]----+
|    ..oo=o. o.   |
|     o ..o o...  |
|    o   . .. +   |
|     . o    = .  |
|    . + S  = o  =|
|     + *  . oo.=o|
|      o E ..o B.+|
|       o . =.* +o|
|      .   +++ . .|
+----[SHA256]-----+
[root@localhost ~]#
[root@localhost ~]# cat .ssh/id_rsa.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDVThfq4brrlsPGtAknVB0TLPx+7Dd3qlxTbSIrUOsGC5Y8JuNqVTlIntZB4oNj8cSQrWvec9CKm0a8o7WwaJIiqpxurz+YpQHP2KbapftKIxsX4hPf/z+p0El1U6arQa35/xmNsq+cJLH/bDdRG+EMDhuCBmjVZOlLj/hEdeIT6s56AnnCkaWoF+sq58KCF7Tk54jRbs/YiyE4SN7FuA70r+07sA/uj0+lmuk4E190KtQUELhjX/E9stivlqiRhxnKvVUqXDywsjfM8Rtvbi4Fg9R8Wt9fpd4QwnWksYUoR5qZJFYXO4hSZrUnSMruPK14xXjDJcFDcP2eHIzKgLD1 meteor@163.com
```

4、复制以上的公钥，在github 中添加ssh key

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672126398447-6ecf2c8d-3e30-47bf-9255-4ca56d8a3b21.png#averageHue=%2315191e&clientId=ub0265ce7-51f5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=671&id=uc7898488&name=image.png&originHeight=1208&originWidth=2693&originalType=binary&ratio=1&rotation=0&showTitle=false&size=349357&status=done&style=none&taskId=u3438ed9a-666e-4244-ab42-9a76cd142b9&title=&width=1496.1111507445216)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672126385558-93b86e3c-8be1-40e1-b09e-84a3e27277fe.png#averageHue=%23181a1f&clientId=ub0265ce7-51f5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=696&id=ufceae4a3&name=image.png&originHeight=1252&originWidth=2592&originalType=binary&ratio=1&rotation=0&showTitle=false&size=199610&status=done&style=none&taskId=u77a3f370-95b2-4d3a-81a2-c95d32dc4a1&title=&width=1440.0000381469736)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672126370425-d4449529-51b0-48b5-9758-a4ce4458677c.png#averageHue=%230f1116&clientId=ub0265ce7-51f5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=346&id=u3bd3e5c5&name=image.png&originHeight=623&originWidth=1258&originalType=binary&ratio=1&rotation=0&showTitle=false&size=96845&status=done&style=none&taskId=ufce0f931-2bf9-4950-8bb8-224b743fa0d&title=&width=698.8889074031223)

创建好库之后，在库里创建几个文件，方便测试

5、测试：拉取github仓库

```shell
[root@localhost ~]# yum install git
[root@localhost ~]# git config --global user.name 'meteor_by'
[root@localhost ~]# git config --global user.email 'meteor@163.com'
[root@localhost tmp]# cd /tmp

[root@localhost tmp]# git clone git@github.com:yangge666a/yangge666.git
```

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672126357977-df96e28a-00fc-48ff-85b4-747c9df4be1b.png#averageHue=%23181615&clientId=ub0265ce7-51f5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=222&id=u93fe949b&name=image.png&originHeight=399&originWidth=1260&originalType=binary&ratio=1&rotation=0&showTitle=false&size=276774&status=done&style=none&taskId=uad8289f3-2d9f-4376-a79a-e234276d18d&title=&width=700.0000185436678)

7、在本地添加远程仓库，并推送至github仓库

```shell
[root@localhost tmp]# cd /tmp/yangge666/
[root@localhost yangge]# ls
as.txt  README.md
[root@localhost yangge]# cat as.txt 
this is 2002 test file
[root@client yangge]# ls
as.txt  README.md
[root@localhost yangge]# vim pengge666.txt
[root@localhost yangge]# git add .
[root@localhost yangge]# git commit -m "oldsix"
[master 0f6a3de] laoyang
 1 file changed, 2 insertions(+)
 create mode 100644 a.txt
[root@client yangge]# git push origin main
Counting objects: 4, done.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 288 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To git@github.com:yangge/yangge.git
   ba8225d..0f6a3de  master -> master
```

去github界面查看

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672126346987-ed9cdb1a-6fee-4cd0-ba35-b29b1d5516f4.png#averageHue=%23191c21&clientId=ub0265ce7-51f5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=554&id=u5b4d0cec&name=image.png&originHeight=997&originWidth=2288&originalType=binary&ratio=1&rotation=0&showTitle=false&size=153185&status=done&style=none&taskId=u39f268e8-6830-4557-8f5f-64dca30c92e&title=&width=1271.111144784057)

8、连接远程仓库方法

```shell
#[root@localhost testapp]# git remote -v 
#origin	git@github.com:meteor/python1804.git (fetch)
#origin	git@github.com:meteor/python1804.git (push)
#[root@localhost python1804]#
#[root@localhost python1804]#  git remote rm origin  (如果连接远程的方式不是ssh,可以删除重新添加)
#[root@localhost ~]# git remote add origin git@github.com:meteor/python1804.git
#或
#git remote add origin https://github.com/meteor/python1804.git
#git push -u origin master

[root@client yangge]# git remote -v
origin  git@github.com:yangge/yangge.git (fetch)
origin  git@github.com:yangge/yangge.git (push)
[root@client yangge]# git remote rm origin
[root@client yangge]# git remote add origin git@192.168.62.131:root/testapp.git
[root@client yangge]# ls
a.txt  as.txt  README.md
[root@client yangge]# pwd

[root@client ~]# cd /root/testapp/
[root@client testapp]# ls
test.sql  test.txt  update.txt
[root@client testapp]# vim modify.txt
[root@client testapp]# git add .
[root@client testapp]# git commit -m "modify gitlab from github"
[master fde12c2] modify gitlab from github
 1 file changed, 1 insertion(+)
 create mode 100644 modify.txt

[root@client testapp]# git push origin master
Username for 'http://192.168.62.131': root
Password for 'http://root@192.168.62.131': 
Counting objects: 4, done.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 337 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To http://192.168.62.131/root/testapp.git
   23bae45..fde12c2  master -> master
```

去自己部署的gitlab上查看

![image.png](https://cdn.nlark.com/yuque/0/2022/png/25576613/1672068795695-fa0445ce-6f2b-448f-9e30-62b833e1d859.png#averageHue=%23fcfbfb&clientId=u959daffe-061e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=547&id=u05a84419&name=image.png&originHeight=985&originWidth=1720&originalType=binary&ratio=1&rotation=0&showTitle=false&size=82604&status=done&style=none&taskId=ufbb9a182-2659-4667-be55-7b6bd8efb5b&title=&width=955.5555808691338#averageHue=%23fcfbfb&crop=0&crop=0&crop=1&crop=1&id=QwYj1&originHeight=985&originWidth=1720&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
