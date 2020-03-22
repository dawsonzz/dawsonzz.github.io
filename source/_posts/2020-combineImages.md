---
title: 利用opencv将图像拼接为一张图片
date: 2020-03-22 09:29:24
tags: [opencv, C++]
description: <center></center>
---

## 使用colRange和rowrRange

```c++
//MatrixCom=[Matrix1 Matrix1] 并排拼接
void comMatbyCol(cv::Mat &Matrix1, cv::Mat &Matrix2, cv::Mat &MatrixCom)
{

	CV_Assert(Matrix1.rows == Matrix2.rows);//行数不相等，出现错误中断    
	MatrixCom.create(Matrix1.rows, Matrix1.cols + Matrix2.cols, Matrix1.type());
	cv::Mat temp = MatrixCom.colRange(0, Matrix1.cols);//创建的是对MatriCom的引用
    // 取得图像范围是MatrxiCom[:][0, Matrix1.cols) 不包括  Matrix1.cols
	Matrix1.copyTo(temp);
	cv::Mat temp1 = MatrixCom.colRange(Matrix1.cols, Matrix1.cols + Matrix2.cols);
	Matrix2.copyTo(temp1);
}
//MatrixCom=[Matrix1;Matrix1] 垂直拼接
void ImgSpliter::comMatbyRow(cv::Mat &Matrix1, cv::Mat &Matrix2, cv::Mat &MatrixCom)
{
	CV_Assert(Matrix1.cols == Matrix2.cols);//列数不相等，出现错误中断    
	MatrixCom.create(Matrix1.rows + Matrix2.rows, Matrix1.cols, Matrix1.type());
	cv::Mat temp = MatrixCom.rowRange(0, Matrix1.rows);
	Matrix1.copyTo(temp);
	cv::Mat temp1 = MatrixCom.rowRange(Matrix1.rows, Matrix1.rows + Matrix2.rows);
	Matrix2.copyTo(temp1);

}

//MatrixCom.colRange(0, Matrix1.cols) 等价于 
//MatrixCom(Range:all(), Range(0, Matrix1.cols))
//Mat E = A(Range:all(), Range(1,3)); // using row and column boundaries
//虽然它们的信息头不同，但通过任何一个对象所做的改变也会影响其它对象

```

## 使用ROI方式获取对应引用再进行拷贝

```c++
cv::Rect rec(1143, 0, 1036, 153)；
cv::Mat templateImg = img(rec)；
```

