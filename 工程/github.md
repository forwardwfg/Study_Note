## 简介

Git是一种分布式版本管理系统。

什么是版本管理？例如对于策划某个活动，老板可能让你修改很多遍，如果你只是在同一份文档上修改的话，最后老板说：“嗯！还是用回第一次那份策划把”，这是你可能会崩溃了，因为这意味这你又要把那份文档改回第一个版本。但是如果你的每次修改的文档都做一个备份，plan1.doc，plan2.doc等等，你就可以很方便地第一个版本的文档扔到老板面前了。其实后面对每个修改了的文档都做备份，形成不同的版本，这就是版本管理。

什么是分布式？“分布式"相对于"集中式"而分布式而言。把所有数据保存在服务器上节点上，所有客户节点都从服务节点上获取数据的版本控制称为集中式版本管理。因为你的数据不仅仅保存在Git的服务器上，还会保存在自己本地，所以Git称为分布式版本管理系统。



## 上传本地文件

1.命令创建一个空的Git仓库或重新初始化一个现有仓库

```
git init
```

2.添加文件

```
git add 文件名
```
3.添加注释

```
git commit -m "注释"
```

4.建立远程仓库

```
git remote add origin https://github.com/用户名/xxxx.git
```
ps 在默认情况下，origin指向的就是你本地的代码库托管在Github上的版本。origin就是一个名字，它是在你clone一个托管在Github上代码库时，git为你默认创建的指向这个远程代码库的标签。

5.提交到origin的主分支
首次提交到master

```
git push -u origin master
```

以后提交可以不用-u了

```
git push origin master
```



## 从远程仓库获取最新版本的代码

获取最新版本有两种方法，拉取和获取pull和fetch

* git pull 从远程拉取最新版本到本地，自动合并merge， git pull origin master

* git  fetch  从远程获取最新版本 到本地  不会自动合并 merge   git fetch  origin master  



## Error

**git push origin master,error: failed to push some refs to**

```
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

错误原因：在推送前没有进行仓库和远程服务器同步(对github上的代码进行了在线修改但是没有对本地仓库同步)

例如：

原始代码try.py为：

```Python
1 print("try")
2 print("try")
```

在某个时间把github上代码修改为：

```python 
1 print("try")
2 print("try")
3 print("try11")
```

后面又在自己本地把原始代码修改为：

```python
1 print("try")
2 print("try")
3 print("try222")
```

这样子做导致了本地仓库和远程仓库的commit记录不一致。

解决方法：

先把远程仓库同步到本地:

```
git pull origin master
```

但是因为第3行存在竞争行，在自动合并代码出现了冲突，这时候可以通过手动合并。

```
Auto-merging try.py
CONFLICT (content): Merge conflict in try.py
Automatic merge failed; fix conflicts and then commit the result.
```

在本地查看的代码如下图：

![](https://gitee.com/weifagan/MyPic/raw/master/img/git_merge.PNG)

这时候如果只想保留某一处的代码，可以点击采用传入的更改或者采用当前更改，如果想合并两者，则保留双方更改。本例保留双方更改，保留结果如下图。

![](https://gitee.com/weifagan/MyPic/raw/master/img/git_merge1.PNG)

还需要把合并的结果提交

```
git add try.py
git commit -m “merge conflict”
```

此时本地和远程仓库的commit记录才保持一致，然后就可以愉快地把本地合并的代码push到远程仓库了。

```
git push origin master
```

**git status 后，出现**：

```On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)```
```

**git pull origin master，fatal: refusing to merge unrelated histories**

错误原因：git认为本地和远程仓库没有是两个独立的仓库，他们之间没有联系。在github网页下载下来的仓库会遇到这种问题，但是通过命令git clone则不会。

在 git pull origin master 添加 --allow-unrelated-histories，即

```
git pull origin master --allow-unrelated-histories
```

合并后，还需要提交结果才能使得本地和远程仓库同步。

```
git add .
git push origin master
```

**git push后，Everything up-to-date**:

错误原因：可能是没有git add和git commit，重新add 和commit即可。

## Git的三个分区

Git的三个分区分别是工作目录，Index索引区，版本库。

* 工作目录：操作系统上的文件，所有代码开发编辑都在这上面完成。
* 索引区(缓存区)：暂时存放数据的区域
* 版本库：存放commit后的数据，push时候，就会把这个区域的数据push到远程仓库了。

<img src="https://gitee.com/weifagan/MyPic/raw/master/img/.git5.png" style="zoom:80%;" />

刚开始三个分区都一样，没区别。当在工作区修改数据时候，工作区内容发生改变，但暂存区和版本库内容不变。当执行git add之后，修改被同步到缓存区，这时候工作区和缓存区保持内容一样了。执行git commit后，三个区域内容保持一致。push后，数据推送到远程仓库。

三个分区对比命令：

```
git diff	            工作区 vs 暂存区
git diff head	        工作区 vs 版本库
git diff –cached        暂存区 vs 版本库
```

## Git存储方式

当在一个新目录下或者已有目录内执行进行git init，就会创建一个.git目录，几乎所有存储和操作的内容都位于该目录下。.git目录如下图

<img src="https://gitee.com/weifagan/MyPic/raw/master/img/.git1.png" style="zoom:67%;" />

* config文件包含了项目的配置选项

* objects目录存储所有数据内容

* HEAD文件指向当前分支

* index文件保存了暂存区信息

* refs目录存储 指向数据的提交对象的指针
* hooks 目录保存了客户端或服务端钩子脚本
* info 目录保存了一份不希望在 .gitignore 文件中管理的忽略模式 (ignored patterns) 的全局可执行文件

其中，HEAD 及 index 文件，objects 及 refs 目录是 Git 的核心部分。

objects目录存储了所有的数据，该目录下的每一份文件都是Git为每份存储数据生成一个文件，如下图。

![](https://gitee.com/weifagan/MyPic/raw/master/img/.git2.png)

文件夹下存放的文件如下图所示，它是一个二进制文件。

![](https://gitee.com/weifagan/MyPic/raw/master/img/.git4.png)

objects存储目录中的二进制文件对象类型有三种，blob，tree和commit。

* blob，用于存储数据内容，不包括文件名等其他信息。
* tree，用于存储目录结构。
* commit，用于存储提交的信息。