---
layout: post
title: tensorflow machine learning cookbook的小知识点
description: 书的一些零碎的知识点，和一些错误
category: 其他
tags: [tensorflow]
---

*最后修改时间: 2019-01-10*

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
	+ same: new_height = ceil(W/S)

### [ch7]word2vec
+ 很好的参考文章
	+ [详解skip-gram1](https://www.leiphone.com/news/201706/PamWKpfRFEI42McI.html)
	+ [详解skip-gram2](https://www.leiphone.com/news/201706/eV8j3Nu8SMqGBnQB.html)
	+ [详解skip-gram3](https://www.leiphone.com/news/201706/QprrvzsrZCl4S2lw.html)
	+ [腾讯云old zhang](https://cloud.tencent.com/developer/article/1005771)
+ 一些创新(只有第三点书中用到了)
	+ 常见的单词组合可作为单个词处理
	+ 高频词单词抽样，限制样本出现次数
	+ 对输出层进行负采样，提升计算效率
+ skip-gram和CBOW的区别(书的结论和网上找的不一致)
	+ skip-gram:一个学生(输入)多个老师(输出)，结果更准确但更耗时
	+ CBOW: 多个输入一个输出
+ word2vec训练的好坏？
	+ 词聚类: 可以采用 kmeans 聚类，看聚类簇的分布
	+ 词cos 相关性
	+ Analogy对比:a:b 与 c:d的cos距离 (man-king woman-queen )
	+ 使用tnse，pca等降维可视化展示(tensorboard或matplotlib)

### [ch8] Retrain Existing CNNs models
+ [tensorflow hub Image Retrain](https://www.tensorflow.org/hub/tutorials/image_retraining)
+ 获取一个分类模型有三种方式(后两种成为迁移学习):
    + train from scratch: 从头训练
    + Fine-tune model: 所以参数都可变化
    + retrain model: 只训练模型最后一层softmax层，其余层参数全部固化; 实际使用中, 计算每张图片的bottleneck(倒数第二层)并保存

### [ch10] Taking TF to production tips
+ tf.reset_default_graph() 
+ 使用tf.train.Saver() 保存checkpoint
+ save之前确保每个variable都有name
+ tf.app.flags设置命令行参数
+ tf.set_verbosity()设置内置的log级别

# 一些质疑
### [ch6]井字棋
最后的井字棋用demo训练出来的结果很烂, 而且既然已经枚举出了所有情况，又何必用机器学习呢？总之不是一个好例子

### [ch7]Working with Bag of Words
learn.preprocessing.VocabularyProcessor应该先用fit，再transform，否则会导致sentence_size和min_word_freq两个参数失效

### [ch7]Implementing TF-IDF
TF-IDF简单的说是在本文中出现的多且在所有文章出现的少的词的权重高，但我认为在这个预测是否spam的例子中，使用TF-IDF是没有意义的。

### [ch7]Working with skip-gram
+ 结果随机性太高，和随机出来的初始值关系很大
+ 结果可理解为算近义词，但是肉眼看来和算命似的，你说他像也行不像也行..

### [ch8]Implementing an Adcanced CNN(cifar-10)
+ model_learning_rate的衰减，因为generation_num没有放入tf.train.GradientDescentOptimizer().minimize()，或者使用feed_dict，导致未生效

