---
layout: post
title: cross validation推荐
description: 说几种不同的cross validation(cv)
category: 技术
tags: [cv]
---

# 1. 缘起

我比较懒，以前在做模型的时候一般都选用holdout cross validation，最近在看k-fold cv的时候偶然接触到nested cv, 至此一发不可收拾, 实在是想搞明白这个方法，说是使"the true error of the estimate is almost unbiased relative to the test set when nested cross-validation is used"。

看下来的结论是，这句话是没错的，但是nested cv用起来性价比并不高，我自己一般不会使用。

此外也发现了k-fold作为的含义并不复杂大家都能理解，但实际的工作流程却各有说法，我也总结了自己认为性价比最高的一个流程(3.3.3.1 流程二)


# 2. cv的目的
+ 模型与超参数的选择
+ 如果不用cv, 选择模型时测试数据的信息会被泄漏，造成高方差



# 3. cv的不同方法

### 3.1 holdout 留出法
即分为train/validation/test三个数据集，用train,validation两个数据集确定超参数/选择模型，并在test数据集上评价模型
优点是简单，缺点有很多:
+  受数据分割影响大
+ 如果分为三份，可利用的数据量就少了，浪费了数据
+ 容易过拟合

**评价: 不能算真正的cv, 不推荐**

### 3.2 LOO 留一法 or LPO 留P法
+ LOO : 对于整个数据集而言,每次选取一个样本作为验证集,其余样本作为训练集
+ LPO : 对于整个数据集而言,每次选取P个样本作为验证集,其余样本作为训练集(P个样本是排列组合的，所以消耗的计算量惊人)

**评价: 超高的性能开销，不推荐**

### 3.3 k-fold
#### 3.3.1 简述
使用 k - 1 个折叠中的数据训练，最后一个剩下的折叠会用于测试

#### 3.3.2 分层(StratifiedKFold)/非分层
分层可以确保训练集，测试集中各类别样本的比例与原始数据集中相同。

#### 3.3.3 实际工作流程
网上的各个人写的流程都不太一样，关键点在于建模的目的不同：
1. 模型评估
2. 输出训练数据的predict结果
3. 输出模型以预测新数据
我觉得不管是工作还是打比赛，都应该以输出模型为主，以此为基础，我总结了两种流程

###### 3.3.3.1 流程一
1. 分为n份数据，其中1份作为test数据
2. 可选的超参数组合/模型有m个
3. 用cv对每个超参数组合进行模型训练并评估, 一共m x (n-1)次循环，得到平均分数
	+ 评估方式可采用accuracy, f1, auc等，看情况而定
	+ 每次计算都是用n-1份数据训练，1份数据测试
	+ 在流程一中认为，如果不单独拿出一份test数据，即使用了cv，评价结果也会偏高，即高方差
4. 选出最优的超参数组合
5. 使用该超参数对全量train数据进行训练，输出模型
6. 使用该模型对test数据进行预测，并输出最终评价结果
7. 使用该超参数和全量数据再次训练模型，输出最终模型
	+ 这一步大多文章都不会提到，我心里也没底，但是总觉得在数据量不大的情况下，test数据不加入模型就被浪费了

###### 3.3.3.1 流程二(更推荐)
我更倾向于认为，既然已经用了cv，所谓的高方差可以忽略不计，不需要额外再分出一份test数据，大大简化了流程。但**这似乎有偷懒的嫌疑，如果有人因此提出质疑，似乎也没有问题。**

1. 可选的超参数组合/模型有m个，train切为n份
2. 用cv对每个超参数组合进行模型训练并评估, 一共m x n次循环，得到平均分数，作为最终评价结果
3. 使用全量数据和超参数，再次训练模型，并输出

#### 3.3.4 tips: i.i.d假设

即Independent and Identically Distributed，在实践中很少成立。

- 如果数据是时间相关的，使用 time-series aware cross-validation scheme
- 如果数据是具有 group structure (群体结构), 则使用 group-wise cross-validation 

### 3.4 nested cv

#### 3.4.1 简述
使用内层cv(inner loop)进行模型/超参数选择，使用外层cv(outer loop)进行模型评估。可以认为在流程一的外面再套了一层cv

优点是"其挑选的模型在训练集和测试集上的误差估计几乎没有出入"(The true error of the estimate is almost unbiased relative to the test set when nested cross-validation is used)

#### 3.4.2 流程
1. 分为n份数据，其中1份作为test数据
2. 可选的超参数组合/模型有m个
3. 对m个超参数组合进行模型训练并评估，一共m x (n-1)次循环，得到平均分数
4. 选出最优的超参数组合
5. 使用该超参数对全量train数据进行训练，输出模型
	+ 到这里与3.3.3.1的流程一致
6. 使用该模型，对test数据进行评估打分
7. 使用外层cv, 换一份数据作为test数据，重复3-6的过程，共 m x (n-1) x n次循环
8. 对外层cv的10个分数进行平均，就是模型最后的分数

#### 3.4.3 nested cv的问题
最大的问题很明显！他没有输出模型！
参考文档里有一篇stackexchange的回答对此说的很清楚:
+ 模型选择是通过内层循环做的
+ 外层循环只做模型评价(输出分数)，不输出模型，你也不应该选择表现最好的那个模型
+ 此外外层模型可以得出这些结论:
	+ 预测的稳定性: 看平均分是否差不多
	+ 超参数的稳定性: 看超参数是否差不多
	+ 看内外层模型的结果的稳定性，来判断是否过拟合

这些分析是没错，但是问题就是，如果我不能选择表现最好的那个模型，就算模型很稳定，我也不能随机选一个吧。我进行了 m x (n-1) x n次循环计算，但是却只能得到一个准确的模型评价，但是无法输出一个模型，实在是让人很无语。

# 4. kfold流程二的py代码
1. 可选的超参数组合/模型有m个，train切为n份
2. 用cv对每个超参数组合进行模型训练并评估, 一共m x n次循环，得到平均分数，作为最终评价结果
3. 使用全量数据和超参数，再次训练模型，并输出

```python
from sklearn import datasets
from sklearn import svm
from sklearn.pipeline import make_pipeline
from sklearn.model_selection import GridSearchCV
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline

def load_data():
    iris = datasets.load_iris()
    X = iris.data
    y = iris.target
    return X,y

def print_best_score(gsearch):
    print("Best score: %0.3f" % gsearch.best_score_)
    print("Best params: %s"%gsearch.best_params_)

def make_pipe():
    pipe = Pipeline([("std",StandardScaler()),
                     ("clf",svm.SVC(random_state=1))])
    return pipe

def get_gsearch(pipe, X, y):
    param_range = [0.0001,0.001,0.01,0.1,1.0,10.0,100.0,1000.0]
    param_grid = [{"clf__C":param_range,
                   "clf__kernel":['linear']},
                  {"clf__C":param_range,
                   "clf__gamma":param_range,
                   "clf__kernel":['rbf']}]
    gsearch = GridSearchCV(pipe , param_grid = param_grid, scoring='accuracy', cv=5 )
    gsearch.fit(X, y)
    print_best_score(gsearch)
    return gsearch

def get_full_model(gsearch, X, y):
    clf__C = gsearch.best_params_['clf__C']
    clf__kernel = gsearch.best_params_['clf__kernel']
    if gsearch.best_params_.get('clf__gamma') is not None:
        clf__gamma = gsearch.best_params_.get('clf__gamma')
    else:
        clf__gamma = None

    pipe = Pipeline([("std",StandardScaler()),
                     ("clf",svm.SVC(kernel=clf__kernel, C=clf__C, gamma=clf__gamma))])
    pipe.fit(X, y)
    return pipe

def main():
    X, y = load_data()
    pipe = make_pipe()
    gsearch = get_gsearch(pipe, X, y)    # 通过cv搜索最佳的超参数
    pipe = get_full_model(gsearch, X, y) # 使用全量数据对模型重新训练
    print('New data Result: ', pipe.predict([[6.5, 3.1, 5.8, 2.1]]))

if __name__=="__main__":
    main()
```



# 5. 参考资料
+ [Sklearn中的CV与KFold详解(cnblog)](https://blog.csdn.net/fontthrone/article/details/79219663)
+ [sklearn中的cv官方文档](https://scikit-learn.org/stable/modules/cross_validation.html)
+ [维基百科cv](https://en.wikipedia.org/wiki/Cross-validation_(statistics)#Nested_cross-validation)
+ [cross validation - 机器学习中的交叉验证法探究(简书)](https://www.jianshu.com/p/cdf6df99b44b)
+ [Nested cross validation for model selection(stackexchange)](https://stats.stackexchange.com/questions/65128/nested-cross-validation-for-model-selection)
+ [通过网格搜索和嵌套交叉验证寻找机器学习模型的最优参数(cnblog)](https://blog.csdn.net/sinat_29957455/article/details/79690613)
+ [nested cv的流程(youtube)](https://www.youtube.com/watch?v=LpOsxBeggM0)
