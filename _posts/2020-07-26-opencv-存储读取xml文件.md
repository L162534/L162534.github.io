---
layout:  post
title:  opencv-存储读取xml文件
subtitle:  最近学习热情不高
date:  2020-07-26
author:  L&X
catalog:  true
tags:
    -opencv
    -C++
---

## 读取和写入xml文件

```
FileStorage fs;
fs.open(filename, FileStorage::WRITE);//打开写入通道
fs.open(filename, FileStorage::READ);//打开读取通道
fs.release();//关闭文件
```



## xml的存储结构

分为两种：sequence型和map型

1. sequence型的写入和读取

   ```c++
   fs << "名称" << "[" << 内容1 << 内容2 << "]"; //写入
   FileNode n = fs["名称"];//读取
   FileIterator it = n.begin(), it_end = n.end();
   for (;it!=it_end;++it)
       cout << (数据类型)*it
   ```

2. map型的写入和读取

   ```C++
   fs << "名称" << "{" << "子集名称1" << 内容1 << "子集名称2" << 内容2 << "}";//写入
   FileNode n = fs["名称"];
   cout << (数据类型)n["子集名称1"];
   cout << (数据类型)n["子集名称2"];
   ```


## 读取和写入自定义的数据结构

```c++
class Mydata
{
public:
	Mydata()
	{
		a = 1;
		b = "hello";
	}
	Mydata(int A, string B)
	{
		a = A;
		b = B;
	}
	void write(FileStorage& fs) const //这里设置常成员函数，避免改变成员变量的值
	{
		fs << "mydata" << "{" << "a" << a << "b" << b << "}";
	}
	void read(const FileNode& node) //将xml文件的内容读进对象，xml文件不能改变，用const修饰
	{
		a = (int)node["a"];
		b = (string)node["b"];
	}
public:
	int a;
	string b;
};

static void write(FileStorage& fs, const std::string&, const Mydata& m)
{
	m.write(fs);
}

static void read(const FileNode& node, Mydata& m, const Mydata& default_value = Mydata())
{
	if (node.empty())
		m = default_value;
	else
		m.read(node);
}

int main()
{
	FileStorage fs;
	fs.open("2.xml", FileStorage::WRITE);      
	write(fs, Mydata());
	return 0;
}
```

