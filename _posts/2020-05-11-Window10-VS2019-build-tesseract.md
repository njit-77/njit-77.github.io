---
layout:     post
title:      Window10 VS2019配置tesseract源码
category: 	blog
---

1、配置tessract源码，折腾了好久[2020.5.5-2020.5.11]，今晚终于配置成功了。

### 成功

```c
1、从github下克隆Tesseract-OCR_for_Windows，注意是克隆不是下载
 
    git clone https://github.com/peirick/Tesseract-OCR_for_Windows.git

2、 从github下克隆leptonica
    
    git clone https://github.com/DanBloomberg/leptonica.git

3、 从github下克隆tesseract
    
    git clone https://github.com/tesseract-ocr/tesseract.git

4、把步骤2克隆完成的leptonica文件夹里面文件拷到Tesseract-OCR_for_Windows\leptonica

5、把步骤3克隆完成的tesseract文件夹里面文件拷到Tesseract-OCR_for_Windows\tesseract_3.05
    
6、tesseract.sln，编译，会有报错。

7、把
static const STRING kCharsToEx[] = {"'", "`", "\"", "\\", ",", ".",
	"〈", "〉", "《", "》", "」", "「", ""};

修改成
    
static const STRING kCharsToEx[] = { "'", "`", "\"", "\\", ",", ".",
	"<", ">", "<<", ">>", "" };    

8、再次编译运行，成功。
    
如果需要dll，打开项目属性，在常规->配置类型处选择动态库;C/C++预处理 -> 预处理器定义处增加TESS_EXPORTS

```

### 方法[但是不知道怎么用到vs工程里面]

```
sw setup
sw build org.sw.demo.google.tesseract.tesseract-master
```

### 失败

```
cppan --build pvt.cppan.demo.google.tesseract.tesseract-master
```

### 失败

```
C:\WINDOWS\system32>sw setup
Downloading database from origin remote

C:\WINDOWS\system32>d:

D:\>cd D:\Software\Programming Software\Open Source\tesseract\sw\tesseract-4.1.1

D:\Software\Programming Software\Open Source\tesseract\sw\tesseract-4.1.1>mkdir win64 && cd win64

D:\Software\Programming Software\Open Source\tesseract\sw\tesseract-4.1.1\win64>cmake .. -G "Visual Studio 16 2019"

```

还有其它尝试就不列出了。