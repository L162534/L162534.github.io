---
layout:  post
title:  opencv-DFT
subtitle:  keep moving
date:  2020-05-28
author:  L&X
catalog:  true
tags:
    -opencv
    -C++
---

# 离散傅里叶变换

首先读入图片：

```c++
Mat img = imread("images/5low.jpg", IMREAD_GRAYSCALE);
imshow("1", img);
```

将原图像大小优化成适合DFT运算的大小,并对添加的部分进行填充，本例中采用补零：

```c++
Mat padImg;
int m = getOptimalDFTSize(img.rows);
int n = getOptimalDFTSize(img.cols);
copyMakeBorder(img, padImg, 0, m - img.rows, 0, n - img.cols, BORDER_CONSTANT, Scalar::all(0)); 
```

> copyMakeBorder(InputArray src, OutputArray dst, int top, int bottom, int left, int right, int borderType, const Scalar& value = Scalar() )
>
> * top表示填充顶部的像素点行数，left表示填充左端的像素点列数；
>
> * borderType：
>
>   ​    BORDER_CONSTANT    = 0, //!< `iiiiii|abcdefgh|iiiiiii`  表示用i值填充
>   ​    BORDER_REPLICATE   = 1, //!< `aaaaaa|abcdefgh|hhhhhhh`表示用边缘值填充
>   ​    BORDER_REFLECT     = 2, //!< `fedcba|abcdefgh|hgfedcb`表示镜像填充
>   ​    BORDER_WRAP        = 3, //!< `cdefgh|abcdefgh|abcdefg`用另一边的边缘像素填充
>   ​    BORDER_REFLECT_101 = 4, //!< `gfedcb|abcdefgh|gfedcba`以边缘像素为对称轴镜像填充
>   ​    BORDER_TRANSPARENT = 5, //!< `uvwxyz|abcdefgh|ijklmno`

通过数组生成2维矩阵，1维表示实数，1维表示虚数：

```c++
Mat planes[] = { Mat_<float>(padImg), Mat::zeros(padImg.size(), CV_32F) };
Mat I_complex;
merge(planes, 2, I_complex);
```

进行DFT运算：

```c++
dft(I_complex, I_complex);
```

再将二维矩阵转换成数组，分离出实数和虚数：

```c++
split(I_complex, planes);
```

根据实数和虚数矩阵，计算出幅度矩阵：

```c++
Mat I_mag;
magnitude(planes[0], planes[1], I_mag);
```

将幅度用y= log（x+1）进行变换：

```c++
I_mag += Scalar::all(1);
log(I_mag,I_mag);
```

接下来要将频谱的直流点移动到图像中心，类似于matlab中的fftshift,首先要保证图像尺寸为偶数：

```C++
I_mag = I_mag(Rect(0, 0, I_mag.cols&-2, I_mag.rows&-2));
int w = I_mag.cols / 2;
int h = I_mag.rows / 2;
```

> I_mag.cols&-2是位运算，就是求出I_mag.cols的最大偶数

将图像分为四块，采用中间变量，交换赋值：

```
Mat q1(I_mag, Rect(0, 0, w, h));
Mat q2(I_mag, Rect(w, 0, w, h));
Mat q3(I_mag, Rect(0, h, w, h));
Mat q4(I_mag, Rect(w, h, w, h));
Mat temp;
q1.copyTo(temp);
q4.copyTo(q1);
temp.copyTo(q4);
q2.copyTo(temp);
q3.copyTo(q2);
temp.copyTo(q3);
```

为了显示频谱，将频谱图像进行归一化：

```c++
normalize(I_mag, I_mag, 0, 1, NORM_MINMAX);
```

测试代码：

```c++
#include "stdafx.h"
#include <iostream>
#include <opencv2/core.hpp>
#include <opencv2/imgproc.hpp>
#include <opencv2/imgcodecs.hpp>
#include <opencv2/highgui/highgui.hpp>
#include <string>

using namespace std;
using namespace cv;
Mat fun(const String filename)
{
	Mat img = imread(filename, IMREAD_GRAYSCALE);
	Mat padImg;
	int m = getOptimalDFTSize(img.rows);
	int n = getOptimalDFTSize(img.cols);
	copyMakeBorder(img, padImg, 0, m - img.rows, 0, n - img.cols, BORDER_CONSTANT, Scalar::all(0)); 
	Mat planes[] = { Mat_<float>(padImg), Mat::zeros(padImg.size(), CV_32F) };
	Mat I_complex;
	merge(planes, 2, I_complex);
	cout << I_complex.channels() << endl;
	dft(I_complex, I_complex);
	split(I_complex, planes);
	Mat I_mag;
	magnitude(planes[0], planes[1], I_mag);
	I_mag += Scalar::all(1);
	log(I_mag, I_mag);
	I_mag = I_mag(Rect(0, 0, I_mag.cols&-2, I_mag.rows&-2));
	int w = I_mag.cols / 2;
	int h = I_mag.rows / 2;
	Mat q1(I_mag, Rect(0, 0, w, h));
	Mat q2(I_mag, Rect(w, 0, w, h));
	Mat q3(I_mag, Rect(0, h, w, h));
	Mat q4(I_mag, Rect(w, h, w, h));
	Mat temp;
	q1.copyTo(temp);
	q4.copyTo(q1);
	temp.copyTo(q4);
	q2.copyTo(temp);
	q3.copyTo(q2);
	temp.copyTo(q3);
	normalize(I_mag, I_mag, 0, 1, NORM_MINMAX);
	return I_mag;
}
int main()
{
	Mat img1 = fun("images/arrow1.jpg");
	Mat img2 = fun("images/arrow2.jpg");
	Mat img3 = fun("images/arrow3.jpg");
	imshow("1", img1);
	imshow("2", img2);
	imshow("3", img3);
	waitKey(0);
    return 0;
}
```

