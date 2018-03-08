---
title: 置信区间
date: 2017-09-27 22:59:08
mathjax: true
tags: [数学, 《高级工程数学》, 读书]
---

《Advanced Engineering Mathematics》整本书对很多基础概念讲解的很细，但对于Confidence Interval这个概念确有点不直观。书里这么定义：

Confidence intervals for an unknown parameter $\\mu$ of some distribution (e.g., $\\theta = \\mu$) are intervals $\\theta_1 \\le \\theta \\le \\theta_2$ that contain $\\theta$, not with certainty but with a high probability $\\gamma$, which we can choose (95% and 99% are popular). Such an interval is calculated from a sample. $\\gamma = 95%$ means probability $1 - \\gamma = 5% = \\dfrac{1}{20}$ of being wrong---one of about 20 such intervals will not contain $\\theta$. Instead of writing $ \\theta_1 \\le \\theta \\le \\theta_2 $, we denote this more distinctly by writing

$$ CONF\_\\gamma\\{\\theta_1 \\le \\theta \\le \\theta_2\\}.$$

紧接着就开始standard trick等一堆东西了。为了更直观的理解，我们整理一下这个概念是在面对什么问题时提出来的。
# 问题

* 假设：
    我们通过实验，得到了一组样本值，并且我们假设这组样本是由符合某个概率分布模型的随机变量产生的。

* 问题：
    根据这组样本，如何确定这个概率分布模型的参数？

# 具化
把这个问题具化一下，假设我们得到了一组样本数据，绘制一下样本数据的直方图，如下图所示。另外我们假设这个概率分布是正态分布，我们要确定一下未知参数$\\mu$。
![](/images/confidence_interval_p1.png)

通过图中可以看到，即便给定一组样本，$\\mu$实际上可能是任意值，只是不同的$\\mu$产生样本数据的概率大小不一样，例如图中$\\mu_1$产生这组样本(直方图展示)的概率会比$\\mu_2$高。

我们可以使用任意函数$f(sample)$对$\\mu$进行估计，计算样本均值是一种方式，所有样本值相乘也可以是一种方式，只要这个$f(sample)$的值在$\\mu$的取值范围内都OK。那为什么所有书上都是用样本均值来进行估计呢？应该是因为当$\\mu$取值为样本均值时，产生该样本序列的概率更高，不过这个也得是从统计上来讲，对于一个具体的序列，也有可能其他形式估计的均值产生这个序列的概率更高。

# 置信区间和置信水平
在给定样本的情况下$\\mu$的值也成了一个随机变量，而且是一个连续随机变量。连续随机变量取某个固定值的概率是0，值落到某个区间内的概率估计更重要。置信区间(confidence interval)就是一个$\\mu$的值区间，$\\mu$的概率密度函数在这个区间上的积分就是置信水平(confidence level)。
