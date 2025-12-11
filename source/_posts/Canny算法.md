---
title: Canny算法
date: 2024-01-09 10:01
tags: 算法
---

> 原文发表在[公众号](https://mp.weixin.qq.com/s/hUG9ZqSJKRXZnGpUoc_daQ)

### 环境

Visual Studio 2022

OpenCV 4.5.2

```apl
#define USE_OPENCV

#ifdef USE_OPENCV

#ifndef CV
#define CV

#include <opencv2/opencv.hpp>

#define CV_VERSION_ID CVAUX_STR(CV_MAJOR_VERSION) CVAUX_STR(CV_MINOR_VERSION) CVAUX_STR(CV_SUBMINOR_VERSION)

#ifdef _DEBUG
#include <opencv2/core/utils/logger.hpp>
#define cvLIB(name) "opencv_" name CV_VERSION_ID "d"
#else
#define cvLIB(name) "opencv_" name CV_VERSION_ID
#endif

#pragma comment(lib, cvLIB("world"))

#endif // !CV


#endif // USE_OPENCV
```

### 算法基本原理

#### 高斯滤波

高斯核公式
$$
f(x) = \frac{1}{\sqrt{2\sigma}\pi}e^{-\frac{x^2}{2\sigma^2}}
$$

$$
f(x,y) = \frac{1}{2\sigma\pi^2}e^{-\frac{x^2+y^2}{2\sigma^2}}
$$


高斯核生成代码

```c++
cv::Mat cv_getGaussianKernel(int ksize, double sigma)
{
    assert(ksize > 0 || sigma > 0);

    int n = static_cast<int>(ksize > 0 ? ((ksize / 2) * 2 + 1) : (3 * sigma * 2 + 1));

    if (sigma <= 0)
    {
        if (n == 1)
        {
            return (cv::Mat_<double>(n, 1) << 1);
        }
        else if (n == 3)
        {
            return (cv::Mat_<double>(n, 1) << 0.25, 0.5, 0.25);
        }
        else if (n == 5)
        {
            return (cv::Mat_<double>(n, 1) << 0.0625, 0.25, 0.375, 0.25, 0.0625);
        }
        else if (n == 7)
        {
            return (cv::Mat_<double>(n, 1) << 0.03125, 0.109375, 0.21875, 0.28125, 0.21875, 0.109375, 0.03125);
        }
        else if (n == 9)
        {
            return (cv::Mat_<double>(n, 1) << 4 / 256, 13 / 256, 30 / 256, 51 / 256, 60 / 256, 51 / 256, 30 / 256, 13 / 256, 4 / 256);
        }
    }

    double sigmaN = sigma > 0 ? sigma : (0.3 * ((n - 1) * 0.5 - 1) + 0.8);
    double sigmaNN = -1.0 / 2 / std::pow(sigmaN, 2);
    int radium = n / 2;

    /// 高斯核需要归一化处理，故计算时去掉 1/(pi * sqrt(2*sigma))
    cv::Mat kernel = cv::Mat(cv::Size(1, n), CV_64FC1);
    for (int y = 0; y < kernel.rows; y++)
    {
        double sum_y2 = std::pow((y - radium), 2);
        kernel.ptr<double>(y)[0] = std::exp(sigmaNN * sum_y2);
    }
    kernel /= cv::sum(kernel)[0];

    return kernel;
}
cv::Mat cv_filterImage(cv::Mat src, int ksize, double sigma)
{
    cv::Mat kernel = cv_getGaussianKernel(5, 0.8);
    cv::Mat gauss_kernel = kernel * kernel.t();

    cv::Mat gauss_image;
    cv::filter2D(src, gauss_image, -1, gauss_kernel);
    return gauss_image;
}
```

#### 计算图像梯度

```c++
void cv_calcMagnitudesDirection(cv::Mat src,
                                cv::Mat& G_xy,
                                cv::Mat& G_theta,
                                int aperture_size,
                                bool L2gradient)
{
    cv::Mat Gx = cv::Mat::zeros(src.rows, src.cols, CV_64FC1);
    cv::Mat Gy = cv::Mat::zeros(src.rows, src.cols, CV_64FC1);
    G_xy = cv::Mat::zeros(src.rows, src.cols, CV_64FC1);
    cv::Sobel(src, Gx, CV_64FC1, 1, 0, aperture_size, 1, 0, cv::BORDER_REPLICATE);
    cv::convertScaleAbs(Gx, Gx);
    cv::Sobel(src, Gy, CV_64FC1, 0, 1, aperture_size, 1, 0, cv::BORDER_REPLICATE);
    cv::convertScaleAbs(Gy, Gy);
    /*
*L2gradient == false,G = |Gx|+|Gy|
*L2gradient == true,G = (Gx)^2 + (Gy)^2
*/
    if (L2gradient)
    {
        cv::sqrt(Gx.mul(Gx) + Gy.mul(Gy), G_xy);
    }
    else
    {
        G_xy = cv::abs(Gx) + cv::abs(Gy);
    }
    cv::normalize(G_xy, G_xy, 0, 0xFF, cv::NORM_MINMAX);

    /// 梯度方向
    G_theta = cv::Mat::zeros(src.rows, src.cols, CV_64FC1);
    for (size_t y = 0; y < src.rows; y++)
    {
        for (size_t x = 0; x < src.cols; x++)
        {
            G_theta.at<double>(y, x) = std::atan2(Gy.at<uchar>(y, x), Gx.at<uchar>(y, x)) * 180.0 / CV_PI;
        }
    }
}
```

#### 应用非极大值抑制

```c++
cv::Mat cv_nonMaximumSuppression(cv::Mat G_xy,
                                 cv::Mat G_theta)
{
    cv::Mat edge_NMS = cv::Mat::zeros(G_xy.rows, G_xy.cols, CV_64FC1);
    for (size_t y = 1; y < G_xy.rows - 1; y++)
    {
        for (size_t x = 1; x < G_xy.cols - 1; x++)
        {
            if (G_xy.at<uchar>(y, x) == 0)continue;

            double mag = G_xy.at<uchar>(y, x);//当前位置梯度幅值
            double angle = G_theta.at<double>(y, x);//当前位置梯度方向

            // NMS - 非极大值抑制
            // 垂直边缘--梯度方向为水平方向-3*3邻域内左右方向比较
            if (abs(angle) < 22.5 || abs(angle) > 157.5)
            {
                double left = G_xy.at<uchar>(y, x - 1);
                double right = G_xy.at<uchar>(y, x + 1);
                if (mag >= left && mag >= right)
                    edge_NMS.at<double>(y, x) = mag;
            }

            // 水平边缘--梯度方向为垂直方向-3*3邻域内上下方向比较
            if ((angle >= 67.5 && angle <= 112.5) || (angle >= -112.5 && angle <= -67.5))
            {
                double top = G_xy.at<uchar>(y - 1, x);
                double down = G_xy.at<uchar>(y + 1, x);
                if (mag >= top && mag >= down)
                    edge_NMS.at<double>(y, x) = mag;
            }

            // +45°边缘--梯度方向为其正交方向-3*3邻域内右上左下方向比较
            if ((angle > 112.5 && angle <= 157.5) || (angle > -67.5 && angle <= -22.5))
            {
                double right_top = G_xy.at<uchar>(y - 1, x + 1);
                double left_down = G_xy.at<uchar>(y + 1, x - 1);
                if (mag >= right_top && mag >= left_down)
                    edge_NMS.at<double>(y, x) = mag;
            }

            // +135°边缘--梯度方向为其正交方向-3*3邻域内右下左上方向比较
            if ((angle >= 22.5 && angle < 67.5) || (angle >= -157.5 && angle < -112.5))
            {
                double left_top = G_xy.at<uchar>(y - 1, x - 1);
                double right_down = G_xy.at<uchar>(y + 1, x + 1);
                if (mag >= left_top && mag >= right_down)
                    edge_NMS.at<double>(y, x) = mag;
            }
        }
    }

    return edge_NMS;
}
```

#### 双阈值处理和抑制孤立弱边缘完成边缘检测

```c++
cv::Mat cv_doubleThreshold(cv::Mat edge_NMS, double low_thresh,
                           double high_thresh)
{
    cv::Mat edge_canny = cv::Mat::zeros(edge_NMS.rows, edge_NMS.cols, CV_8UC1);

    // 双阈值处理
    for (size_t y = 1; y < edge_NMS.rows - 1; y++)
    {
        for (size_t x = 1; x < edge_NMS.cols - 1; x++)
        {
            float mag = edge_NMS.at<double>(y, x);
            if (mag >= high_thresh)
            {
                edge_NMS.at<double>(y, x) = high_thresh;
                edge_canny.at<uchar>(y, x) = 255;
            }
            else if (mag < low_thresh)
            {
                edge_NMS.at<double>(y, x) = 0;
            }
            else
            {
                edge_NMS.at<double>(y, x) = low_thresh;
            }
        }
    }

    // 抑制孤立弱边缘完成边缘检测
    for (size_t y = 1; y < edge_canny.rows - 1; y++)
    {
        for (size_t x = 1; x < edge_canny.cols - 1; x++)
        {
            if (edge_NMS.at<double>(y, x) == low_thresh)
            {
                /// 3*3 区域强度
                cv::Rect rect = cv::Rect(x - 1, y - 1, 3, 3);

                cv::Mat rect_image = edge_NMS(rect);
                for (int r = 0; r < rect_image.rows; r++)
                {
                    for (int c = 0; c < rect_image.cols; c++)
                    {
                        if (rect_image.at<double>(r, c) == high_thresh)
                        {
                            edge_canny.at<uchar>(y, x) = 255;
                            break;
                        }
                    }
                }
            }
        }
    }
    return edge_canny;
}
```

### 完整代码实现

```c++
int main()
{
    cv::Mat image = cv::imread("lena.jpg", cv::IMREAD_GRAYSCALE);

    cv::Mat gauss_image;
    cv::GaussianBlur(image, gauss_image, cv::Size(5, 5), 1.4);

    cv::Mat canny_image, canny_image2;
    cv::Canny(gauss_image, canny_image, 20, 60);
    cv_canny(image, canny_image2, 20, 60);

    cv::Mat result = cv::Mat::zeros(cv::Size(image.cols * 3 + 10 * 2, image.rows), image.type());
    image.copyTo(result(cv::Rect(0, 0, image.cols, image.rows)));
    canny_image.copyTo(result(cv::Rect(image.cols + 10, 0, image.cols, image.rows)));
    canny_image2.copyTo(result(cv::Rect(2 * (image.cols + 10), 0, image.cols, image.rows)));

    cv::imshow("Canny", result);
    cv::waitKey(0);
    system("pause");
}
```



![图片](/images/Canny算法/1.png)
