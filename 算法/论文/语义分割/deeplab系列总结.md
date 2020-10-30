<<<<<<< HEAD
# Deeplab v1总结

## 1.Deeplab v1 解决什么问题？

如果用CNN解决语义分割，主要有两个问题：

* maxpooling和downsample(‘striding’)带来的降采样使得信息丢失
* CNN的空间不变性(图像旋转平移，不影响图像级别的分类结果，但是影响像素级别的分类结果)

## 2.Deeplab v1算法思路？

**2.1 如何解决section1的问题？**

* 对于cnn的降采样问题，Deeplab v1 引入了空洞卷积。那么空洞卷积为什么能够解决降采样问题？

  为了获得较大的感受野，在网络通常会引入maxpooling和striding，但是降低图像分辨率，使得信息丢失。能否减少引入maxpooling和striding也有较大的感受野呢？当然有！那就是空洞卷积，由于空洞卷积增加了卷积核大小，自然也增加了感受野。此外，对于同一个输入，采用不同的空洞率，也可以获得多尺度的上下文信息。不过deeplab v1中并没有采用不同空洞率。

* 对于CNN空间不变性问题，Deeplab v1引入全连接CRF来改善捕获细节的能力。关于CRF，deeplab v3已经不采用了，自己也没怎么细看。

**2.2 deeplab v1结构**

* 主要对原有的VGG网络进行了一些修改：
  * 将原来的全连接层（fc6、fc7、fc8）换成卷积层，为了让每个像素进行分类。
  * VGG网络中原来有5个maxpooling，将最后两个池化层的步长2改成步长1
  * 把最后三个卷积层的空洞率设置为2，并且将第一个全连接层(fc6，其实是修改后的卷积层)的空洞率设置为4。
  * 最后一层全连接层(fc8，其实是修改后的卷积层)的通道数从原来的1000修改为21。
  * 第一个全连接层fc6，通道数从4096变为1024，卷积核大小从7x7变为3x3，后续实验中发现此处的dilate rate为`12`时（LargeFOV），效果最好。
* 多尺度预测
  * 在输入图像和前四个maxpooling都接MLP(第一层：128个3x3的卷积，第二层：128个1x1的卷积)，他们的特征映射与主网络最后一层特征映射concatencated在一起。但是效果没有直接引入CRF效果好。

## 3 Deeplab v1 效果如何？

* 准确率高，当时在PASCAL的语义分割集上效果最好，达到71.6% mean IOU
* 速度很快，DCNN 8fps，CRF需要0.5秒
* 多尺度预测+CRF+LargeFOV效果最好

**参考:**

[1]https://www.jianshu.com/p/295dcc4008b4

[2]https://zhuanlan.zhihu.com/p/36052038



# Deeplab V2 总结

## 1.Deeplab V2 解决什么问题？

* 由于maxpooling和striding导致特征分辨率下降
* 多尺度物体存在
* CNN的空间不变性导致细节丢失

## 2.Deeplab V2 算法思路？

**2.1如何解决section1谈到的问题？**

* 对于特征分辨率下降问题，采用空洞卷积
* 对于存在多尺度物体问题，引入了空洞空间金字塔池化(ASPP)
* 对于CNN空间不变性问题，Deeplab v2也是引入全连接CRF来改善捕获细节的能力

**2.2 ASPP?**

在输入特征中采用不同的空洞率进行卷积(如图4所示)，能够得到不同大小的感受野，捕获不同的上下文信息，从而更好地处理多尺度物体。不同空洞率提取到的特征后续在不同的分支处理并融合到最后的结果中（如图7所示）。

![](https://gitee.com/weifagan/MyPic/raw/master/img/aspp.PNG)

![](https://gitee.com/weifagan/MyPic/raw/master/img/aspp1.PNG)

## 3.Deeplab V2 效果如何？

* 在PASCAL VOC 2012 测试集上，deeplab-CRF(resnet101) 效果达到最优，79.7%mIOU



# Deeplab V3 总结

## 1.Deeplab V3 解决什么问题?

* 由于maxpooling和striding导致特征分辨率下降
* 多尺度物体存在

## 2. Deeplab V3 算法思路？

**2.1如何解决section1谈到的问题？**

* 对于特征分辨率下降问题，采用空洞卷积
* 对于多尺度物体问题，串联或者并行(v2的ASPP)不同空洞率的空洞卷积来得到更多的上下文信息

**2.2 更深的空洞卷积**

1.串联结构：

striding能够更加容易捕获远距离的信息，但是连续的striding不利于语义分割，所以使用空洞卷积，减少striding，这样子也能捕获远距离信息，图3中，Block4、Block5、Block6是拷贝，使用空洞卷积最后的output_stride=16，不使用空洞卷积的话output_stride=256

![](https://gitee.com/weifagan/MyPic/raw/master/img/deeper_atrous.PNG)

2.multi-grid

Multi-grid = [r1,r2,r3]，Block4-Block6中有各自的unit rates，Block中的rates=unit rates*Multi-grid。

**2.3 ASPP**

作者发现，随着空洞率越来越大，有效权重越少，当空洞率足够大时，只有卷积核中间的权重才有效，退化成1x1的卷积核，不能得到全局的上下文信息。为了解决这个问题，在最后一层使用全局平局池化，把image-level的特征“喂入”256个1x1的卷积核中(和bn)然后进行双线性采样。改进后的ASPP包括了(a)1x1卷积，rates=(6,12,18)的空洞卷积和(b)image-level的特征。

![](https://gitee.com/weifagan/MyPic/raw/master/img/para_aspp.PNG)

还是不懂全局平均池化层是怎么用起来？

## 3.Deeplab V3 效果如何？

* PASCAL VOC 2012 test set上达到86.9%mean IOU
* Cityscapes test set上达到81.3% mean IOU

# Deeplab V3+ 总结

## 1.Deeplab V3+解决什么问题

* 主要解决降采样后的恢复物体边缘信息难以恢复的问题

## 2. Deeplab V3+ 算法思路？

**2.1如何解决section1谈到的问题？**

* 使用encoder-decoder结构逐渐恢复空间信息来捕获sharper物体边缘。

**2.2 Deeplab V3+结构**

在deeplab V3上应用并行的ASPP捕获多尺度context，然后增加简单但有效的decoder模型来恢复物体边缘(如图1，图2所示)。

![](https://gitee.com/weifagan/MyPic/raw/master/img/deeplab%20v3%20.PNG)

![](https://gitee.com/weifagan/MyPic/raw/master/img/deeplab%20v3%201.PNG)

1.Encoder

* 输入尺寸与输出尺寸比（output stride = 16），最后一个stage的膨胀率rate为2

* Atrous Spatial Pyramid Pooling module（ASPP）有四个不同的rate，额外一个全局平均池化

2.Decoder

* 将encoder的结果进行4倍上采样，然后与resnet下采样前的conv2特征concat，再进行3x3卷积，然后在4倍上采样

* 融合低层次信息前，先进行1x1的卷积，目的是降通道（例如有512个通道，而encoder结果只有256个通道） 
=======
# Deeplab v1总结

## 1.Deeplab v1 解决什么问题？

如果用CNN解决语义分割，主要有两个问题：

* maxpooling和downsample(‘striding’)带来的降采样使得信息丢失
* CNN的空间不变性(图像旋转平移，不影响图像级别的分类结果，说明CNN对边缘细节信息不敏感，难以捕获细节信息)

## 2.Deeplab v1算法思路？

**2.1 如何解决section1的问题？**

* 对于cnn的降采样问题，Deeplab v1 引入了空洞卷积。那么空洞卷积为什么能够解决降采样问题？

  为了获得较大的感受野，在网络通常会引入maxpooling和striding，但是降低图像分辨率，使得信息丢失。能否减少引入maxpooling和striding也有较大的感受野呢？当然有！那就是空洞卷积，由于空洞卷积增加了卷积核大小(3x3的标准卷积核，若空洞率为2，空洞卷积核大小为k+(k-1)(r-1)=3+(3-1)(2-1)=5)，自然也增加了感受野。此外，对于同一个输入，采用不同的空洞率，也可以获得多尺度的上下文信息。不过deeplab v1中并没有采用不同空洞率。

* 对于CNN空间不变性问题，Deeplab v1引入全连接CRF来改善捕获细节的能力。关于CRF，deeplab v3已经不采用了，自己也没怎么细看。

**2.2 deeplab v1结构**

![](https://gitee.com/weifagan/MyPic/raw/master/img/vgg16.PNG)

* 主要对原有的VGG16网络进行了一些修改：
  * 将原来的全连接层（fc6、fc7、fc8）换成卷积层，为了让每个像素进行分类。
  * VGG网络中原来有5个maxpooling，将最后两个池化层的步长2改成步长1
  * 把最后三个卷积层的空洞率设置为2，并且将第一个全连接层(fc6，其实是修改后的卷积层)的空洞率设置为4。
  * 最后一层全连接层(fc8，其实是修改后的卷积层)的通道数从原来的1000修改为21。
  * 第一个全连接层fc6，通道数从4096变为1024，卷积核大小从7x7变为3x3，后续实验中发现此处的dilate rate为`12`时（LargeFOV），效果最好。
* 多尺度预测
  * 在输入图像和前四个maxpooling都接MLP(第一层：128个3x3的卷积，第二层：128个1x1的卷积)，他们的特征映射与主网络最后一层特征映射concatencated在一起。但是效果没有直接引入CRF效果好。

## 3 Deeplab v1 效果如何？

* 准确率高，当时在PASCAL的语义分割集上效果最好，达到71.6% mean IOU
* 速度很快，DCNN 8fps，CRF需要0.5秒
* 多尺度预测+CRF+LargeFOV效果最好

**参考:**

[1]https://www.jianshu.com/p/295dcc4008b4

[2]https://zhuanlan.zhihu.com/p/36052038



# Deeplab V2 总结

## 1.Deeplab V2 解决什么问题？

* 由于maxpooling和striding导致特征分辨率下降
* 多尺度物体存在
* CNN的空间不变性导致细节丢失

## 2.Deeplab V2 算法思路？

**2.1如何解决section1谈到的问题？**

* 对于特征分辨率下降问题，采用空洞卷积
* 对于存在多尺度物体问题，引入了空洞空间金字塔池化(ASPP)
* 对于CNN空间不变性问题，Deeplab v2也是引入全连接CRF来改善捕获细节的能力

**2.2 ASPP?**

在输入特征中采用不同的空洞率进行卷积(如图4所示)，能够得到不同大小的感受野，捕获不同的上下文信息，从而更好地处理多尺度物体。不同空洞率提取到的特征后续在不同的分支处理并融合到最后的结果中（如图7所示）。

![](https://gitee.com/weifagan/MyPic/raw/master/img/aspp.PNG)

![](https://gitee.com/weifagan/MyPic/raw/master/img/aspp1.PNG)

## 3.Deeplab V2 效果如何？

* 在PASCAL VOC 2012 测试集上，deeplab-CRF(resnet101) 效果达到最优，79.7%mIOU



# Deeplab V3 总结

## 1.Deeplab V3 解决什么问题?

* 由于maxpooling和striding导致特征分辨率下降
* 多尺度物体存在

## 2. Deeplab V3 算法思路？

**2.1如何解决section1谈到的问题？**

* 对于特征分辨率下降问题，采用空洞卷积
* 对于多尺度物体问题，串联或者并行(v2的ASPP)不同空洞率的空洞卷积来得到更多的上下文信息

**2.2 更深的空洞卷积**

1.串联结构：

striding能够更加容易捕获远距离的信息，但是连续的striding不利于语义分割，所以使用空洞卷积，减少striding，这样子也能捕获远距离信息，图3中，Block5、Block6、Block7是Block4拷贝，使用空洞卷积最后的output_stride=16，不使用空洞卷积的话output_stride=256

![](https://gitee.com/weifagan/MyPic/raw/master/img/deeper_atrous.PNG)

2.multi-grid

Multi-grid = [r1,r2,r3]，Block4-Block6中有各自的unit rates，Block中的atrous rates=unit rates*Multi-grid。

**2.3 ASPP**

作者发现，随着空洞率越来越大，有效权重越少，当空洞率足够大时，只有卷积核中间的权重才有效，退化成1x1的卷积核，不能得到全局的上下文信息。为了解决这个问题，在最后一层使用全局平均池化，把image-level的特征“喂入”256个1x1的卷积核中(和bn)然后进行双线性采样。改进后的ASPP包括了(a)1x1卷积，rates=(6,12,18)的空洞卷积和(b)image-level的特征。(为什么得不到全局的上下文信息就去使用全局平均池化)

![](https://gitee.com/weifagan/MyPic/raw/master/img/para_aspp.PNG)

## 3.Deeplab V3 效果如何？

* PASCAL VOC 2012 test set上达到86.9%mean IOU
* Cityscapes test set上达到81.3% mean IOU

# Deeplab V3+ 总结

## 1.Deeplab V3+解决什么问题

* 主要解决降采样后的物体边缘信息难以恢复的问题

## 2. Deeplab V3+ 算法思路？

**2.1如何解决section1谈到的问题？**

* 使用encoder-decoder结构逐渐恢复空间信息来捕获物体边缘。

**2.2 Deeplab V3+结构**

在deeplab V3上应用并行的ASPP捕获多尺度context，然后增加简单但有效的decoder模型来恢复物体边缘(如图1，图2所示)。

![](https://gitee.com/weifagan/MyPic/raw/master/img/deeplab%20v3%20.PNG)

![](https://gitee.com/weifagan/MyPic/raw/master/img/deeplab%20v3%201.PNG)

1.Encoder

* 输入尺寸与输出尺寸比（output stride = 16），最后一个stage的空洞率为2

* Atrous Spatial Pyramid Pooling module（ASPP）有四个不同的rate，额外一个全局平均池化

2.Decoder

* 将encoder的结果进行4倍上采样，然后与resnet下采样前的conv2特征concat，再进行3x3卷积，然后在4倍上采样
* 融合低层次信息前，先进行1x1的卷积，目的是降通道（例如有512个通道，而encoder结果只有256个通道） 

## Deeplab V3+ 效果如何？

* PASCAL VOC 2012 and Cityscapes 的测试集上  89.0% and 82.1%，不需要任何后处理。
>>>>>>> 8e840698c751ccf7dfb2b896f6ce326a603646e7
