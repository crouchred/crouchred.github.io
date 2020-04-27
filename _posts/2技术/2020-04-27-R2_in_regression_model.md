---
layout: post
title: 回归模型中的R2
description: 回归模型中的R2， 从R2的取值范围开始说起
category: 技术
tags: [数据挖掘]
---

## 1. 摘要
本文讨论一下R2这个指标。R2的别名有: R方,RSquare,决定系数。可以通俗地理解为使用均值作为误差基准，看预测误差是否大于或者小于均值基准误差。
公式为: 1 - RSS/TSS  即 1减去均方根误差和方差的比值

结论是: R2的高低不能用来评价模型的好坏，但我认为在区分了训练测试集的情况下，**用R2选择回归预测模型至少是有意义的**，尽管有时用其他指标更好。

**No statistic is dangerous if you understand what it means**, 详见5.3了解哪些情况会影响R2


## 2. 缘起
前些天帮分析师做了一个回归模型，结果R2大概在0.3左右，分析师对此很质疑说R2应该到0.8-0.9才好吧？emm其实回归模型做的真的很少，用R2是因为R2是最常见的指标，大概知道用来判断模型是否好于基准模型。
查了资料之后最大的困惑在于**有些文章说R2是0-1的，另一些说是可以小于0的**，我自己测用excel使用算不出负数，但用sklearn很容易算出负数，一度让我以为R2在不同场景代表的计算是不一样(其实一样)，于是写下这个文章。

## 3. R2取值范围
在python的sklearn包中，明确说到sklearn.metrics.r2_score是可以小于0的，如果小于0代表模型还不如你所有y取均值。
另一些文章中说，R2的取值为0-1，但大部分文章都没有说明的是，这个取值指的是在**线性回归**的条件下，因为只要线性回归的代码过程正确(或者使用excel等软件)，最差情况就是y取均值，即R2等于0

## 4. R2是否适用于非线性回归
我认为是可以的，有一些[文章](https://blog.minitab.com/blog/adventures-in-statistics-2/why-is-there-no-r-squared-for-nonlinear-regression)有这样的主张: 
1. 非线性回归的R2会小于0，所以不适合。但我认为小于0也可以表示模型的效果不如均值模型，这并非没有意义。
2. 线性回归的R2计算基于这个公式: SSR + SSE = SST,  但是非线性回归这个等式不成立，我其实没看懂这一条..但如果这一条是上一条小于0的原因，那么意义不大
3. 非线性回归如果过拟合了，会使R2很高。这个需要注意用区分训练集和测试集

## 5. 为什么不能单纯的看R2的高低
1. R2不能看出数据偏差问题
2. R2低，只影响y预测值的精确度，如果某些字段的p值低，不影响字段显著性的判断
3. **除了模型的好坏之外，有很多其他影响R2的因素(即R2低但模型好，和R2高但模型差的情况)**
	+ 用户心理等，本来就比较难预测的情况, R2低；例如预测股价，即使R2低，也是好模型(可以赚钱)
	+ 物理过程，R2高，例如通过身高预测体重
	+ 如果y本来就是x的一个变化，比如通过x是摄氏度，y是华氏度
	+ 如果x和y同时和时间/地理显著相关,都随着时间(经纬度)增加而增加，R2偏高
	+ 过拟合(仅非线性)
	+ y是categorical , R2会低
	+ 数据量大，R2越低
	+ x越多, R2越高，这一点可以使用R2-adjust
	+ R2可能因为某些异常点，降低

## 6. 其他可以使用的指标
+  F-Tests 
+  Bayes' Factors
+  Information Criteria(AIC, BIC)
+  out-of-sample predictive accuracy
+  standard error, mean absolute error, mean absolute percentage error, mean absolute scaled error，**可以参考**[sklearn的Regression metrics](https://scikit-learn.org/stable/modules/model_evaluation.html#regression-metrics)

## 7. 参考
+ [Is R2 Useful or Dangerous](https://stats.stackexchange.com/questions/13314/is-r2-useful-or-dangerous)
+ [Why Is There No R-Squared for Nonlinear Regression](https://blog.minitab.com/blog/adventures-in-statistics-2/why-is-there-no-r-squared-for-nonlinear-regression)
+ [Five Reasons Why Your R-squared Can Be Too High](https://blog.minitab.com/blog/adventures-in-statistics-2/five-reasons-why-your-r-squared-can-be-too-high)
+ [sklearn r2score]http://scikit-learn.org/stable/modules/generated/sklearn.metrics.r2_score.html
+ [sklearn Regression metrics ](https://scikit-learn.org/stable/modules/model_evaluation.html#regression-metrics)
+ [8 Tips for Interpreting R-Squared](https://www.displayr.com/8-tips-for-interpreting-r-squared/)


