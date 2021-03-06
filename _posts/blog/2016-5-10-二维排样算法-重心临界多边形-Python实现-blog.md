---
layout:     post
title:      二维排样算法-重心临界多边形-Python实现
category: blog
description: 使用Python实现基于重心临界多边形的排样算法
---

#二维排样算法-重心临界多边形-Python实现

[TOC]

##1. 算法说明

###1.1 概述

算法分为两个部分，一是**基于重心NFP的零件定位**，二是**启发式零件排序**。

###1.2 基于重心NFP的零件定位

该部分通过计算NFP，寻找能使零件重心最低的位置。因为**零件重心越低，排样算法的效率越高**[[1]](http://cdmd.cnki.com.cn/Article/CDMD-10248-2007153774.htm)。

####1.2.1 流程图

![Git Bash](./images/chart-SA.png)

####1.2.2 算法描述

* Step 1. 输入参数。
	*  a. 初始约束：根据板材大小，确认初始边界，以及优先方向，本文优先策略是最低最左。
	*  b. 旋转空间：零件旋转角度空间，定义为$Rotation=\left \{ \rho_{1},\rho_{2},...,\rho_{n}\right \}$。本文只有两个角度，所以$Rotation=\left \{ 0^{\circ},180^{\circ}\right \}$。
	*  c. 初始步长以及迭代次数：计算最低重心时用来搜索边界的初始步长$delta_{0}$，还有用来优化精度的迭代搜索次数$k$。
	*  d. 零件集合：$P=\left \{ p_{1},p_{2},...,p_{m}\right \}$，零件个数为$m$。

* Step 2. 计算零件重心位置。重心集合定义为
$$center=\begin{pmatrix}
 c_{11}  &  c_{12}  & ... & c_{1n}\\ 
 c_{21}  &  c_{22}  & ... & c_{2n} \\ 
 ...  &  ... & ... & ...\\ 
 c_{m1}  &  c_{m2}  & ... & c_{mn} 
\end{pmatrix}_{m\times n}$$

* Step 3. 开始循环，依次将零件集合$P$中的元素定位。

* Step 4. 初始化最低重心位置$Y_{min}$，初始值为约束边界的最低位置。

* Step 5. 开始循环搜素最低重心位置。
	* a. 按步长搜索边界和旋转空间，计算NFP。初始步长为$delta_{0}$。得到最低重心位置$Y_{min}$，以及相应的$\rho$和边界位置。
	* b. 更新步长，令$delta_{i+1}=delta_{i}/10$。
	* c. 重复a步骤，共迭代$k$次结束循环。

*Step 6. 根据Step 5得到的$\rho$和边界位置更新边界约束。完成定位。

*Step 7. 重复Step 4，直到零件集合中所有零件都定位到约束空间中，结束循环。


###1.2 启发式零件排序

####1.2.1 解空间和最优解

上一节中，零件的插入顺序一般按照面积从大到小排列。改变定位顺序，定位也会随之改变。使得利用率（或者排样高度？）最高的定位顺序即为最优解，而解空间就是零件编号的排序方式。

假设零件集合中有$m$个零件，解的个数就是$m!$个，遍历搜索显然是不能接受的。

####1.2.2 搜索策略

定义如下三种搜索策略：

* 1级：2-change，随机选取两个零件交换排序位置。
* 2级：n-change，n个零件一组，随机选取两组交换。
* 3级：插入交换，随机选取一个零件插入到其他随机位置。

我们认为以上三种策略的扰动范围依次减小。

####1.2.3 模拟退火搜索

模拟退火的基本原理可以参考[维基](https://en.wikipedia.org/wiki/Simulated_annealing)，这里不做太多介绍。下面主要说明本文用到的输入参数和一些算法改进。

（1）参数说明

* 目标函数：$E(S)$定义为当前状态$state$的板材利用率或高度。
* 领域函数：即更新策略，这里我们根据温度不同选择不同的更新策略。**目的是越接近最优解的时候能选择扰动范围越小的搜索空间**。
* 稳定准则：使用抽样判断是否稳定，判断条件有以下三个，为或关系：
	* 检验目标函数的均值是否稳定。
	* 连续若干个抽样值变化小于阈值。
	* 抽样次数等于给定数量。

（2）算法流程

这里的算法流程参考上海交通大学刘胡瑶的博士论文：[基于临界多边形的二维排样算法研究](http://cdmd.cnki.com.cn/Article/CDMD-10248-2007153774.htm)。

![Git Bash](./images/algorithm-SA.png)


##2. 实现方法

###2.1 版本说明

* CentOS 7.0
* Python 2.7
* OpenCV 2.4
* numpy

###2.2 代码模块

略

----------------------------------------
[1] 刘胡瑶. 基于临界多边形的二维排样算法研究[D]. 上海: 上海交通大学, 2007.




