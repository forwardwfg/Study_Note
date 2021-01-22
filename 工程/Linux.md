# 1.Linux常用工具

## 1.1 gcc g++

### 1.1.1 编译

编译过程分成四个阶段：预处理(.i文件)->编译(.s)->汇编(.o)->链接(可执行文件)

1. 一步到位的编译指令

```
gcc test.c -o test
```

2. 预处理

```
gcc -E test.c -o test.i
```

-E 的选项，可以让编译器在预处理后停止，并输出结果

3. 编译

```
gcc -S test.i -o test.s
```

-S的选项，可以让编译器在生成汇编代码后停止，并输出结果

4. 汇编
5. 

```
gcc -c  test.s -o test.o
```

5. 链接

```
gcc test.o -o test
```

### 1.1.2 常用参数说明

-c：只激活预处理，编译和汇编，也就源程序做成obj文件

-S：只激活预处理和编译，也就是把源程序做成汇编代码

-E：只激活预处理，生成.i文件

-o：制定目标名称



## 1.2 objdump 反汇编

objdump可以把目标文件(甚至任意一个二进制文件)进行反汇编

目标文件包含了默认包含三个section，.text存放代码，.data存放需要初始化的数据，.bss存放不需要初始化的数据，如下图：

![](https://gitee.com/weifagan/MyPic/raw/master/img/section.png)



1. 对代码段(.text)进行解析

```

```

其中：

```
Disassembly of section .text:
```

就是就是对.text字段的反编译结果

objdump -D -b binary -m i386 a.bin



