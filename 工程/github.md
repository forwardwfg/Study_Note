## 上传本地文件

1.命令创建一个空的Git仓库或重新初始化一个现有仓库

<<<<<<< HEAD

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

错误原因：

在 git pull origin master 添加 --allow-unrelated-histories，即

```
git pull origin master --allow-unrelated-histories
```

**Automatic merge failed; fix conflicts and then commit the result.**

clone了远程的代码，然后做了修改了代码，

**git push 出现Everything up-to-date**:

错误原因：可能是没有git add和git commit，重新add 和commit即可。

git push -u origin master



