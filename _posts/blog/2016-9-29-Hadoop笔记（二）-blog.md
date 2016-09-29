---
layout:     post
title:    Hadoop笔记（二）
category: blog
description: Hadoop笔记（二）
---
#Hadoop笔记（二）

[TOC]

##18. 如何编译和调试Hadoop代码？

1. 使用Eclipse

从Github上下载[hadoop-eclipse插件](https://github.com/winghc/hadoop2x-eclipse-plugin)，然后将hadoop-eclipse-plugin-2.6.0.jar放置在eclipse的plugins目录。

重启eclipse，在windows->preferences可以看到Hadoop Map/Reduce选项。

![Alt text](./1474872245747.png)

`Hadoop installation directory`是Hadoop的目录。

打开Windows->Open Perspective->Other，选择Map/Reduce。

![Alt text](./1474873303249.png)

点击左上角配置Map/Reduce Location选项，输入Location Name，任意名称即可。配置Map/Reduce Master和DFS Mastrer，Host和Port配置成与core-site.xml的设置一致即可。

![Alt text](./1474873292166.png)

点击左侧的DFSLocations—>myhadoop（上一步配置的location name)，如能看到user，表示安装成功。

![Alt text](./1475142073496.png)



2. 使用Maven

参考使用[Maven搭建Hadoop](http://blog.csdn.net/kongxx/article/details/42339581)。


3. 使用Ant


##19. MapReduce编程实例I：Top-N问题



##20. MapReduce编程实例II：K-means聚类


##21. MapReduce编程实例III：贝叶斯分类