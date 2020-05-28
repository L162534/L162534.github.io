---
layout:  post
title:  opencv基础知识
subtitle:  keep moving
date:  2020-05-28
author:  L&X
catalog:  true
tags:
    -opencv	
	-C++
---

# Mat相关的知识

## Mat类具有两个数据部分：
1. 矩阵头：包含矩阵大小、存储方法、存储地址、引用次数等信息
2. 存储像素矩阵的指针，值得注意的是，当读取一张图不作任何操作存储的矩阵是不会变的，但是我们用不同的数据结构去解析所产生的的图像不一样，比如转换矩阵数据类型：

```c++
temp = imread("image.jpg")
temp.convertTo(temp3, CV_8UC1)
//这里只改变了temp的矩阵头里的存储方法数据矩阵并没有变，当然前提是转换过程中精度不改变，精度改变了那数据矩阵还是会变
```
## Mat类的值传递
用赋值运算符、复制构造方法新建的对象只包含矩阵头

```c++
a = temp
Mat b(temp)   
//这里 a,b,temp均是指向图像矩阵的不同矩阵头，当通过自己的矩阵头更改像素矩阵数据时，就会影响其他的矩阵头
  
```

