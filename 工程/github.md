## 上传本地文件

1.命令创建一个空的Git仓库或重新初始化一个现有仓库

git init

2.添加文件

git add 文件名

ps git add. 可以添加所有

3.添加注释

git commit -m "注释"

4.建立远程仓库

git remote add origin https://github.com/WeifaGan/Study_Note.git

ps 在默认情况下，origin指向的就是你本地的代码库托管在Github上的版本。origin就是一个名字，它是在你clone一个托管在Github上代码库时，git为你默认创建的指向这个远程代码库的标签。

5.提交到origin的主分支

git push -u origin master



## Error

**git fatal: 远程 origin 已经存在**

此时只需要将远程配置删除，重新添加即可:git remote rm origin