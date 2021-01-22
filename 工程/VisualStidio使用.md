# 1.项目属性配置

项目配置注意**Debug**模式还是**Release**模式

**链接器->输入->附加依赖项**：添加链接时候的第三方lib库(cpp打包成的.lib文件)。

在添加opencv依赖项，注意Debug模式和Release模式。如果是Debug模式，添加的是\bin\*d.lib的库(有后缀d)，如果release模式，则需要的是不带d的lib，但是我们能不能就直接\bin\*.lib这样子添加呢？如果这样子的话，可能带d的也会包含进来。

**C/C++->附加包含目录**：添加include头文件的路径

例如，配置opencv时候就要添加：

```
C:\OpenCV2.4.6\include
C:\OpenCV2.4.6\include\opencv
C:\OpenCV2.4.6\include\opencv2
```

