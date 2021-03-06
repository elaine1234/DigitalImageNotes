#  用于消除摩尔纹噪声的中值高斯滤波器框架

>   A median-Gaussian filtering framework for Moiré pattern noise removal from X-ray microscopy image
>
>  https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3858302/

## Part 1 概述 Abstract

摩尔纹噪声给扫描STXM(Transmission X-ray Microscopy)图带来了很大的影响。既然无法在图像获取阶段避免摩尔纹， 本文介绍一种从STXM图像中过滤出摩尔纹的处理方法。这种方法包括用本地中值滤波器将图片转换一个傅里叶振幅域半自动的光谱峰值探测器。用高斯陷波滤波器评估光谱噪声峰值。

**Keywords: **STXM, Moiré pattern, Image processing, Median filter, Gaussian notch filter

## Part 2 介绍 Introduction

摩尔纹噪声通常是周期噪声。这种周期噪声一般是在图像的傅里叶域中处理的，因为纯周期噪声的光谱是正弦波适当位置的共轭频率一对共轭脉冲。因此，修正光谱振幅能有效消除周期噪声。考虑两种频率滤波器： Wiener滤波器和陷波滤波器。但是Wiener需要精确的噪声模型，这些模型很难获取，并且计算十分复杂。所以我们用陷波滤波器来做摩尔纹消除。

陷波滤波器的原理就是通过去除孤立的噪声或扭曲的频率排除图像“坏”的部分。因为纯周期噪声的带宽很窄，量级很大，所以在傅里叶域会有一个很陡的峰值。这个通过简单的二值化就能定位到。然后通过陷波滤波器就能修正它。但是在大多数场景下的摩尔纹不是纯周期的，因此摩尔纹是准周期的，图像不止一个正弦模式。另外，有些高频峰值肉眼都无法判别。这样会导致两个问题。第一，摩尔纹噪声并没有很陡的峰值。因此，用二值化进行峰值检测变得不可行。第二，因为光谱峰值还会影响周围的像素，只纠正光谱峰值是不够的。

基于上述情况，我们介绍一种半自动光谱噪声探测器和一个光谱高斯陷波滤波器。光谱噪声探测器是用均值滤波器做的脉冲噪声探测器。

## Part 3 中值高斯陷波滤波算法 Median-Gaussian notch filtering algorithm

### 3.1 探测摩尔纹噪声的光谱峰值

$$
H(k, l)=\sum_{n=0}^{N_1-1} \sum_{m=0}^{M_1-1}h(k,l)e^{-i2\pi ((km/M_1)+( ln/N_1))}
$$

其中$h(m,n)$是$M_1 \cdot N_1$大小图片的像素，$H(k,l)$是在点$(k,l)$的振幅。得到的DFT图像中亮白色点表示峰值。实现方式是将当前的信号值与本地的中值对比。映射到频率域，会分成两步：

1. 定义一个低频区域

   因为低频系数决定了信号能量，我们希望所有光谱系数能保存下来。所以简单的定义一个在0频系数$H(0,0)$周围的$M\cdot N$的矩形。

2. 本地化一个光谱峰值
   $$
   \frac{H_{kl}}{MED_{K \cdot L}H(_{kl})} \ge T
   $$
   ​

### 3.2 高斯陷波滤波器

