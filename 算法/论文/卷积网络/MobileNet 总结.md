# MobileNet V1-V3总结

## 1. MoblieNet V1 总结

### 1.1 MobileNet v1 解决什么问题？

在机器人，自动驾驶和增强现实等应用中，都要在资源有限的平台下进行实时执行，MobileNet V1主要解决网络计算量大的问题，以平衡精度和速度。

###  1.2 MobileNet V1 如何解决计算量大的问题？

MobileNet V1 的关键是使用了深度可分离卷积网络，与常规的卷积网络相比，它能够大大减少计算量。

**常规卷积网络**：

下面卷积操作的图片均来自

[知乎的回答]: https://zhuanlan.zhihu.com/p/92134485



常规的卷积网络如下图所示，得到的特征图个数为卷积核个数。

![](https://gitee.com/weifagan/MyPic/raw/master/img/cnn1.PNG)

此时，图中的卷积核参数量为：3x3x3x4 = 108。

设输入大小为5x5x3，那么卷积的计算量为：3x3x3x4x(5-3+1)x(5-3+1) = 972

**深度可分离卷积网络**：

第一步：逐通道卷积。输入的每个通道对应一个卷积核，卷积核个数为输入通道输入，得到的特征图个数为输入通道个数。

![](https://gitee.com/weifagan/MyPic/raw/master/img/dw1.PNG)

此时，图中的卷积核参数量为：3x3x1x3 = 27。

卷积的计算量为：3x3x1x3x(5-3+1)x(5-3+1) = 243

第二步：逐点卷积。为了扩充特征图通道数量，引入逐点卷积。

![](https://gitee.com/weifagan/MyPic/raw/master/img/dw2.PNG)

此时，卷积核参数量为：1x1x3x4 = 12。

卷积计算量为：1x1x3x4x(5-1+1)x(5-1+1)  = 108

**计算结果对比**

所以深度可分离卷积参数量为：12+27 = 39，约为常规CNN的1/3。

当只有一层卷积的情况下，深度可分离卷积网络的计算量约为常规CNN的(243+108)/972 = 0.36

计算公式如下图所示(仅计算了卷积过程中的乘法计算量)。

![](https://gitee.com/weifagan/MyPic/raw/master/img/dw_c.PNG)

其中，$D_K$为卷积核大小(边长)，$M$为输入通道数，$N$为卷积核个数，$D_F$为特征图大小(边长)。

**MobileNet V1 网络结构**：

![](https://gitee.com/weifagan/MyPic/raw/master/img/mobile%20v1.PNG)

在该网络结构中，逐通道卷积和逐点卷积都接BN和RULU，如下图所示。

![](https://gitee.com/weifagan/MyPic/raw/master/img/cnn%20and%20moblie%20v1.PNG)

此外，MobileNet V1还添加了**宽度因子和分辨率因子**分别来控制通道数和输入分辨率。

**总结**：

为了减少计算量，MobileNet V1使用了深度分离卷积网络来替代常规CNN，此外也添加了宽度因子和分辨率因子来控制网络结构大小。

### 1.3 效果如何?

精度跟常规CNN接近(略有下降)，但是计算量和参数都大大下降。

![](https://gitee.com/weifagan/MyPic/raw/master/img/mobile_v1_effect.PNG)

参考：

[MobileNet]: https://arxiv.org/abs/1704.04861
[知乎回答]: https://zhuanlan.zhihu.com/p/92134485

## 2 MobileNet V2 总结

### 2.1 MobileNet V2 主要解决什么问题？

* Relu和数据坍缩

## 2.2 MobileNet V2算法思路？

* 对于Relu数据坍缩问题，在使用深度可卷积分离之前先增加维度再进行relu操作，之后再降维。
* 引入linear bottleneck，在bottleneck结构中删除最后一层的relu，保持表达能力。

**Inverted Residuals** ：

![](https://gitee.com/weifagan/MyPic/raw/master/img/relu.PNG)

将2维空间的数据通过随机矩阵映射到高维然后进行Rulu操作，然后再映射到2维空间，实验结果表明，当n=2，3时候，Rulu操作会丢失较大的信息。维度越高(第三个通道的数量)，信息丢失越少。

所以用深度可卷积分离之前先增加维度再进行relu操作，之后再使用1x1的卷积进行降维，过程如下图所示。默认的通道增加为原来的6倍。residual block先是通过1x1升道然后通过relu6，在3x3dw卷积然后relu，再是1x1降道。两头小，中间大，像逆沙漏一样。

<img src="https://gitee.com/weifagan/MyPic/raw/master/img/bottleneck_mobilev2.jpg" style="zoom: 33%;" />

<img src="https://gitee.com/weifagan/MyPic/raw/master/img/Inverted_residual_block.PNG" style="zoom: 50%;" />



**linear bottleneck**：

在bottleneck结构中删除最后一层的relu，如上图，保持表达能力。ps，Xception已经实验证明了Depthwise卷积后再加ReLU效果会变差。

**relu6**：

relu6是限制了relu的最大输出值为6，因为在移动设备端，float16的低精度已经有很好的分辨率，但是如果不对relu限制，当relu很大的时候，精度会丢失。

## 1.3 效果如何?

* 相比于Mobilenetv1，v2 精度更高，参数少，计算量更少， 速度更快。

