---
layout:     post
title:      印刷品背景去除 - OpenCV - C/C++实现
category: blog
description: 去除印刷品图像的背景
---

##1. 概述

算法目标是自动去除印刷品图像中的非白色背景。简单的阈值分割和填充算法都很难满足要求，我的思路是这样的：

1. 用`漫水填充算法`简单填充背景，得到`图1`。
2. 提取原图像的内外轮廓并填充，然后与原图像叠加以锐化边缘，得到`图2`。
3. 填充`图2`的背景，然后与`图1`叠加，得到效果图。

##2. 优势与缺陷

以上算法明显的优点有两个：

> 1）**清晰地保留文字信息**。在实验过程中，我们发现`漫水填充算法`可以非常清晰地保留文字骨架，不会出现发虚现象，但是由于部分字迹已经褪色，该算法会直接剔除这些字迹。解决这个问题的方法是轮廓填充和锐化边缘，相当于在漫水填充时使用了蒙版。
> 2）**完美去除背景噪声**。二值化方法很大的缺陷是会在背景上留下噪声，如果印刷品有水渍或者阴影，二值化方法的效果会非常糟糕。本文算法较好解决该问题，基本不会有背景噪声留下。

由于时间匆忙，算法的缺陷也遗留很多：

> 1）**孔洞背景很难填充**。因为我们用的是填充算法，闭区域内的背景很难消除。我做了一定改进，按照步长遍历图像，遇到大于灰度阈值的像素则开始填充，虽然改进效果也很不错，但不并不能完全填充孔洞，而且有可能误填充字迹。
> 2）**部分文字缺失**。仍然存在部分字迹缺失。这一问题有可能是边缘检测和轮廓提取阈值选定的问题，通过调参会有改善。

##3. 部分代码

``` cpp
#include <cv.h>
#include <opencv2/core.hpp> 
#include <opencv2/opencv.hpp>  
#include <opencv2/imgproc/imgproc.hpp> 
#include <highgui.h>
#include <math.h>
#include "CEdge.h"
#include <iostream>

using namespace std;
using namespace cv;

void sharpenImage1(const Mat &image, Mat &result);
void FillInternalContours(IplImage *pBinary);

int main(int argc, char* argv[])
{
	char srcName[] = "test.jpg";

	IplImage * src = cvLoadImage(srcName, 0);
	IplImage * img = cvLoadImage(srcName, 0);
	IplImage * imgSrc = img;
	CvSize sizeSrc = cvGetSize(imgSrc);

	CvScalar mean = 0;
	CvScalar std = 0;
	double minValue = 0, maxValue = 0;

	cvAvgSdv(imgSrc, &mean, &std);
	cvMinMaxLoc(imgSrc, &minValue, &maxValue);

	Rect ccomp;

	CEdge edge;
	edge.extractEdge(srcName);
	imgSrc = edge.getEdge();
	
	for (int i = 0; i < sizeSrc.width; i+=5)
	{
		for (int j = 0; j < sizeSrc.height; j+=5)
		{
			float tmp = CV_IMAGE_ELEM(src, float, j, i);
			if (tmp > mean.val[0] + std.val[0])
			{
				cvFloodFill(src, Point(i, j), CV_RGB(255, 255, 255), Scalar(20, 20, 20), Scalar(50, 50, 50));
			}
		}
	}

	FillInternalContours(imgSrc);
	
	cvMax(img, imgSrc, imgSrc);
	for (int i = 0; i < sizeSrc.width; i += 5)
	{
		for (int j = 0; j < sizeSrc.height; j += 5)
		{
			float tmp = CV_IMAGE_ELEM(imgSrc, float, j, i);
			if (tmp > mean.val[0] + std.val[0])
			{
				cvFloodFill(imgSrc, Point(i, j), CV_RGB(255, 255, 255), Scalar(20, 20, 20), Scalar(50, 50, 50));
			}
		}
	}
	
	cvMin(src, imgSrc, imgSrc);

	cvNamedWindow("Result", CV_WINDOW_AUTOSIZE);
	cvShowImage("Result", imgSrc);

	cvWaitKey(0);

	cvDestroyWindow("Result");
	cvReleaseImage(&src);
	cvReleaseImage(&img);
	cvReleaseImage(&imgSrc);

	return 0;
}

void FillInternalContours(IplImage *pBinary)
{
	double dConArea;
	CvSeq *pContour = NULL;
	CvSeq *pConInner = NULL;
	CvMemStorage *pStorage = NULL;
	if (pBinary)
	{
		pStorage = cvCreateMemStorage(0);
		cvFindContours(pBinary, pStorage, &pContour, sizeof(CvContour), CV_RETR_CCOMP, CV_CHAIN_APPROX_SIMPLE);
		cvDrawContours(pBinary, pContour, 0, 0, 2, 1, 8, cvPoint(0, 0));
		for (; pContour != NULL; pContour = pContour->h_next)
		{
			for (pConInner = pContour->v_next; pConInner != NULL; pConInner = pConInner->h_next)
			{
				cvDrawContours(pBinary, pConInner, 0, 255, 0, 1, 8, cvPoint(0, 0));
			}
		}
		cvReleaseMemStorage(&pStorage);
		pStorage = NULL;
	}
}
```

##4. 测试用例

###4.1 未遍历填充的用例

直接漫水填充的效果：

![Git Bash](/images/opencv/2.jpg)

本文算法的效果：

![Git Bash](/images/opencv/1463627599019.png)

可以发现很多缺失字母被补充。

其他用例：

![Git Bash](/images/opencv/1463629607638.png)

![Git Bash](/images/opencv/1463629563111.png)

![Git Bash](/images/opencv/1463629668590.png)

###4.2 遍历填充的用例

![Git Bash](/images/opencv/1463629732455.png)

------------------------------------------
Ps: 以上用例均来自Google图片。

