---
title: OpenCV 单相机 内参数标定 （棋盘格）
date: 2024-01-31 15:48
tags: 算法
---

> 原文发表在[公众号](https://mp.weixin.qq.com/s/i2Zo0ns17NB_1JDBcdqJYg)

#### 1、运行环境

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

#### 2、获取棋盘格3D点坐标

```c++
void Init3DPoint(cv::Size patternSize, std::vector<cv::Point3f> &singlePatternPoint)
{
    for (int h = 0; h < patternSize.height; h++)
    {
        for (size_t w = 0; w < patternSize.width; w++)
        {
            cv::Point3f tempPoint(w * 5.0f, h * 5.0f, 0.0f);
            singlePatternPoint.push_back(tempPoint);
        }
    }
}
```

#### 3、获取棋盘格角点坐标

```c++
std::vector<cv::Point2f> GetChessboardCorners(cv::Mat image, cv::Size patternSize, int index)
{
    assert(image.data != nullptr);

    std::vector<cv::Point2f> pointBuf;
    auto find = cv::findChessboardCorners(image, patternSize, pointBuf,
                                          cv::CALIB_CB_ADAPTIVE_THRESH | cv::CALIB_CB_NORMALIZE_IMAGE);
    if (find)
    {
        cv::cornerSubPix(image, pointBuf, cv::Size(11, 11), cv::Size(-1, -1),
                         cv::TermCriteria(cv::TermCriteria::EPS + cv::TermCriteria::COUNT, 30, 0.0001));

        auto IsPointReverse = [](cv::Point2f pa, cv::Point2f pb) -> bool
        {
            double dis_pa = std::pow(pa.x, 2) + std::pow(pa.y, 2);
            double dis_pb = std::pow(pb.x, 2) + std::pow(pb.y, 2);
            return dis_pa >= dis_pb;
        };

        if (IsPointReverse(pointBuf[0], pointBuf[pointBuf.size() - 1]))
        {
            std::reverse(pointBuf.begin(), pointBuf.end());
        }

        if (1)
        {
            cv::Mat rgb;
            cv::cvtColor(image, rgb, cv::COLOR_GRAY2BGR);
            cv::drawChessboardCorners(rgb, patternSize, cv::Mat(pointBuf), find);
            cv::circle(rgb, cv::Point(pointBuf[0].x, pointBuf[0].y), 10, cv::Scalar(0, 0, 255), 10);

            char buff[MAXBYTE];
            sprintf_s(buff, "./Chessboard_%04d.bmp", index);
            cv::imwrite(buff, rgb);

            std::cout << "Index = " << index << ", Num = " << pointBuf.size() << std::endl;
        }
    }
    return pointBuf;
}
```

#### 4、相机标定   世界坐标系原点位于标定板左上角(第一个方格的左上角)

```c++
// 摄像机内参数矩阵 M=[fx γ u0,0 fy v0,0 0 1]
cv::Mat cameraMatrix = cv::Mat(3, 3, CV_32FC1, cv::Scalar::all(0));
// 摄像机的5个畸变系数：k1,k2,p1,p2,k3
cv::Mat distCoeffs = cv::Mat(1, 5, CV_32FC1, cv::Scalar::all(0));
// 每幅图像的旋转向量
std::vector<cv::Mat> rotationMat;
// 每幅图像的平移向量
std::vector<cv::Mat> translationMat;

// 标定
cv::calibrateCamera(objectPoints, imagePoints, imageSize, cameraMatrix, distCoeffs, rotationMat, translationMat);

// 保存标定结果的文件
std::ofstream fout("caliberation_result.txt");
fout << "每幅图像的标定误差：" << std::endl;

double err_sum = 0.0;

// 评估标定误差
for (int i = 0; i < objectPoints.size(); i++)
{
    // 保存重新计算得到的投影点
    std::vector<cv::Point2f> newImagePoint;
    cv::projectPoints(objectPoints[i], rotationMat[i], translationMat[i], cameraMatrix, distCoeffs, newImagePoint);

    // 计算新的投影点和旧投影点之间的误差
    cv::Mat oldImagePointMat = cv::Mat(imagePoints[i]);
    cv::Mat newImagePointMat = cv::Mat(newImagePoint);

    double err = cv::norm(newImagePointMat, oldImagePointMat, cv::NORM_L2) / newImagePoint.size();
    err_sum += err;

    fout << "第" << (i + 1) << "幅图像的平均误差：" << err << "像素" << std::endl;
}
fout << "总体平均误差：" << err_sum / objectPoints.size() << "像素" << std::endl;

fout << std::endl << "相机内参数矩阵：" << std::endl;
fout << cameraMatrix << std::endl;
fout << "畸变系数：" << std::endl;
fout << distCoeffs << std::endl << std::endl;
```

#### 5、标定结果

```apl
每幅图像的标定误差：
第1幅图像的平均误差：0.0288592像素
第2幅图像的平均误差：0.0288012像素
第3幅图像的平均误差：0.0295449像素
第4幅图像的平均误差：0.0418704像素
第5幅图像的平均误差：0.0262285像素
第6幅图像的平均误差：0.0307893像素
第7幅图像的平均误差：0.0203212像素
第8幅图像的平均误差：0.0365108像素
第9幅图像的平均误差：0.0329124像素
第10幅图像的平均误差：0.0275175像素
第11幅图像的平均误差：0.0333364像素
第12幅图像的平均误差：0.0295045像素
第13幅图像的平均误差：0.0383969像素
第14幅图像的平均误差：0.0343613像素
第15幅图像的平均误差：0.0458408像素
总体平均误差：0.0323197像素

相机内参数矩阵：
[2847.126463793122, 0, 1827.373583263593;
0, 2847.966452019607, 1345.232882836906;
0, 0, 1]
畸变系数：
[0.02941333344336914, -0.09318478673057608, -0.0004518282012472091, 2.88176623334251e-05, 0.2391830557850373]
```

#### 6、完整代码

```c++
int main()
{
    #if (defined(_DEBUG) && defined(CV))
    cv::utils::logging::setLogLevel(cv::utils::logging::LOG_LEVEL_SILENT);
    #endif

    // 保存检测到的所有角点
    std::vector<std::vector<cv::Point2f>> imagePoints;
    // 保存标定板上角点的三维坐标
    std::vector<std::vector<cv::Point3f>> objectPoints;

    // 图像尺寸
    cv::Size imageSize;

    // 标定板上每行每列的角点数
    cv::Size patternSize = cv::Size(13, 9);

    // 生成3D点坐标
    std::vector<cv::Point3f> singlePatternPoints;
    Init3DPoint(patternSize, singlePatternPoints);

    // 获取图片信息
    std::vector<std::string> file_paths;
    FileHelper::GetInstance().GetFormatFile("./Chessboard", file_paths, "jpg");

    for (size_t i = 0; i < file_paths.size(); i++)
    {
        cv::Mat img = cv::imread(file_paths[i], cv::IMREAD_GRAYSCALE);
        imageSize = img.size();
        auto corners = GetChessboardCorners(img, patternSize, i + 1);
        if (corners.size() > 0)
        {
            imagePoints.push_back(corners);
            objectPoints.push_back(singlePatternPoints);
        }
    }

    // 摄像机内参数矩阵 M=[fx γ u0,0 fy v0,0 0 1]
    cv::Mat cameraMatrix = cv::Mat(3, 3, CV_32FC1, cv::Scalar::all(0));
    // 摄像机的5个畸变系数：k1,k2,p1,p2,k3
    cv::Mat distCoeffs = cv::Mat(1, 5, CV_32FC1, cv::Scalar::all(0));
    // 每幅图像的旋转向量
    std::vector<cv::Mat> rotationMat;
    // 每幅图像的平移向量
    std::vector<cv::Mat> translationMat;

    // 标定
    cv::calibrateCamera(objectPoints, imagePoints, imageSize, cameraMatrix, distCoeffs, rotationMat, translationMat);

    // 保存标定结果的文件
    std::ofstream fout("caliberation_result.txt");
    fout << "每幅图像的标定误差：" << std::endl;

    double err_sum = 0.0;

    // 评估标定误差
    for (int i = 0; i < objectPoints.size(); i++)
    {
        // 保存重新计算得到的投影点
        std::vector<cv::Point2f> newImagePoint;
        cv::projectPoints(objectPoints[i], rotationMat[i], translationMat[i], cameraMatrix, distCoeffs, newImagePoint);

        // 计算新的投影点和旧投影点之间的误差
        cv::Mat oldImagePointMat = cv::Mat(imagePoints[i]);
        cv::Mat newImagePointMat = cv::Mat(newImagePoint);

        double err = cv::norm(newImagePointMat, oldImagePointMat, cv::NORM_L2) / newImagePoint.size();
        err_sum += err;

        fout << "第" << (i + 1) << "幅图像的平均误差：" << err << "像素" << std::endl;
    }
    fout << "总体平均误差：" << err_sum / objectPoints.size() << "像素" << std::endl;

    fout << std::endl << "相机内参数矩阵：" << std::endl;
    fout << cameraMatrix << std::endl;
    fout << "畸变系数：" << std::endl;
    fout << distCoeffs << std::endl << std::endl;
}
```

#### 生成棋盘格代码

```c++
void CreateChessboard(int length = 60/*棋盘格大小，像素为单位*/, int dx = 7/*棋盘格水平方向数目*/, int dy = 5/*棋盘格垂直方向数目*/)
{
    int count = 1;//边界多余dx大小为边界区域
    cv::Mat chessboardImage(cv::Size(2 * length * (dx + count), 2 * length * (dy + count)), CV_8UC1, cv::Scalar::all(200));

    for (int h = count; h < 2 * dy + count; h++)
    {
        for (int w = count; w < 2 * dx + count; w++)
        {
            for (int m = h * length; m < (h + 1) * length; m++)
            {
                for (int n = w * length; n < (w + 1) * length; n++)
                {
                    chessboardImage.at<uchar>(m, n) = ((h + w) % 2 == 0) ? 255 : 0;
                }
            }
        }
    }
    cv::imwrite("./Chessboard.bmp", chessboardImage);
}
```

#### 摄像头拍摄图片代码

```c++
void create_images()
{
    cv::Mat frame;
    cv::VideoCapture capture(0);
    int index = 1;
    while (true)
    {
        bool ret = capture.read(frame);
        flip(frame, frame, 1);
        if (!ret) break;
        imshow("frame", frame);
        char c = cv::waitKey(50);
        printf("%d ", c);
        if (c == 113)
        {
            // Q
            imwrite(cv::format("D:/%d.png", index), frame);
            index += 1;
        }
        if (c == 27)
        {
            break; // ESC
        }
    }
    capture.release();
}
```
