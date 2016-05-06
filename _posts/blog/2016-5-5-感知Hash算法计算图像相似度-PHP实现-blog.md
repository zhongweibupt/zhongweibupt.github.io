---
layout:     post
title:      感知Hash算法计算图像相似度-PHP实现
category: blog
description: 使用PHP实现Perceptual-Hash
---

##原理说明

###概述
**基于感知Hash算法计算图像相似度**最大的优点是**简单快速，适用于大规模图像匹配**。 

该算法的原理与Google搜索引擎的`SIMHash`算法有相同之处，通过生成文本或图片的“指纹”（Hash值），计算Hash值的汉明距离，来表征图片之间的相似度。汉明距离越小，图片越相似。

因为Hash码一般只有有限的位数，可以短时间筛选出相似图片。如果需要精匹配，再考虑`SIFT特征匹配`等算法。目前，许多图像搜索引擎都在使用`Perceptual Hash`算法。

###算法步骤

1. 缩小尺寸：将图像缩小到8*8的尺寸，总共64个像素。这一步的作用是去除图像的细节，只保留结构/明暗等基本信息，摒弃不同尺寸/比例带来的图像差异； 
2. 简化色彩：将缩小后的图像，转为64级灰度，即所有像素点总共只有64种颜色； 
3. 计算平均值：计算所有64个像素的灰度平均值； 
4. 比较像素的灰度：将每个像素的灰度，与平均值进行比较，大于或等于平均值记为1，小于平均值记为0； 
5. 计算哈希值：将上一步的比较结果，组合在一起，就构成了一个64位的整数，这就是这张图像的指纹。组合的次序并不重要，只要保证所有图像都采用同样次序就行了； 
6. 得到指纹以后，就可以对比不同的图像，看看64位中有多少位是不一样的。在理论上，这等同于"汉明距离”(Hamming distance,在信息论中，两个等长字符串之间的汉明距离是两个字符串对应位置的不同字符的个数)。如果不相同的数据位数不超过5，就说明两张图像很相似；如果大于10，就说明这是两张不同的图像。 

##实现方法

###版本说明

* CentOS 7.0
* PHP 5.4
* Composer 1.0.3
* jenssegers/imagehash

###环境部署

略

###部分代码说明

计算图片的Hash值：

``` php
use Jenssegers\ImageHash\ImageHash;

$hasher = new ImageHash;
$hash = $hasher->hash('path/to/image.jpg');
```

计算两个Hash码的汉明距离：

``` php
$distance = $hasher->distance($hash1, $hash2);
```

计算两张图片的相似度：
``` php
use Jenssegers\ImageHash\Implementations\DifferenceHash;
use Jenssegers\ImageHash\ImageHash;

$implementation = new DifferenceHash;
$hasher = new ImageHash($implementation);
$distance = $hasher->compare('path/to/image1.jpg', 'path/to/image2.jpg');
```

###测试用例

这里使用的用例是指定的`树叶图像`。我们选取以下8张正面图片，分别编号从`1-8`，以及8张反面图片，分别编号从`1_1-8_1`。

![Git Bash](/images/图片1.png)

然后用以下代码计算对应的汉明距离：

``` php
<?php
require '/root/vendor/autoload.php';
use Jenssegers\ImageHash\Implementations\DifferenceHash;
use Jenssegers\ImageHash\ImageHash;

$image_path = '/data/wwwroot/default/cv/images_test/';
$implementation = new DifferenceHash;
$hasher = new ImageHash($implementation);

echo "<table border='1'>";
for( $i = 1; $i <=8; $i++)
{
    echo "<tr>";
    for( $j = 1; $j <=8; $j++)
    {
        $distance = $hasher->compare($image_path.$j.'_1.jpg', $image_path.$i.'.jpg');
        echo "<td>".$distance."</td>";
    }
    echo "</tr>";
}
echo "</table>";
?>

```

得到结果如下：
<table border='1'><tr><td></td><td>image 1</td><td>image 2</td><td>image 3</td><td>image 4</td><td>image 5</td><td>image 6</td><td>image 7</td><td>image 8</td></tr><tr><td>image 1_1</td><td>6</td><td>31</td><td>27</td><td>21</td><td>26</td><td>24</td><td>25</td><td>20</td></tr><tr><td>image 2_1</td><td>28</td><td>17</td><td>25</td><td>27</td><td>28</td><td>24</td><td>27</td><td>30</td></tr><tr><td>image 3_1</td><td>23</td><td>26</td><td>8</td><td>12</td><td>7</td><td>9</td><td>10</td><td>7</td></tr><tr><td>image 4_1</td><td>27</td><td>24</td><td>16</td><td>14</td><td>13</td><td>19</td><td>18</td><td>13</td></tr><tr><td>image 5_1</td><td>29</td><td>24</td><td>10</td><td>10</td><td>11</td><td>5</td><td>10</td><td>13</td></tr><tr><td>image 6_1</td><td>31</td><td>30</td><td>10</td><td>20</td><td>11</td><td>11</td><td>12</td><td>11</td></tr><tr><td>image 7_1</td><td>28</td><td>23</td><td>5</td><td>13</td><td>8</td><td>6</td><td>7</td><td>12</td></tr><tr><td>image 8_1</td><td>32</td><td>27</td><td>19</td><td>13</td><td>20</td><td>16</td><td>15</td><td>18</td></tr></table>

##存在问题和解决方案

pHash算法可以快速从海量图片中筛选相似图片，但不能解决精匹配问题，因为图片变换（旋转、缩放、投影）对pHash算法的影响较大。待解决问题总结如下：

1. 精匹配问题。利用pHash快速匹配后可以用`SIFT特征，互信息、互相关、PNSR等`配准和计算相似度。
2. 图像的自动ReSize问题。根据样本模板自动调整待搜索图片的尺寸和角度，算法待设计。 (Ps. 最好采用窗口搜索的方法，类似于目标检测，比图像变换效果要好。)

