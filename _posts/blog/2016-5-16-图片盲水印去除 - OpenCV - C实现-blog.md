---
layout:     post
title:      图片盲水印去除 - OpenCV - C/C++实现
category: blog
description: 自动检测水印和复原图像
---

##1. 概述

算法目标是批量检测和去除图片的水印，去除部分使用OpenCV的InPaint接口不难实现，终点在于如何识别和分割水印。

我的想法比较偷懒，主要是以下几步：
1. 将具有相同水印的图片作为一组，取其中一张图片与其他图片（注意用来配准的图像与原图像差别越大越好）进行配准，得到匹配的特征点集即为水印区域。
2. 然后用得到的特征点集匹配相同组的图片，将匹配区域直接分割扣去。
3. 调用OpenCV的InPaint复原图像，完成。

以上算法的本质其实是**识别一组图像中的共同元素，然后去除该元素，并进行复原。**
(除本篇介绍的方法以外，我还考虑了反向投影方法，将在以后的文章中介绍。)

##2. 算法实现

###2.1 提取水印

![Git Bash](/images/opencv/1463457407355.png)

以上两张图像具有相同水印，其实我们有很简便的方法，将两张图像二值化直接相减就能提取到水印。但是并不是所有水印都能找到合适的图片，所以还是需要做图像配准。

这里我选择了使用轮廓特征配准，一方面减少配准算法的复杂度，一方面便于后期的图像分割。首先我们提取图像的Canny算子：

![Git Bash](/images/opencv/1463457507203.png)

然后我们提取图像的轮廓信息：

![Git Bash](/images/opencv/1463538163838.png)

轮廓匹配通常的做法是使用Hu矩，这里我们直接调用了OpenCV的Match方法。

为找出两张图片共同的轮廓，我们以其中一张作为模板，遍历图中的轮廓信息，与另一张图片的轮廓进行匹配。筛选出最匹配的轮廓，即为水印位置。

我们在其中一张图片上标出识别的水印位置：

![Git Bash](/images/opencv/1463545029114.png)

最后输出水印图像以及轮廓特征的数据。（改进方案：对两张图片匹配的轮廓求交集，这样可以去除水印位置的噪声信息。如果增多图片数量，用两张以上的图片提取水印效果会更好。）


###2.2 匹配分割

这一节分为两步，一是识别待复原图像的水印位置，而是将水印作为蒙版分割图像。我们以下图为例演示分割：

![Git Bash](/images/opencv/11.jpg)

识别水印位置：

![Git Bash](/images/opencv/1463551922807.png)

实际应用场景中，匹配位置不可能非常准确，优化的做法有两种，一种是继续精匹配，一种是识别出大致区域后用漫水填充法分割目标区域。

分割图像：

![Git Bash](/images/opencv/1463558437342.png)

这里的做法是在识别区域内，将待复原图像的轮廓内像素填充为0。


###2.3 图像复原

 最后一步，调用InPaint函数复原图像。首先输出一个蒙版图像，作为Mask：

![Git Bash](/images/opencv/1463559189130.png)

复原之后的效果：

![Git Bash](/images/opencv/1463562970577.png)

下面是其他用例的复原效果：

![Git Bash](/images/opencv/1463563160076.png)

![Git Bash](/images/opencv/1463563246958.png)

![Git Bash](/images/opencv/1463563358366.png)

![Git Bash](/images/opencv/1463563451500.png)

##3. 存在问题

这个算法的缺陷其实很多，主要包括以下几个：

1. **必须分组处理图片。**因为是盲水印，我们只能通过图像配准的方法得到水印模板。
2. **容易漏过细小轮廓。**本文介绍的算法对于矩形水印的处理效果最好，细微的水印很难处理。
3. **复原效果依赖参数调节**如果要批量处理图片，需要人工调参之后才能使用，否则复原效果无法保证。
4. **复杂度较高。**当然，主要可能是因为我还没有重构代码……

重构代码后我会放到GitHub上，下面的代码片段只包含识别水印位置、分割图像、复原等功能，当然我还没有重构，结构很乱……

##4. 代码示例


``` cpp
#include "opencv2/highgui/highgui.hpp"
#include "opencv2/imgproc/imgproc.hpp"
#include "CEdge.h"
#include <iostream>
#include <stdio.h>

using namespace std;
using namespace cv;

int Match(IplImage* imgRGB, IplImage* imgMask, IplImage* img, char src[], char tmp[])
{
	IplImage* imgSrc = img;
	IplImage* imgTemp = cvLoadImage(tmp, 0);
	IplImage* imgContourSrc = cvCreateImage(cvGetSize(imgSrc), IPL_DEPTH_32F, 1);
	IplImage* imgContourTemp = cvCreateImage(cvGetSize(imgTemp), IPL_DEPTH_32F, 1);
	IplImage* imgResult = cvCreateImage(sizeResult, IPL_DEPTH_32F, 1);
	IplImage* imgPart = cvCreateImage(cvGetSize(imgTemp), IPL_DEPTH_32F, 1);
	IplImage* imgContour = cvCreateImage(sizeTemp, IPL_DEPTH_32F, 1);

	cvZero(imgContourSrc);
	cvZero(imgContourTemp);

	CEdge edgeTemp;
	edgeTemp.extractEdge(tmp);
	imgTemp = edgeTemp.getEdge();

	CvSeq * contoursSrc = 0;
	CvMemStorage * storageSrc = cvCreateMemStorage(0);

	cvFindContours(imgSrc, storageSrc, &contoursSrc, sizeof(CvContour),
		CV_RETR_TREE, CV_CHAIN_APPROX_SIMPLE, cvPoint(0, 0));

	cvDrawContours(imgContourSrc, contoursSrc,
		CV_RGB(0, 0, 255), CV_RGB(255, 0, 0),
		2, -1, 8);

	CvSeq * contoursTemp = 0;
	CvMemStorage * storageTemp = cvCreateMemStorage(0);

	cvFindContours(imgTemp, storageTemp, &contoursTemp, sizeof(CvContour),
		CV_RETR_TREE, CV_CHAIN_APPROX_SIMPLE, cvPoint(0, 0));

	cvDrawContours(imgContourTemp, contoursTemp,
		CV_RGB(0, 0, 255), CV_RGB(255, 0, 0),
		2, -1, 8);

	CvSize sizeSrc = cvGetSize(imgSrc);
	CvSize sizeTemp = cvGetSize(imgTemp);
	CvSize sizeResult = cvSize(sizeSrc.width - sizeTemp.width + 1, sizeSrc.height - sizeTemp.height + 1);
	
	cvMatchTemplate(imgContourSrc, imgContourTemp, imgResult, 4);

	float dMax = 0.;
	CvPoint point = cvPoint(0, 0);

	for (int cx = 0; cx < sizeResult.width; cx++)
	{
		for (int cy = 0; cy < sizeResult.height; cy++)
		{
			float fTemp = CV_IMAGE_ELEM(imgResult, float, cy, cx);
			if (dMax < fTemp) 
			{
				dMax = fTemp;
				point = cvPoint(cx, cy); 
			}
		}
	}


	CvPoint point2 = cvPoint(point.x + sizeTemp.width, point.y + sizeTemp.height); 
	cvRectangle(img, point, point2, cvScalar(255));

	cvSetImageROI(img, cvRect(point.x, point.y, sizeTemp.width, sizeTemp.height));
	
	CvSeq * contours = 0;
	CvMemStorage * storage = cvCreateMemStorage(0);
	
	cvFindContours(imgTemp, storage, &contours, sizeof(CvContour),
		CV_RETR_TREE, CV_CHAIN_APPROX_SIMPLE, cvPoint(0, 0));

	cvDrawContours(imgContour, contours,
		CV_RGB(0, 0, 255), CV_RGB(0, 0, 255),
		2, 5, 8);
	
	int channels = imgRGB->nChannels;
	for (int cx = 0; cx < sizeTemp.width; cx++)
	{
		for (int cy = 0; cy < sizeTemp.height; cy++)
		{
			float fTemp = CV_IMAGE_ELEM(imgContour, float, cy, cx);
			if (255 == fTemp)  
			{
				int j = point.x + cx;
				int i = point.y + cy;
				(imgRGB->imageData + i*imgRGB->widthStep)[j* channels + 0] = 255;
				(imgRGB->imageData + i*imgRGB->widthStep)[j* channels + 1] = 255;
				(imgRGB->imageData + i*imgRGB->widthStep)[j* channels + 2] = 255;
			}
		}
	}
	
	for (int cx = 0; cx < sizeTemp.width; cx++)
	{
		for (int cy = 0; cy < sizeTemp.height; cy++)
		{
			float fTemp = CV_IMAGE_ELEM(imgContour, float, cy, cx);
			if (255 == fTemp)
			{
				int j = point.x + cx;
				int i = point.y + cy;
				cvSet2D(imgMask, i, j, 255);
			}
		}
	}

	cvResetImageROI(img);

	cvReleaseMemStorage(&storage);
	cvReleaseMemStorage(&storageSrc);
	cvReleaseMemStorage(&storageTemp);

	cvReleaseImage(imgSrc);
	cvReleaseImage(imgTemp);
	cvReleaseImage(imgContour);
	cvReleaseImage(imgPart);
	cvReleaseImage(imgContourSrc);
	cvReleaseImage(imgContourTemp);
	cvReleaseImage(imgResult);

	return 0;
}

int main()
{
	char src[] = "C:\\Users\\zhwei\\Desktop\\InPaint\\test\\test.jpg";
	char tmp[] = "C:\\Users\\zhwei\\Desktop\\InPaint\\test\\model.jpg";

	IplImage* img = cvLoadImage(src, 0);
	IplImage* imgRGB = cvLoadImage(src, 1);
	IplImage* imgMask = cvCreateImage(cvGetSize(img), IPL_DEPTH_8U, 1);
	IplImage* imgInpaint = cvCreateImage(cvGetSize(img), IPL_DEPTH_8U, 3);
	cvZero(imgMask);

	CEdge edgeSrc;
	edgeSrc.extractEdge(src);
	img = edgeSrc.getEdge();

	Match(imgRGB, imgMask, img, src, tmp);

	cvNamedWindow("img", CV_WINDOW_AUTOSIZE);
	cvShowImage("img", img);

	cvNamedWindow("RGB", CV_WINDOW_AUTOSIZE);
	cvShowImage("RGB", imgRGB);

	cvInpaint(imgRGB, imgMask, imgInpaint, 10, CV_INPAINT_NS);
	cvNamedWindow("Inpaint", CV_WINDOW_AUTOSIZE);
	cvShowImage("Inpaint", imgInpaint);
	cvWaitKey(0);

	cvDestroyWindow("img");
	cvDestroyWindow("RGB");
	cvDestroyWindow("Inpaint");
	
	cvReleaseImage(&img);
	cvReleaseImage(&imgRGB);
	cvReleaseImage(&imgMask);
	cvReleaseImage(&imgInpaint);

}
```
