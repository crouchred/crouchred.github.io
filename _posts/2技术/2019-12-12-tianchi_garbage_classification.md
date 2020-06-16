---
layout: post
title: 天池比赛：垃圾图片分类
description: 天池比赛：垃圾图片分类
category: 技术
tags: [deeplearning]
---

*最后修改时间: 2019-12-12*

## 背景
+ [赛题与赛制](https://tianchi.aliyun.com/competition/entrance/231743/information)
+ 初赛:我们队伍(小野兽队)的成绩只有16.33分，主要知识点Flink和Analytics Zoo我都不熟，此篇文章不讨论。
+ 复赛:上传代码+模型，使用天池线上的6000张图片在3小时内进行训练，并预测600张图片。我队的成绩为79.97，第三名。
+ 决赛: 优胜奖(第四名), 初赛分数占了20%，我队比较吃亏。但别的队伍很强也是真的，这篇文章也主要想记录一下其他队伍的创新点。

## 资料
+ [天池链接](https://tianchi.aliyun.com/competition/entrance/231743/introduction)
+ [代码资料](https://github.com/spyfree/Tianchi)
+ [分享](https://tianchi.aliyun.com/notebook-ai/detail?postId=83701)
+ [其他队伍分享见论坛](https://tianchi.aliyun.com/competition/entrance/231743/forum)

## 我队关键点
#### 根据用户视角，制作`categorical_crossentropy_weight`
+ 简述: 对自己下载的图片做了人工筛选和匹配的工作，并把错误分为了5大类。根据错误分类设置weight和loss function结合，给不同的错误赋予不同的权重。
+ 效果: 提高了模型精度的同时(约1个点)而且收敛速度会快很多。
+ 例子: 把拖把识别为耳机的错误,比把充电器识别成充电线的错误更严重。
+ 错误类型
  1. 人肉很难识别(权重:0.1): 充电器=》充电线，纸箱=》快递盒
  2. 形状颜色相近(权重:0.3): 奶粉=》白糖盐，毛毯=》抹布。
  3. 颜色相近(权重:0.6): 纸尿裤=》杯子
  4. 形状相近(权重:0.6): 筷子=》水彩笔，杏仁=》葡萄干
  5. 明显不同(权重:1)：拖把=》耳机
+ 代码参考: [Is there a way in Keras to apply different weights to a cost function?](https://github.com/keras-team/keras/issues/2115)

#### 线下两环境+线上环境
1. 线下调参环境
	+ 区分train(1000张图片)和validation(300张图片)数据集，根据validation accuracy来选择模型和参数
	+ 运行时间控制在30分钟(按照线上3小时的估算) 	
	+ 基本上所有keras支持的模型都试了，决赛的队伍中我们好像是唯一一个模型和参数测得那么详细的。efficientNet因为线上python2不支持就没上，有其他队伍折腾出来了。

2. 线下预训练环境
	+ 全量(1w张)图片均作为训练集，不关注训练集的accuracy，保存预训练之后的模型
	+ 运行时间数个小时甚至半天，参数基本和其他环境一致，可能会使用: 1.更小的学习速率，2.更多epochs，3.更多的图片变形 4.更多的layer设置为可训练
	+ **二次迁移**。所谓的二次迁移指的是，我们一般使用keras就是做了一次迁移学习，而由于这次比赛线上图片不可下载，我们线下和线上环境之间的迁移学习可以称为二次迁移。这种二次迁移大概能提升0.5-1.5分(蚊子肉也是肉系列..)

3. 线上天池环境
	+ 使用线下调参环境的参数+线下预训练环境的模型

## 其他队伍的关键点
1. 湖人总冠军: 使用mixup进行图片增强，并把one-hot的label软化变成soft label，以此改进focal-loss。
2. SpacePro: 3EfficientNets+attention+SVD模型。把三个模型的最后几层拆掉之后，接了新的attention层+svd层，高级啊。但三个融合模型如果不都用efficientNet或许更好。
3. SkyPeace: 1. 多模型置信度+投票，应该就是加权平均的意思。2. 线上保存多个checkpoint，根据val_accuracy选择最好的模型，cnn并没有那么容易过拟合，所以有没有用我持保留意见，但思路不错。
