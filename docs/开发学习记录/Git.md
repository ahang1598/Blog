# 0. 命令速查

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/Git.png)

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/git-cheat-sheet-large01-cn.png)



推荐学习网址
[廖雪峰教程](https://www.liaoxuefeng.com/wiki/896043488029600)
[动画模拟学习](https://learngitbranching.js.org/?locale=zh_CN)

## 0.1 更新git软件版本

`git update-git-for-windows`


## 0.2 设置git
```shell
缺省等同于 local
$ git config --local  该仓库
$ git config --global  本用户所有仓库
$ git config --system  所有登录用户
```

显示 config 的配置，加 --list
```shell
$ git config --list --local
$ git config --list --global
$ git config --list --system
```

配置user.name和user.email
```shell
$ git config --global user.name ‘your_name’
$ git config --global user.email ‘your_email@domain.com’
```

## 0.3 忽略文件上传
对于部分不需要上传的文件：比如java的.class文件、密码信息文件等

在git的本地仓库添加一个`.gitignore`文件
里面加入不想上传的文件

1. `#`  表示注释
2. `/`  目录分割
3. `*`匹配任意内容：
	- `a/*`：`a/a`, `a/a/a`(官方文档说不可以匹配`/`，但是测试发现可以匹配)
	- `a.*` ：`a.a`， `a.`
4. 字符 `?`匹配除 `/`之外的任何一个字符
5. `**`匹配任意路径
	- `**/img` 包含`img`,`a/img`, `a/b/img`...
	- `a/**/b`匹配“ `a/b`”、“ `a/x/b`”、“ `a/x/y/b`”等
	- `/**`匹配里面的所有内容：`abc/**`匹配目录“ `abc`”内的所有文件
6. 当忽略某个文件夹内所有文件，除了一个特殊文件时
	用`!`来维护该特殊文件
	注意点：需要先忽略该文件夹，后排除该特殊文件
```shell
# 有效顺序
properties/*
!properties/password.txt

# 无效顺序
!properties/password.txt
properties/*
```

检查某个文件是否匹配了忽略的规则`git check-ignore -v <file-path>`： `-v`详细信息打印出匹配的规则
```shell
$ git check-ignore -v docs/img/img/11.txt
# 匹配了.gitignore第四行规则docs/img
.gitignore:4:docs/img   docs/img/img/11.txt
```

## 0.4 别名
```shell
$ git config --global alias.co checkout
$ git config --global alias.ci commit
$ git config --global alias.br branch
```

也可以将一连串命令组合别名
`git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"`

也可以在该仓库配置文件中配置`alias`
```shell
$ cat .git/config
...
[branch "main"]
        remote = origin
        merge = refs/heads/main
[alias]
        lg = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
```


# 1. 创建版本库

```bash
# 进入一个准备用作版本库的文件夹内
Ahang@Ahang MINGW64 ~
$ cd Documents/Git/

# 初始化创建该库
Ahang@Ahang MINGW64 ~/Documents/Git
$ git init
Initialized empty Git repository in C:/Users/Ahang/Documents/Git/.git/
```

```bash
// 复制一个已创建的仓库:
 git clone ssh://user@domain.com/repo.git
```



# 2. 将文件添加到版本库中
![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/java/thread202205152127222.png)

- 先在版本库的文件夹里面添加文件如：README,可以写入些内容
- 再通过`git add README`添加至库中
- 最后通过`git commit -m "wrote a README file"` 后面的`" "`内容可以根据实际情况填写

```bash
# 可以通过dir, ls 查看当前目录下文件
Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ dir
README

Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ ls
README

# 添加到版本库
Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ git add README

# 提交到库内，并加以注解，方便自己和他人查看
Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ git commit -m "wrote a README file"
[master (root-commit) ae27363] wrote a README file
 1 file changed, 2 insertions(+)
 create mode 100644 README
```

**对于同时添加多个文件时**

```bash
# 创建3个文件
Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ touch a

Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ touch c

Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ touch b

# 报错示例：直接提交还没有添加时
Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ git commit -m "add 3 files"
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        a
        c
        b

nothing added to commit but untracked files present (use "git add" to track)


# 先添加，可以同时添加多个，空格隔开即可
Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ git add a b c

# 再提交，会将目前添加还没有提交的全部提交
Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ git commit -m "add 3 files"
[master 7602874] add 3 files
 3 files changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 a
 create mode 100644 b
 create mode 100644 c

```

# 3. 对仓库文件修改并保存

- 查看仓库状态：`git status`
- 查看修改后文件的区别：`git diff README`
- 然后对文件保存通过`git add README`和 `git commit -m <detail>`

```bash
# 查看状态
Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ git status
On branch master
nothing to commit, working tree clean

Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   README

no changes added to commit (use "git add" and/or "git commit -a")

# 查看区别
Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ git diff README
diff --git a/README b/README
index 75977d0..682c059 100644
--- a/README
+++ b/README
@@ -1,2 +1,4 @@
 Hello world!!!
-This is my frist example!
\ No newline at end of file
+This is my second examples!
+
+add some word
\ No newline at end of file

# 文件保存
Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ git add README

Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   README


Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ git commit -m "add some word"
[master adde561] add some word
 1 file changed, 3 insertions(+), 1 deletion(-)

Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ git status
On branch master
nothing to commit, working tree clean

```

# 4. 对修改过的文件还原到修改前和修改后

- 修改前后跳转：`git reset --hard commit_id` 
    - 跳到修改前一次：`commit_id` 改为`HEAD^`
    - 跳到修改前两次：`commit_id` 改为`HEAD^^`
    - 跳到修改前10次：`commit_id` 改为`HEAD~10`
    - 跳转到修改后某个点：`commit_id`改为该版本对应得hash值前几个唯一的字母如 `a3798`
- 查看提交的版本记录，对修改前后，查看每个版本对应得hash值：`git log`或`git log --pretty=oneline`
- 回到跳转前版本，此时查看log没有，需要查看历史记录: `git reflog`


- 原理：对多个版本的会记录下多个节点，通过`HEAD`指针指向当前版本文档，对文档的版本指定修改实质上也就是修改了`HEAD`指针的位置

---vision1---vision2---vision3  <--HEAD
```bash
# 查看提交的版本记录
Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ git log
commit a47393fb96dea41a21d9d674b6e12faff06b6edc (HEAD -> master)
Author: ahang1598 <hcie_zpc@163.com>
Date:   Fri Jul 17 21:44:58 2020 +0800

    add GPL

commit adde56105ebed13fe13dd735343dff32d6ddb8a9
Author: ahang1598 <hcie_zpc@163.com>
Date:   Fri Jul 17 21:34:55 2020 +0800

    add some word

~

# 查看提交的版本记录简易版
Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ git log --pretty=oneline
a47393fb96dea41a21d9d674b6e12faff06b6edc (HEAD -> master) add GPL
adde56105ebed13fe13dd735343dff32d6ddb8a9 add some word
ae27363a081d748911a88b6bbf86054f2373baf3 wrote a README file

# 还原上一个版本
Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ git reset --hard HEAD^
HEAD is now at adde561 add some word

# 查看内容
Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ cat README
Hello world!!!
This is my second examples!

add some word

# 此时再查看提交的版本记录，就没有了最后一次提交的了
Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ git log
commit adde56105ebed13fe13dd735343dff32d6ddb8a9 (HEAD -> master)
Author: ahang1598 <hcie_zpc@163.com>
Date:   Fri Jul 17 21:34:55 2020 +0800

    add some word

# 此时可以通过查看历史记录找到对应得commit_id
Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ git reflog
adde561 (HEAD -> master) HEAD@{0}: reset: moving to HEAD^
a47393f HEAD@{1}: reset: moving to a4739
adde561 (HEAD -> master) HEAD@{2}: reset: moving to HEAD^
a47393f HEAD@{3}: commit: add GPL
adde561 (HEAD -> master) HEAD@{4}: commit: add some word
cbbddf8 HEAD@{5}: commit: add 6 files
7602874 HEAD@{6}: commit: add 3 files
ae27363 HEAD@{7}: commit (initial): wrote a README file

# 跳转到指定的commit_id
Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ git reset --hard a4739
HEAD is now at a47393f add GPL
```

# 5. 撤销修改
- 命令`git checkout -- readme.txt`意思就是，把`readme.txt`文件在工作区的修改全部撤销，这里有两种情况：
    - 一种是`readme.txt`自修改后还==没有被放到暂存区==，现在，撤销修改就回到和版本库一模一样的状态；
    - 一种是`readme.txt`已经添加==到暂存区后，又作了修改==，现在，撤销修改就回到添加到暂存区后的状态

撤销修改☞回到最后一个状态
就是让这个文件回到最近一次`git commit`或`git add`时的状态

# 6. 撤销删除
1. 对于已经在仓库/工作区里面的文件删除：`git rm README`
2. 对于删除的文件恢复，先要从版本区恢复到暂存区:`git restore --staged README `
3. 再从暂存区恢复到工作区和仓库中:`git checkout -- README`
```bash
# 先删除
Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ git rm README
rm 'README'
# 查看状态，已删除
Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        deleted:    README
# 此时还没有提交，文件仍在在仓库里面，但是工作区没有了
Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ git log --pretty=oneline
492042c21840a7feb35984cd8a782c2cac7711f1 (HEAD -> master) remove a
43ede64fbf10c501c8969c831aa92e53ef4bae07 add boss
385781f7b45fab5e48f51addc205ac982b7542ae add a delete
714b14f8c577a4b4dedc0b650c01f322bc127080 delete chinese character
0f77a88b30fbfe27f50a2255cffb646abdfc82a4 add some
a47393fb96dea41a21d9d674b6e12faff06b6edc add GPL
adde56105ebed13fe13dd735343dff32d6ddb8a9 add some word
cbbddf8cf9394d2ed57523d87ca3a3546088bc5d add 6 files
7602874655e60aed97b4b7f0c3e5eec1a5f97a1b add 3 files
ae27363a081d748911a88b6bbf86054f2373baf3 wrote a README file
# 工作区已被删除
Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ ls
 a  'a[a-c]'   b   bb   cc   cc,dd   dd
# 直接从暂存区无法恢复
Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ git checkout -- README
error: pathspec 'README' did not match any file(s) known to git
# 从仓库里面恢复到暂存区
Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ git restore --staged README
# 再从暂存区恢复到工作区
Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ git restore README
Ahang@Ahang MINGW64 ~/Documents/Git (master)
$ ls
 a  'a[a-c]'   b   bb   cc   cc,dd   dd   README
```

**如果删除后提交了的话需要`git reset --hard commit_id`去回退操作**





# 7. 连接github操作

## 7.1 首次将本地库同步到远程库中

```bash
echo "# resume" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:ahang1598/resume.git
git push -u origin main
```


```bash
git remote add origin git@github.com:ahang1598/resume.git
git branch -M main
git push -u origin main
```

## 7.2 将本地库同步到远程库中

要关联一个远程库，使用命令`git remote add origin git@server-name:path/repo-name.git`；

关联后，使用命令`git push -u origin master`第一次推送`master`分支的所有内容；

此后，每次本地提交后，只要有必要，就可以使用命令`git push origin master`推送最新修改

## 7.2 将远程库克隆到本地

`$ git clone git@github.com:ahang1598/spider.git`

## 7.3 修改部分本地代码后同步远程库

思路是将整个`git`上的代码都`clone`下来，然后手动把自己的文件拉进指定文件夹，再上传上去

1. 创建一个文件夹"text"来右击点`git bash`

```bash
git init  

git clone [https].git
```

假设clone下来的文件名叫A

2. clone成功后我们打开A，将本地要上传的文件拖到指定文件夹内，然后git bash clone下来的文件A

```bash
git init   #默认下载下来的文件中包含了初始化的文件，无需初始化

git add .  # 注意add后面有一个空格和一个点

git commit -m  "提交的注释"

# git branch -M main 如果本地还是master

git push origin main
```

## 7.4 获取远程仓库并更新到本地仓库

`git pull`：相当于是从远程获取最新版本并`merge`到本地

```bash
$ git checkout branch2
$ git pull origin branch2
```

## 7.5在远程仓库和本地仓库都发生了变化后想要将本地修改提交到远程的流程：

1. `git add . `，然后`git commit`提交本地仓库
2. `git pull`拉取远程仓库，提示不能自动合并
3. **手动解决矛盾**，修改文件后再次`git add. `，`git commit`
4. `git push`成功

## 7.6 远程库已存在文件，首次将本地库同步到远程库中

```bash
git init
git remote add origin git@github.com:ahang1598/JavaStudy.git
git branch -M main
git pull origin main
git add .
git commit -m "add Code include kuang and springboot"
git push origin main
```



详解：

```bash
105@DESKTOP-V5RLSCF MINGW64 /d/BaiduNetdiskDownload/Java学习/kuang/Code
# 先初始化本地新的库
$ git init
Initialized empty Git repository in D:/BaiduNetdiskDownload/Java学习/kuang/Code/.git/

# 添加远端库
105@DESKTOP-V5RLSCF MINGW64 /d/BaiduNetdiskDownload/Java学习/kuang/Code (master)
$ git remote add origin git@github.com:ahang1598/JavaStudy.git

# 注意这里默认是master，而GitHub已经将主分支修改为main了，所以需要修改主分支
105@DESKTOP-V5RLSCF MINGW64 /d/BaiduNetdiskDownload/Java学习/kuang/Code (master)
$ git branch -M main

# 由于第一次使用远端已存在的库，所以为了内容同步，需要先pull拉去远程库所有内容
105@DESKTOP-V5RLSCF MINGW64 /d/BaiduNetdiskDownload/Java学习/kuang/Code (main)
$ git pull origin main
Enter passphrase for key '/c/Users/105/.ssh/id_rsa':
remote: Enumerating objects: 17, done.
remote: Counting objects: 100% (17/17), done.
remote: Compressing objects: 100% (12/12), done.
remote: Total 17 (delta 2), reused 6 (delta 1), pack-reused 0
Unpacking objects: 100% (17/17), 77.99 KiB | 108.00 KiB/s, done.
From github.com:ahang1598/JavaStudy
 * branch            main       -> FETCH_HEAD
 * [new branch]      main       -> origin/main

# 添加本地库中所有内容
105@DESKTOP-V5RLSCF MINGW64 /d/BaiduNetdiskDownload/Java学习/kuang/Code (main)
$ git add .

# 必须要提交
105@DESKTOP-V5RLSCF MINGW64 /d/BaiduNetdiskDownload/Java学习/kuang/Code (main)
$ git commit -m "add Code include kuang and springboot"
[main e583065] add Code include kuang and springboot
 6 files changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 Code/CODE-main.zip
 create mode 100644 Code/SpringBoot-master.zip

# 最后push上传
105@DESKTOP-V5RLSCF MINGW64 /d/BaiduNetdiskDownload/Java学习/kuang/Code (main)
$ git push origin main


```





# 8.分支

## 8.1 创建合并分支

命令总结：
1. 查看所有分支：`git branch`
2. 创建分支：`git branch <name>`
3. 切换分支：`git checkout <name>`或者`git switch <name>`
4. 创建+切换分支：`git checkout -b <name>`或者`git switch -c <name>`
5. 合并某分支到当前分支：`git merge <要被合并的子分支>`
6. 删除分支：`git branch -d <name>`


merge与rebase区别
`git merge <name>` 是要合并这个文件`<name>`到我这儿来
`git rebase <name>` 是要合并到他`<name>`那去

1. 创建并切换到该分支`git checkout -b b1`
2. 在该分支上操作：如创建新文件或修改文件`touch a`
3. 在分支上提交该操作: `git add a`, `git commit -m 'add a'`
4. 切换到主分支去合并该分支`git checkout main` ,  `git merge b1`


```bash
git clone [项目地址]		//克隆远程代码库到本地
cd [刚刚克隆的项目文件夹]		//进入本地仓库
git checkout -b dev		 //	创建分支dev（或者 git branch dev）
git branch -a		//查看所有分支


git checkout master		//切换到master
git  pull		//拉取master最新的内容
git checkout dev		//切换到分支dev
git merge master		//同步master的内容 (或者切换到master上　git rebase dev)
```



## 8.2 合并冲突
操作如下
1. 创建并切换到该分支`git checkout -b b1`
	1. 在该分支上操作：如修改文件`a`最后一行添加`add last line`
	2. 在分支上提交该操作: `git add a`, `git commit -m 'change a'`
2. 切换到主分支`git checkout main` 
	1. 在主分支上对同一文件`a`修改：修改第一行`haha`
	2. 在主分支上提交`git add a` ,  `git commit -m 'change a'`
3. 合并子分支`b1`到主分支`main`上：`git merge b1`
	1. 此时会报错：`Automatic merge failed; fix conflicts and then commit the result`
	2. 手动修改文件`a`为最后版本
4. 修改完后，再次在主分支上提交该文件：`git add a` ,  `git commit -m 'change a'`，此时完成本地仓库合并冲突解决。

## 8.3 合并分支
Git会用`Fast forward`模式，但这种模式下，删除分支后，会丢掉分支信息。
`--no-ff`方式：强制禁用`Fast forward`模式，Git就会在merge时生成一个新的commit，这样，从分支历史上就可以看出分支信息。

在合并分支时：
`git merge --no-ff -m "merge with no-ff" dev`
加`-m "info"` 是添加注释

默认`Fast forward`
```
          A---B---C feature
         /         main
D---E---F 
```

使用`--no-ff`
```
          A---B---C feature
         /         \
D---E---F-----------G main
```

此时可以显示出各个分支信息
![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/java/thread202205161638217.png)


## 8.4 提交远程分支

将本地分支`b1`内数据上传到远程仓库的分支`b1`中
进入到分支下：`git checkout b1`
然后
`git push origin b1`
或者
`git push --set-upstream origin b1`
或者
`git push origin main:b1`

## 8.5 拉取远程分支

默认拉去远程主分支`main`：`git pull`

对于远程子分支`b3`
先设置拉取追踪的位置为`b3`：`git branch --set-upstream-to=origin/b3 b3`
然后`git pull`

设置将拉取位置语法：
`git branch --set-upstream-to=origin/<branch>  <localBranch>`

## 8.6 rebase合并
当前`HEAD`在子分支`bugFix`上
此时通过`git rebase main`合并到主分支上，而且是单线模式，更加清晰简洁
![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/git_202205171607039.png)


现在 bugFix 分支上的工作在 main 的最顶端，同时我们也得到了一个更线性的提交序列。
注意，提交记录 C3 依然存在（树上那个半透明的节点），而 C3' 是我们 Rebase 到 main 分支上的 C3 的副本。
现在唯一的问题就是 main 还没有更新

现在切换到主分支上
然后通过`git rebase bugFix`，由于 `bugFix` 继承自 `main`，所以 Git 只是简单的把 `main` 分支的引用向前移动了一下而已。
![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/git_202205171604516.png)



以上操作来回切换过于繁琐，可以直接指定合并的两个分支
`git rebase <保留base分支> <要合并的子分支>`

![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/git_202205171552156.png)

当合并分支某点时，其**上面所有分支**都会按顺序被合并到**指定节点**下面
![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/git_202205171557055.png)




# 9. 分支移动
`HEAD`指向当前操作版本位置，当通过`checkout, switch`切换位置后，`HEAD`也跟着切换

`git checkout c1`将`HEAD`指向某一个版本，这个`c1`可以为某版本的hash值前几位数
将HEAD指向上一个版本
`git checkout main^`
`git checkout HEAD^`
![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/git_202205171610549.png)


相对引用：从某个确定的位置找之前的第几个位置
-   使用 1个`^` 向上移动 1 个提交记录：`HEAD^^`向上两个
-   使用 `~<num>` 向上移动多个提交记录，如 `HEAD~3`向上三个


多个父节点分支，通过`^n`来确定第`n`个父节点
![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/git_202205171618886.png)

链式移动`git checkout HEAD~^2~2 `
![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/git_202205171623602.png)




## 9.1 强制修改分支位置
`git branch -f <需要移动分支名称> <相对HEAD位置 | 分支HASH值>`
例如：
`git branch -f bugFix 73fjd`

`git branch -f main HEAD~3`
当前HEAD指向bugFix，移动main分支到HEAD往上三个分支位置

![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/java/thread202205170903834.png)


## 9.2 分支回退

本地分支回退
`git reset <HEAD相对位置 | 分支HASH值>`

`git reset HEAD^`
![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/java/thread202205170911190.png)


远程分支回退
新提交记录 `C2'` 引入了**更改** —— 这些更改刚好是用来撤销 `C2` 这个提交的。也就是说 `C2'` 的状态与 `C1` 是相同的。
revert 之后就可以把你的更改推送到远程仓库与别人分享啦。
`git revert HEAD`

![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/java/thread202205170920961.png)

## 9.3 整理分支
  `git cherry-pick <提交号>...`
将一些提交复制到当前所在的位置（`HEAD`）下面的话

`git cherry-pick c2 c4`
![](https://ahang.oss-cn-guangzhou.aliyuncs.com/img/git_202205170939852.png)
## 9.4 分支标签

给分支打标签，方便后面使用该分支，更有识别性
`git tag <标签别名> <分支>`
`git tag v1.0 c1dfse `

后面再使用该分支时，直接通过标签名`v1.0`操作
`git checkout v1.0`