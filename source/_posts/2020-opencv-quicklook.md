---
title: opencv基础操作
date: 2020-02-29 09:20:21
tags: [opencv, c++]
description: <center>基础操作</center>
---
# opencv常用速查

opencv3.4.9，内容截取自官网教学文档

## 矩阵复制

```c++
//浅拷贝
Mat A, C;                                 // 只创建信息头部分
A = imread(argv[1], CV_LOAD_IMAGE_COLOR); // 这里为矩阵开辟内存

Mat B(A);                                 // 使用拷贝构造函数

C = A;                                    // 赋值运算符

Mat D (A, Rect(10, 10, 100, 100) ); // using a rectangle
Mat E = A(Range:all(), Range(1,3)); // using row and column boundaries

//深拷贝
Mat F = A.clone();
Mat G;
A.copyTo(G);
```

## 矩阵创建

```c++
Mat M(2,2, CV_8UC3, Scalar(0,0,255));   

//超过二维的矩阵
int sz[3] = {2,2,2}; 
Mat L(3,sz, CV_8UC(1), Scalar::all(0));

//开辟内存
M.create(4,4, CV_8UC(2));
```

## 图像扫描

```c++
//isContinuous() 判断图像内存空间是否连续
//c风格
int channels = I.channels();
int nRows = I.rows;
int nCols = I.cols * channels;
if (I.isContinuous())
{
    nCols *= nRows;
    nRows = 1;
}

int i,j;
uchar* p; 
for( i = 0; i < nRows; ++i)
{
    p = I.ptr<uchar>(i);
    for ( j = 0; j < nCols; ++j)
    {
        p[j] = table[p[j]];             
    }
}

//迭代器，比c风格慢
const int channels = I.channels();
switch(channels)
{
    case 1: 
        {
            MatIterator_<uchar> it, end; 
            for( it = I.begin<uchar>(), end = I.end<uchar>(); it != end; ++it)
                *it = table[*it];
            break;
        }
    case 3: 
        {
            MatIterator_<Vec3b> it, end; 
            for( it = I.begin<Vec3b>(), end = I.end<Vec3b>(); it != end; ++it)
            {
                (*it)[0] = table[(*it)[0]];
                (*it)[1] = table[(*it)[1]];
                (*it)[2] = table[(*it)[2]];
            }
        }
}

//lookup table,使用查找替换方式避免重复运算
Mat lookUpTable(1, 256, CV_8U);
uchar* p = lookUpTable.ptr();
for( int i = 0; i < 256; ++i)
    p[i] = table[i];

LUT(I, lookUpTable, J);
```

## 滤波

```c++
Mat kernel = (Mat_<char>(3,3) <<  0, -1,  0,
              -1,  5, -1,
              0, -1,  0);
filter2D( src, dst1, src.depth(), kernel );
```

## 读写显示，颜色转换

```c++
Mat img = imread(filename, IMREAD_GRAYSCALE);
imwrite(filename, img);
Mat grey;
cvtColor(img, grey, COLOR_BGR2GRAY);
namedWindow("image", WINDOW_AUTOSIZE);
imshow("image", img);
waitKey();
```

## 像素访问

```c++
//单通道
Scalar intensity = img.at<uchar>(y, x);
Scalar intensity = img.at<uchar>(Point(x, y));

//多通道
Vec3b intensity = img.at<Vec3b>(y, x);
uchar blue = intensity.val[0];
uchar green = intensity.val[1];
uchar red = intensity.val[2];

img.at<uchar>(y, x) = 128;
Point2f point = pointsMat.at<Point2f>(i, 0);

//roi
Rect r(10, 10, 100, 100);
Mat smallImg = img(r);
```

## 亮度对比度

```c++
Mat lookUpTable(1, 256, CV_8U);
uchar* p = lookUpTable.ptr();
for( int i = 0; i < 256; ++i)
    p[i] = saturate_cast<uchar>(pow(i / 255.0, gamma_) * 255.0);
Mat res = img.clone();
LUT(img, lookUpTable, res);
```

