---
layout: post
title: "分析预测模型在机器学习中的应用之线性回归"
description: "回归能做任何事情"
tags: [线性回归, machine learning , regression, algorithms]
---
回归能做什么呢？《机器学习实战》的作者认为：
>回归能做任何事情。

回归模型大致分为2个阶段，训练阶段和预测阶段， 训练阶段需要对历史确定结果的数据进行反馈训练，使得模型的参数适应当前的输入与预期输出， 预测阶段在完善模型的基础上对新的数据进行模拟输出。这两个阶段对于一个模型都至关重要。

回归模型广泛地应用于机器学习，深度学习，AI，人工智能等领域，在这里我们讲对它的应用进行
深入浅出的学习与探讨。

针对不同的数据(数值型和标称型)，我们一般有两种预测倾向，数值预测，类别预测，这两种预测有某种潜在的关系，可以说，分类是高层次及符合条件下的数量预测，它建立在数量预测的基础上(这个后面的章节会cover到)

这篇文章的研究环境，基于[`jupiter notebook`](https://github.com/jupyter/notebook)， 依赖的python包有[`pydataset`](),[ `scilearn`](http://scikit-learn.org/),
 [ `pandas`](http://pandas.pydata.org/), [`seaborn`](https://github.com/mwaskom/seaborn)以及必要的`线性代数`，`统计学`及`高等数学`的知识。
[TOC]

###线性回归模型
这是最简单的回归模型，也是最general的机器学习算法，也包含了机器学习与回归模型最朴素和使用的思考方法，  它基于一个朴素的假设：输入和输出之间存在简单的线性关系.
$$
y = h(\mathbf{x}) = \theta_0 + \theta_1*x_1 + \theta_2*x_1 +  ...\theta_n*x_n = \mathbf{\theta}^T\mathbf{x}
$$
其中y 是输出的数值,$x_i$ 是输入的数值,x_0 =1,  n是输入的特征数量
$$\mathbf{x}=[x_0,x_1...x_n]^T$$ 
$$ \mathbf{\theta}=[\theta_0,\theta_1...\theta_n]$$


线性回归算法需要确定$\theta_j$使得平方误差的和最小, 构造损失函数:
$$
J(\theta)={1 \over 2}\sum_{i=1}^{m}(h_\theta(x^{(i)})-y^{(i)})^2
$$
这里简单地给出选用误差函数为平方和的概率证明
首先我们提出了一个符合常理的假设：误差是服从均值为0，的高斯分布 的
$$y^{(i)}=\theta^Tx^{(i)}+\epsilon^{(i)}$$
即
$$\epsilon^{(i)} = y^{(i)}-\theta^Tx^{(i)}  \sim  N(0, \sigma^2)$$
最大似然估计
$$
max { \prod_{i=1}^{m}p(y^{(i)}|x^{(i)}; \theta) }  = max{ \prod_{i=1}^{m}} {1 \over {\sqrt{2\pi}\sigma}} e^{- { { ({y^{(i)} - \theta^T x^{(i)} })^2 } \over 2\sigma^2}}
$$
$$
max\,{ log( \prod_{i=1}^{m}p(y^{(i)}|x^{(i)}; \theta) ) } =  max\,{mlog({1 \over {\sqrt{2\pi}\sigma}}) - {1 \over 2\sigma^2}\sum_{i=1}^{m}(h_\theta(x^{(i)})-y^{(i)})^2}
$$
$$
\Rightarrow  min\,{1 \over 2}\sum_{i=1}^{m}(h_\theta(x^{(i)})-y^{(i)})^2 = min\,{J(\theta)}
$$
####最小二乘法
对$\mathbf{\theta}$求导得到:
$$J'(\theta)=X^T(Y-X\theta)=0$$
$$\Rightarrow \theta = (X^TX)^{-1}X^TY$$
####梯度下降法
选定learning rate  $\alpha$,对 $\mathbf{\theta}$ 的每个分量求偏导去更新$\theta_j$:
$$tmp_j=\theta_j-\alpha*{\partial \over \partial\theta_j}J(\theta)$$ 
其中
$$\mathbf{\theta} = [tmp_0, tmp_1...tmp_n]^T $$
$$
{\partial \over \partial\theta_j}J(\theta)={\partial \over \partial\theta_j}{1 \over 2m}\sum_{i=1}^{m}(\theta_0x_0^{(i)}+...\theta_nx_n^{(i)} - y^{(i)})^2
$$
$$
\Rightarrow {\partial \over \partial\theta_j}J(\theta)={\partial \over \partial\theta_j}{1 \over m}\sum_{i=1}^{m}((h_\theta(x^{(i)}) - y^{(i)})x^{(i)}
$$
对于n=1时，$y=mx+b$, python梯度下降代码如下：
```python
def stepGradient(b_current, m_current, points, learningRate):
    b_gradient = 0
    m_gradient = 0
    N = float(len(points))
    for i in range(0, len(points)):
        b_gradient += -(2/N) * (points[i].y - ((m_current*points[i].x) + b_current))
        m_gradient += -(2/N) * points[i].x * (points[i].y - ((m_current * points[i].x) + b_current))
    new_b = b_current - (learningRate * b_gradient)
    new_m = m_current - (learningRate * m_gradient)
    return [new_b, new_m]
```
重复执行`stepGradient`后得到$J(\theta)$与执行次数的关系:


下面是sklearn的线性模型的应用:
```python
from sklearn import datasets
from sklearn.cross_validation import cross_val_predict
from sklearn import linear_model
import matplotlib.pyplot as plt

lr = linear_model.LinearRegression()
boston = datasets.load_boston()
y = boston.target

# cross_val_predict returns an array of the same size as `y` where each entry
# is a prediction obtained by cross validated:
predicted = cross_val_predict(lr, boston.data, y, cv=10)

fig, ax = plt.subplots()
ax.scatter(y, predicted)
ax.plot([y.min(), y.max()], [y.min(), y.max()], 'k--', lw=4)
ax.set_xlabel('Measured')
ax.set_ylabel('Predicted')
plt.show()
```
####回归的正则化与特征选择
为了避免过拟合的问题，使模型更general一些，正则化的理念被提了出来，正则化被用来减小单个$\theta_j$的影响,基本原理是在全局损失函数中加入惩罚函数
#####Elastic net方法
L1(曼哈顿距离)正则化,也叫Ridge regression， 它将$sum_{i=0}^{n}|\theta_i|$ 作为惩罚函数, 它保留一个$\theta_j$而使其他的应变量趋向于0
L2(欧几里得距离)正则化,也叫Lasso regression，它将$sum_{i=0}^{n}|\theta_i|^2$作为惩罚函数, 它使得所有的$\theta_j$依存变量拥有相同的影响

将两者结合后亦即elastic net惩罚，我们的损失函数就成了:
$$
J(\theta)={1 \over 2}\sum_{i=1}^{m}(h_\theta(x^{(i)})-y^{(i)})^2 + \lambda({\alpha}sum_{j=0}^{n}|\theta_j|  +  (1-\alpha)sum_{j=1}^{n}|\theta_j|^2)
$$
可以看出,$\alpha$是我们对于L1和L2正则化的偏向，$\lambda$代表正则化的程度,然后使用前面提到的正规方程或者梯度下降来求解, 求得正则化后的$\theta_j$

令$\lambda_1=\lambda\alpha$, $\lambda_2=\lambda(1-\alpha)$, 对于给定的数据(X,Y),  $\lambda$,$\alpha$ 定义一个数据集$(X^\*, Y^\*)$.

$$
X_{(n+p)*p}=(1+\lambda)^{1/2}{\begin{pmatrix} X\\ \sqrt{(\lambda_2I) }\\\end{pmatrix}}
$$

$$
{Y_{n+p}}^*=\begin{pmatrix} Y \\ 0 \\ \end{pmatrix}
$$
令$\gamma=\lambda_1/\sqrt{(1+\lambda_2)}, \beta^*=\sqrt{(1+\lambda_2)}\beta$, 则：
$$
||Y-X\theta||^2+\lambda_2\,sum_{j=1}^{p}\theta_j^2+\lambda_1\,sum_{j=1}^{p}|\theta_j|
=||Y^*-X^*\theta^*||^2+\gamma\,sum_{j=1}^{p}|\theta_j^*|
$$
$$
\Rightarrow \hat \theta_{(Elastic Net)}={1 \over \sqrt{(1+\lambda_2)}}{\hat \theta^*} = {1 \over \sqrt{(1+\lambda_2)}} arg min{||Y^*-X^*\theta^*||^2 + \gamma\,sum_{j=1}^{p}{|\theta_j^*|}}
$$
这便转化为了普通的[套索问题](https://en.wikipedia.org/wiki/Lasso_(statistics))

参考：
[似然函数](http://baike.baidu.com/link?url=akO9Z0DPatdoeDUDYgia2IvbG6MVCJUiqAbHuZmXH16n7sHjjPGngKg1ozrSh3T0kTX_wEgNmiqEwNwD98_kUa#5)
[最小二乘法](http://baike.baidu.com/link?url=BOZMFNcZdLep-ywZnl34ClJ_TWFjiVqYRgKP_Es3UMhldA0H0bB8sDuhvFbGyKQ7ISnQDA2DiJzxadC_AHcJ6_)
[矩阵，向量求导](http://www.cnblogs.com/huashiyiqike/p/3568922.html)
[极大似然估计](http://baike.baidu.com/link?url=Og7CzUF8qUOEmj3avrXXGbB5Tr9kRGiX3u-Z5USycG8DaIofrYILswcePnYfZGY42tm-5yUSuE6OfDUEhFo7Fq)
[广义线性模型基于Elastic+Net的变量选择方法研究](http://www.doc88.com/p-0953739664306.html)
