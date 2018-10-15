---
layout: post
title: tensorflow machine learning cookbook的小知识点
description: 只是一些零碎的知识点，主要是一些不同选择的对比
category: 技术
tags: [tensorflow]
---
# 知识点
### [ch6]神经网络activation funciton
+ sigmoid
	+ 同类型函数: 
		+ tanh: 可通过sigmoid线性变化得到。0均值，可解决收敛慢的问题
	+ 优点: 压缩数据使数据幅度不会出现问题，适合向前传播
	+ 缺点: 收敛慢(因为是非0均值), 幂运算耗时, 容易梯度消失(当导数接近0)
+ relu
	+ 同类型函数: 
		+ leakly relu: max(0.01x, x), 用来解决神经坏死
		+ elu, softplus
	+ 优点: 收敛快，不会梯度消失，运算快，适合向后传播
	+ 缺点: 非零均值，神经坏死(永远在负数部分为0)，不压缩导致的数据幅度不断扩大

### [ch6]TensorFlow中CNN的两种padding方式“SAME”和“VALID”
+ [参考地址](https://blog.csdn.net/wuzqChom/article/details/74785643)
+ 简单说明:当valid发现余下的窗口无法卷积时直接舍弃多余列，而same方式会以0填充新的列
+ 以height为例，新的height的shape的公式:
	+ valid: new_height = ceil((W-F+1)/S)
	+ valid: new_height = ceil(W/S)

# 一些质疑
### [ch6]井字棋
最后的井字棋用demo训练出来的结果很烂, 而且既然已经枚举出了所有情况，又何必用机器学习呢？总之不是一个好例子

### [ch7]Working with Bag of Words
learn.preprocessing.VocabularyProcessor应该先用fit，再transform，否则会导致sentence_size和min_word_freq两个参数失效

### [ch7]Implementing TF-IDF
TF-IDF简单的说是在本文中出现的多且在所有文章出现的少的词的权重高，但我认为在这个预测是否spam的例子中，使用TF-IDF是没有意义的。
