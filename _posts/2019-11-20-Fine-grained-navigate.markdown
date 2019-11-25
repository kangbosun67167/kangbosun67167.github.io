---
layout: article
title:  "细粒度识别论文阅读《Learning to Navigate for Fine-grained Classification》"
date:   2019-11-20 15:57:48 +0000
tags: Fine-grained
mathjax: true
mathjax_autoNumber: true
---


## 任务


细粒度识别，弱监督信息下识别子类分类问题，例如分辨不同鸟、车、飞机。  


## 出发点


使用图片中有信息量的区域进行辅助分类会使得分类更加准确。  


## 难点

1、如何定义区域的信息量？  
2、如何使用这些区域联合分类？ 


## 方法

作者创建了一个 navigate and teacher network，其中navigate网络提取有信息的区域，teacher网络对区域进行信息量判定，从而二者相互促进，使得navigate的提取区域的能力得到强化。  
最后联合原图和多个提取图进行特征提取后分类。

### Navigate网络

<!-- ![navigate](assert/navigate.png) -->

<figure>
<a><img src="{{site.url}}/assert/navigate.png"></a>
</figure>

该网络的输入为原图的CNN特征和N个最大信息目标区域，输出为N个缩放后的区域图。  

预先设置M个anchor，使用 RPN 的方式生成预设的anchor，由teacher获得信息量进行提取信息最高的N个anchor，

从而提取出这些anchor对应的区域。

### Teacher网络

<figure>
<a><img src="{{site.url}}/assert/train.png"></a>
</figure>

该网络针对原图的CNN特征进行评分，评出对应anchor的分数（由于RPN网络的提取框是有规律的，所以Teacher可以从CNN特征去拟合这些分数），
使用非最大值抑制（NMS）的方法挑选出最高的几张anchor，这些anchor的位置信息会送入到Navigate网络中进行截取相应的区域。

### Scrutinizer网络

该网络将原图和其他区域图的特征拼接起来进行最后的分类。

<figure>
<a><img src="{{site.url}}/assert/inference.png"></a>
</figure>


### loss设计

#### 单图的loss

各个图（原图和区域图）的单独分类loss，使用同一个特征提取器和分类器（resnet和fc）

#### 决策loss

Scrutinizer网络的决策loss。

#### 候选框建议loss

*重点部分：*

作者将Teacher网络评价出的N个评分当作信息量的衡量，为一个sorted的信息量list；
将Navigate网络提取出的区域建议图的决策loss作为置信度，即为真实情况下这些图多大程度上能够贴合标签，同样为一个置信度list。

将置信度list作为信息量list的标签，损失函数使用排序学习中的rank_loss。


区域图信息量（由Teacher Net评价得到） $ I = {I_1,I_2,...,I_m} $, 区域图置信度（由Resnet+fc网络分类得到）$ C = {C_1,C_2,...,C_m} $,
候选框建议loss（导航loss）被定义为：

$$ Loss = \sum_{(i,s):C_i<C_s} f(I_s - I_i)$$

$$ f(x) = max(1-x,0) $$

该loss惩罚置信度（$C$）和信息量（$I$）不成顺序的情况。

例如，区域候选图的置信度为：${0.4,0.3,0.2,0.1}$ , 信息量评价为${0.3,0.4,0.2,0.1}$，
则有：$ Loss = (1.1 + 0.9 + 0.8) + (0.8 + 0.7) + (0.9)  = 5.2 $

又例如，区域候选图的置信度为：${0.4,0.3,0.2,0.1}$ , 信息量评价为${0.1,0.2,0.3,0.4}$，
则有：$ Loss = (1.1 + 1.2 + 1.3) + (1.1 + 1.2) + (1.1)  = 7.0 $

可以看到的是，排序越不一致，分类的惩罚越大。
（同样注意到，惩罚的增长不成线性增长（排序不一致情况情况严重时，惩罚力度不够）。）

## 结果

<!-- ![result](assert/result.png) -->

<figure>
<a><img src="{{site.url}}/assert/result.png"></a>
</figure>



[原文链接《Learning to Navigate for Fine-grained Classification》](http://openaccess.thecvf.com/content_ECCV_2018/papers/Ze_Yang_Learning_to_Navigate_ECCV_2018_paper.pdf)
