---
title: 基于BoW和SVM的图片分类
date: 2024-01-15 17:21
tags: 算法
---

> 原文发表在[公众号](https://mp.weixin.qq.com/s/U8B7SvD47ZtF0s3sWyxD8A)

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

![图片](/images/基于BoW和SVM的图片分类/1.png)

### 训练流程

#### 提取图片SIFT特征信息

```c++
void GetFeatureSet(std::string dir,
                   std::string file_format,
                   size_t train_count,
                   std::vector<SIFTFeature>& feature_set,
                   std::vector<int>& feature_labels,
                   bool is_train = true)
{
    std::vector<std::vector<std::string>> file_paths;
    GetFile(dir, file_format, file_paths);


    std::vector<std::string> image_paths;
    for (size_t i = 0; i < file_paths.size(); i++)
    {
        if (is_train)
        {
            for (size_t j = 0; j < std::min(file_paths[i].size(), train_count); j++)
            {
                image_paths.push_back(file_paths[i][j]);
                feature_labels.push_back(i);
            }
        }
        else
        {
            for (size_t j = std::min(file_paths[i].size(), train_count); j < std::max(file_paths[i].size(), train_count); j++)
            {
                image_paths.push_back(file_paths[i][j]);
                feature_labels.push_back(i);
            }
        }
    }


    cv::Ptr<cv::SIFT> sift = cv::SIFT::create();
    feature_set.resize(image_paths.size());
    parallel_for_(cv::Range(0, image_paths.size()),
                  Paraller_GetImageSIFTFeature(image_paths, sift, feature_set));
}
```

#### KMeans聚类获取特征词典

```c++
cv::Mat GetDic(std::vector<SIFTFeature> train_feature, int k, bool is_write = true)
{
    cv::Mat dic;
    if (is_write)
    {
        cv::BOWKMeansTrainer kmeans_trainer = cv::BOWKMeansTrainer(k);
        for (size_t i = 0; i < train_feature.size(); i++)
        {
            kmeans_trainer.add(train_feature[i].descriptors);
        }

        dic = kmeans_trainer.cluster();

        cv::FileStorage fs("./vocabulary - [k = " + std::to_string(k) + "].yml", cv::FileStorage::WRITE);
        fs << "dic" << dic;
        fs.release();
    }
    else
    {
        cv::FileStorage fs("./vocabulary - [k = " + std::to_string(k) + "].yml", cv::FileStorage::READ);
        fs["dic"] >> dic;
        fs.release();
    }
    return dic;
}
```

#### 统计词频描述

```c++
std::vector<cv::Mat> FeatureToSVMSet(cv::Mat dic,
                                     std::string bow_path,
                                     std::vector<SIFTFeature> feature_set)
{
    std::vector<cv::Mat> bow_list(feature_set.size());

    if (FileHelper::GetInstance().IsFileExists(bow_path))
    {
        cv::FileStorage fs(bow_path, cv::FileStorage::READ);
        fs["bow_list"] >> bow_list;
        fs.release();

        return bow_list;
    }


    parallel_for_(cv::Range(0, feature_set.size()),
                  Paraller_FeatureToSVMSet(dic, feature_set, bow_list));

    cv::FileStorage fs(bow_path, cv::FileStorage::WRITE);
    fs << "bow_list" << bow_list;
    fs.release();

    return bow_list;
}
```

#### 训练分类器

```c++
void SVMTrain(std::vector<cv::Mat>train_set, std::vector<int> train_labels)
{
    cv::Mat train_mat;
    cv::vconcat(train_set, train_mat);
    cv::Mat label(train_labels);

    cv::Ptr<cv::ml::SVM> svm = cv::ml::SVM::create();

    svm->setType(cv::ml::SVM::C_SVC);
    svm->setKernel(cv::ml::SVM::LINEAR);
    svm->setTermCriteria(cv::TermCriteria(cv::TermCriteria::MAX_ITER, 100, 1e-6));

    svm->train(train_mat, cv::ml::ROW_SAMPLE, label);

    svm->save("./train_data.yml");
}
```

### 分类流程

#### 提取图片SIFT特征信息

#### 统计词频描述

#### 预测分类

```c++
cv::Ptr<cv::ml::SVM> svm = SVMHelper::GetInstance().GetSVMTrain();
std::vector<int> predictions;
for (size_t i = 0; i < test_bow_list.size(); i++)
{
    predictions.push_back(svm->predict(test_bow_list[i]));
}
print_log("SVM Predict Done!");
```

#### 绘制混淆矩阵

```c++
auto calculateConfusionMatrix = [](std::vector<int> groundTruth, std::vector<int> predictions, int numClasses) -> std::vector<std::vector<int>>
{
    std::vector<std::vector<int>> confusionMatrix(numClasses, std::vector<int>(numClasses, 0));
    for (size_t i = 0; i < groundTruth.size(); i++)
    {
        int trueClass = groundTruth[i];
        int predictedClass = predictions[i];
        confusionMatrix[trueClass][predictedClass]++;
    }
    return confusionMatrix;
};

std::vector<std::vector<int>> confusionMatrix = calculateConfusionMatrix(test_label, predictions, 15);
{
    cv::FileStorage fs("./confusionMatrix.yml", cv::FileStorage::WRITE);
    fs << "confusionMatrix" << confusionMatrix;
    fs.release();
}
print_log("ConfusionMatrix Done!");
```

#### 结果

confusionMatrix:

- [ 28, 1, 6, 6, 7, 2, 1, 1, 2, 3, 0, 1, 3, 3, 2 ]
- [ 0, 81, 1, 0, 3, 0, 3, 0, 0, 1, 0, 1, 0, 1, 0 ]
- [ 11, 4, 56, 5, 10, 1, 1, 4, 14, 8, 1, 20, 9, 7, 10 ]
- [ 3, 0, 2, 34, 5, 0, 0, 0, 3, 0, 0, 2, 0, 9, 2 ]
- [ 32, 3, 10, 27, 44, 1, 0, 0, 3, 0, 0, 2, 2, 8, 7 ]
- [ 3, 0, 1, 0, 0, 154, 0, 15, 0, 14, 16, 1, 6, 0, 0 ]
- [ 0, 0, 0, 0, 0, 0, 168, 0, 0, 8, 1, 1, 0, 0, 0 ]
- [ 1, 2, 7, 1, 0, 24, 1, 59, 5, 1, 3, 3, 3, 0, 0 ]
- [ 3, 1, 18, 8, 0, 0, 0, 3, 87, 1, 1, 15, 6, 3, 12 ]
- [ 0, 0, 1, 0, 1, 15, 24, 2, 0, 167, 11, 2, 1, 0, 0 ]
- [ 2, 1, 4, 0, 0, 50, 29, 9, 0, 57, 105, 0, 2, 0, 1 ]
- [ 4, 0, 16, 1, 4, 4, 0, 3, 5, 5, 4, 80, 3, 0, 13 ]
- [ 5, 3, 16, 5, 4, 4, 1, 9, 16, 2, 7, 13, 117, 1, 3 ]
- [ 8, 1, 2, 13, 8, 0, 0, 0, 2, 0, 0, 1, 0, 30, 0 ]
- [ 1, 4, 14, 5, 5, 0, 3, 0, 5, 0, 1, 19, 4, 5, 99 ]



#### 参考

[计算机视觉（本科） 北京邮电大学 鲁鹏 清晰完整合集](https://www.bilibili.com/video/BV1nz4y197Qv/?p=11&spm_id_from=333.880.my_history.page.click&vd_source=a54b83482179cbc54c6008189dba1a49)



### 完整代码

#### SIFTFeature

```c++
#pragma once


#include<vector>


#include "pch.h"


class SIFTFeature
{
    public:
    cv::Mat image;

    std::vector<cv::KeyPoint> keypoints;

    cv::Mat descriptors;


    public:
    void Init(std::string file, cv::Ptr<cv::SIFT> sift)
    {
        image = cv::imread(file, cv::IMREAD_GRAYSCALE);

        sift->detectAndCompute(image, cv::noArray(), keypoints, descriptors);
    }
};
```

#### FileHelper

```c++
#pragma once


#include <string>
#include <vector>
#include <io.h>
#include <fstream>

class FileHelper
{
    public:
    void GetFormatFile(std::string filePath, std::vector<std::string>& files)
    {
        _finddata_t fileInfo;
        intptr_t hFile = _findfirst((filePath + "\\*").c_str(), &fileInfo);
        std::string p;
        if (hFile != -1)
        {
            do
            {
                if ((fileInfo.attrib & _A_SUBDIR))
                {
                    if (strcmp(fileInfo.name, ".") != 0 && strcmp(fileInfo.name, "..") != 0)
                    {
                        files.push_back(filePath + "\\" + fileInfo.name);
                    }
                }
            } while (_findnext(hFile, &fileInfo) == 0);
            _findclose(hFile);
        }
    }

    void GetFormatFile(std::string filePath, std::vector<std::string>& files, std::string format, bool search_subfolders = false)
    {
        _finddata_t fileInfo;
        intptr_t hFile = search_subfolders ? _findfirst((filePath + "\\*").c_str(), &fileInfo) : _findfirst((filePath + "\\*" + format).c_str(), &fileInfo);
        std::string p;
        if (hFile != -1)
        {
            do
            {
                if ((fileInfo.attrib & _A_SUBDIR))
                {
                    if (strcmp(fileInfo.name, ".") != 0 && strcmp(fileInfo.name, "..") != 0)
                    {
                        GetFormatFile(filePath + "\\" + fileInfo.name, files, format, search_subfolders);
                    }
                }
                else
                {
                    std::string filename = fileInfo.name;
                    if (filename.substr(filename.find_last_of('.') + 1, filename.length()) == format)
                    {
                        files.push_back(filePath + "\\" + fileInfo.name);
                    }
                }
            } while (_findnext(hFile, &fileInfo) == 0);
            _findclose(hFile);
        }
    }

    bool IsFileExists(std::string& name)
    {
        std::ifstream f(name.c_str());
        return f.good();
    }

    public:
    static FileHelper& GetInstance()
    {
        static FileHelper m_instance;
        return m_instance;
    }
};
```

#### GetSIFTFeature

```c++
#pragma once


#include<vector>


#include "pch.h"
#include "FileHelper.h"
#include "SIFTFeature.h"


class Paraller_GetImageSIFTFeature :public cv::ParallelLoopBody
{
    private:
    std::vector<std::string> file_path;
    cv::Ptr<cv::SIFT> sift;
    std::vector<SIFTFeature>& file_set;

    public:
    Paraller_GetImageSIFTFeature(std::vector<std::string> _file_path,
                                 cv::Ptr<cv::SIFT> _sift,
                                 std::vector<SIFTFeature>& _file_set)
        : file_path(_file_path),
    sift(_sift),
    file_set(_file_set)
    {

    }

    virtual void operator()(const cv::Range& r) const
    {
        for (int i = r.start; i != r.end; i++)
        {
            file_set[i].Init(file_path[i], sift);
        }
    }
};

class GetSIFTFeature
{
    private:
    void GetFile(std::string dir,
                 std::string file_format,
                 std::vector<std::vector<std::string>>& file_paths)
    {
        auto GetPhotoNo = [](std::string file_path, std::string file_format) -> int
        {
            int start_index = file_path.find_last_of('\\');
            int end_index = file_path.find_last_of(file_format);
            std::string str = file_path.substr(start_index + 1, end_index - (file_format.length()) - (start_index + 1));
            return std::stoi(str);
        };


        std::vector<std::string> dir_paths;
        FileHelper::GetInstance().GetFormatFile(dir, dir_paths);


        for (size_t i = 0; i < dir_paths.size(); i++)
        {
            std::vector<std::string> files;
            FileHelper::GetInstance().GetFormatFile(dir_paths[i], files, file_format);

            std::sort(files.begin(), files.end(), [&](std::string a, std::string b)
                      {
                          return GetPhotoNo(a, file_format) < GetPhotoNo(b, file_format);
                      });
            file_paths.push_back(files);
        }
    }

    public:
    void GetFeatureSet(std::string dir,
                       std::string file_format,
                       size_t train_count,
                       std::vector<SIFTFeature>& feature_set,
                       std::vector<int>& feature_labels,
                       bool is_train = true)
    {
        std::vector<std::vector<std::string>> file_paths;
        GetFile(dir, file_format, file_paths);


        std::vector<std::string> image_paths;
        for (size_t i = 0; i < file_paths.size(); i++)
        {
            if (is_train)
            {
                for (size_t j = 0; j < std::min(file_paths[i].size(), train_count); j++)
                {
                    image_paths.push_back(file_paths[i][j]);
                    feature_labels.push_back(i);
                }
            }
            else
            {
                for (size_t j = std::min(file_paths[i].size(), train_count); j < std::max(file_paths[i].size(), train_count); j++)
                {
                    image_paths.push_back(file_paths[i][j]);
                    feature_labels.push_back(i);
                }
            }
        }


        cv::Ptr<cv::SIFT> sift = cv::SIFT::create();
        feature_set.resize(image_paths.size());
        parallel_for_(cv::Range(0, image_paths.size()),
                      Paraller_GetImageSIFTFeature(image_paths, sift, feature_set));
    }

    public:
    static GetSIFTFeature& GetInstance()
    {
        static GetSIFTFeature m_instance;
        return m_instance;
    }
};
```

#### GetBow

```c++
#pragma once


#include<vector>


#include "pch.h"
#include "SIFTFeature.h"


class Paraller_FeatureToSVMSet :public cv::ParallelLoopBody
{
    private:
    cv::Mat& dic;
    std::vector<SIFTFeature>& feature_set;
    std::vector<cv::Mat>& bow_list;

    public:
    Paraller_FeatureToSVMSet(cv::Mat& _dic,
                             std::vector<SIFTFeature>& _feature_set,
                             std::vector<cv::Mat>& _bow_list)
        : dic(_dic),
    feature_set(_feature_set),
    bow_list(_bow_list)
    {

    }

    virtual void operator()(const cv::Range& r) const
    {
        for (int i = r.start; i != r.end; i++)
        {
            cv::Ptr<cv::SIFT> sift = cv::SIFT::create();
            cv::Ptr<cv::DescriptorMatcher> matcher = cv::DescriptorMatcher::create("FlannBased");
            cv::BOWImgDescriptorExtractor bow(sift, matcher);
            bow.setVocabulary(dic);

            bow.compute(feature_set[i].image, feature_set[i].keypoints, bow_list[i]);

            cv::normalize(bow_list[i], bow_list[i]);
        }
    }
};


class GetBow
{
    public:
    cv::Mat GetDic(std::vector<SIFTFeature> train_feature, int k, bool is_write = true)
    {
        cv::Mat dic;
        if (is_write)
        {
            cv::BOWKMeansTrainer kmeans_trainer = cv::BOWKMeansTrainer(k);
            for (size_t i = 0; i < train_feature.size(); i++)
            {
                kmeans_trainer.add(train_feature[i].descriptors);
            }

            dic = kmeans_trainer.cluster();

            cv::FileStorage fs("./vocabulary - [k = " + std::to_string(k) + "].yml", cv::FileStorage::WRITE);
            fs << "dic" << dic;
            fs.release();
        }
        else
        {
            cv::FileStorage fs("./vocabulary - [k = " + std::to_string(k) + "].yml", cv::FileStorage::READ);
            fs["dic"] >> dic;
            fs.release();
        }
        return dic;
    }


    std::vector<cv::Mat> FeatureToSVMSet(cv::Mat dic,
                                         std::string bow_path,
                                         std::vector<SIFTFeature> feature_set)
    {
        std::vector<cv::Mat> bow_list(feature_set.size());

        if (FileHelper::GetInstance().IsFileExists(bow_path))
        {
            cv::FileStorage fs(bow_path, cv::FileStorage::READ);
            fs["bow_list"] >> bow_list;
            fs.release();

            return bow_list;
        }

        parallel_for_(cv::Range(0, feature_set.size()),
                      Paraller_FeatureToSVMSet(dic, feature_set, bow_list));

        cv::FileStorage fs(bow_path, cv::FileStorage::WRITE);
        fs << "bow_list" << bow_list;
        fs.release();

        return bow_list;
    }

    public:
    static GetBow& GetInstance()
    {
        static GetBow m_instance;
        return m_instance;
    }
};
```

#### SVMHelper

```c++
#pragma once


#include "pch.h"


class SVMHelper
{
    public:
    void SVMTrain(std::vector<cv::Mat>train_set, std::vector<int> train_labels)
    {
        cv::Mat train_mat;
        cv::vconcat(train_set, train_mat);
        cv::Mat label(train_labels);

        cv::Ptr<cv::ml::SVM> svm = cv::ml::SVM::create();

        svm->setType(cv::ml::SVM::C_SVC);
        svm->setKernel(cv::ml::SVM::LINEAR);
        svm->setTermCriteria(cv::TermCriteria(cv::TermCriteria::MAX_ITER, 100, 1e-6));

        svm->train(train_mat, cv::ml::ROW_SAMPLE, label);

        svm->save("./train_data.yml");
    }

    cv::Ptr<cv::ml::SVM> GetSVMTrain()
    {
        cv::Ptr<cv::ml::SVM> svm = cv::Algorithm::load<cv::ml::SVM>("./train_data.yml");

        return svm;
    }

    public:
    static SVMHelper& GetInstance()
    {
        static SVMHelper m_instance;
        return m_instance;
    }
};
```

#### main

```c++
#include "GetSIFTFeature.h"
#include "GetBOW.h"
#include "SVMHelper.h"

#include <Windows.h>

void print_log(std::string log)
{
    SYSTEMTIME sys;
    GetLocalTime(&sys);
    char buff[MAXBYTE];
    memset(buff, 0, sizeof(buff) / sizeof(char));
    sprintf_s(buff, "[%04d-%02d-%02d %02d-%02d-%02d.%04d]",
              sys.wYear, sys.wMonth, sys.wDay,
              sys.wHour, sys.wMinute, sys.wSecond, sys.wMilliseconds);

    std::string msg;
    msg.append(buff).append(log);

    std::cout << msg << std::endl;
}

int main()
{
    const std::string path = "./15-Scene";
    const std::string file_format = "jpg";
    const size_t train_count = 150;

    print_log("Begin!");


    std::vector<SIFTFeature> train_feature;
    std::vector<int> train_label;
    GetSIFTFeature::GetInstance().GetFeatureSet(path,
                                                file_format,
                                                train_count,
                                                train_feature,
                                                train_label);
    print_log("GetTrainFeatureSet Done!");


    cv::Mat bow_dic = GetBow::GetInstance().GetDic(train_feature, 1000, false);
    print_log("GetDic Done!");


    std::vector<cv::Mat> train_bow_list = GetBow::GetInstance().FeatureToSVMSet(bow_dic,
                                                                                "./train_bow_list.yml",
                                                                                train_feature);
    print_log("TrainFeatureToSVMSet Done!");


    SVMHelper::GetInstance().SVMTrain(train_bow_list, train_label);
    print_log("SVMTrain Done!");








    auto calculateConfusionMatrix = [](std::vector<int> groundTruth, std::vector<int> predictions, int numClasses) -> std::vector<std::vector<int>>
    {
        std::vector<std::vector<int>> confusionMatrix(numClasses, std::vector<int>(numClasses, 0));
        for (size_t i = 0; i < groundTruth.size(); i++)
        {
            int trueClass = groundTruth[i];
            int predictedClass = predictions[i];
            confusionMatrix[trueClass][predictedClass]++;
        }
        return confusionMatrix;
    };

    std::vector<SIFTFeature> test_feature;
    std::vector<int> test_label;
    GetSIFTFeature::GetInstance().GetFeatureSet(path,
                                                file_format,
                                                train_count,
                                                test_feature,
                                                test_label,
                                                false);
    print_log("GetTestFeatureSet Done!");


    std::vector<cv::Mat> test_bow_list = GetBow::GetInstance().FeatureToSVMSet(bow_dic,
                                                                               "./test_bow_list.yml",
                                                                               test_feature);
    print_log("TestFeatureToSVMSet Done!");


    cv::Ptr<cv::ml::SVM> svm = SVMHelper::GetInstance().GetSVMTrain();
    std::vector<int> predictions;
    for (size_t i = 0; i < test_bow_list.size(); i++)
    {
        predictions.push_back(svm->predict(test_bow_list[i]));
    }
    print_log("SVM Predict Done!");

    std::vector<std::vector<int>> confusionMatrix = calculateConfusionMatrix(test_label, predictions, 15);
    {
        cv::FileStorage fs("./confusionMatrix.yml", cv::FileStorage::WRITE);
        fs << "confusionMatrix" << confusionMatrix;
        fs.release();
    }
    print_log("ConfusionMatrix Done!");
}
```
