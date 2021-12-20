---
layout:     post
title:      Window10 CMake 配置OpenCV源码
category: 	blog
---

0、下载opencv-4.2.0-vc14_vc15.exe 和 opencv_contrib-4.2.0.zip 并解压

1、打开CMake gui软件，配置路径

![cmake gui](/images/window10-cmake-opencv4.2.0/1.PNG)

2、点击 Configure，设置如下

![configure](/images/window10-cmake-opencv4.2.0/2.PNG)

3、CMake界面显示红色，

![cmake](/images/window10-cmake-opencv4.2.0/3.PNG)

再次点击Configure，直至CMake界面显示白色

![cmake](/images/window10-cmake-opencv4.2.0/4.PNG)

4、CMake界面找到 OPENCV_EXTRA_MODULES_PATH，设置opencv_contrib-4.2.0所在目录

![OPENCV_EXTRA_MODULES_PATH](/images/window10-cmake-opencv4.2.0/5.PNG)

再次点击Configure,报错。查看提示信息,发现一些文件无法下载。

直接在这个[网页里](https://github.com/opencv/opencv_contrib/issues/1301 )搜索 BenbenIO 这个用户的回答

或者执行以下操作

```c
1、下载https://raw.githubusercontent.com/opencv/opencv_3rdparty/32e315a5b106a7b89dbed51c28f8120a48b368b4/ippicv/ippicv_2019_win_intel64_20180723_general.zip    

2、修改名称为 1d222685246896fe089f88b8858e4b2f-ippicv_2019_win_intel64_20180723_general.zip   

3、存放到目录  D:\Software\Programming Software\OpenCV\opencv 4.2.0\opencv\sources\.cache\ippicv
```

```c
1、下载https://raw.githubusercontent.com/opencv/opencv_3rdparty/a66a24e9f410ae05da4baeeb8b451912664ce49c/ffmpeg/opencv_videoio_ffmpeg.dll   

2、修改名称为  5de6044cad9398549e57bc46fc13908d-opencv_videoio_ffmpeg.dll

3、存放到目录  D:\Software\Programming Software\OpenCV\opencv 4.2.0\opencv\sources\.cache\ffmpeg
```

```c
1、下载https://raw.githubusercontent.com/opencv/opencv_3rdparty/a66a24e9f410ae05da4baeeb8b451912664ce49c/ffmpeg/opencv_videoio_ffmpeg_64.dll   

2、修改名称为  5de6044cad9398549e57bc46fc13908d-opencv_videoio_ffmpeg.dll

3、存放到目录  D:\Software\Programming Software\OpenCV\opencv 4.2.0\opencv\sources\.cache\ffmpeg
```

```c
1、下载https://raw.githubusercontent.com/opencv/opencv_3rdparty/a66a24e9f410ae05da4baeeb8b451912664ce49c/ffmpeg/ffmpeg_version.cmake   

2、修改名称为  5de6044cad9398549e57bc46fc13908d-opencv_videoio_ffmpeg.dll

3、存放到目录  D:\Software\Programming Software\OpenCV\opencv 4.2.0\opencv\sources\.cache\ffmpeg
```

```c
1、下载https://raw.githubusercontent.com/opencv/opencv_3rdparty/a66a24e9f410ae05da4baeeb8b451912664ce49c/ffmpeg/ffmpeg_version.cmake 

2、修改名称为  ad57c038ba34b868277ccbe6dd0f9602-ffmpeg_version.cmake

3、存放到目录  D:\Software\Programming Software\OpenCV\opencv 4.2.0\opencv\sources\.cache\ffmpeg
```

```c
1、下载https://raw.githubusercontent.com/opencv/opencv_3rdparty/34e4206aef44d50e6bbcd0ab06354b52e7466d26/boostdesc_bgm.i

2、修改名称为  0ea90e7a8f3f7876d450e4149c97c74f-boostdesc_bgm.i

3、存放到目录  D:\Software\Programming Software\OpenCV\opencv 4.2.0\opencv\sources\.cache\xfeatures2d\boostdesc
```

```c
1、下载
https://raw.githubusercontent.com/opencv/opencv_3rdparty/34e4206aef44d50e6bbcd0ab06354b52e7466d26/boostdesc_bgm_bi.i

2、修改名称为  34e4206aef44d50e6bbcd0ab06354b52e7466d26-boostdesc_bgm_bi.i

3、存放到目录  D:\Software\Programming Software\OpenCV\opencv 4.2.0\opencv\sources\.cache\xfeatures2d\boostdesc
```

```c
1、下载
https://raw.githubusercontent.com/opencv/opencv_3rdparty/34e4206aef44d50e6bbcd0ab06354b52e7466d26/boostdesc_bgm_hd.i

2、修改名称为  34e4206aef44d50e6bbcd0ab06354b52e7466d26-boostdesc_bgm_hd.i

3、存放到目录  D:\Software\Programming Software\OpenCV\opencv 4.2.0\opencv\sources\.cache\xfeatures2d\boostdesc
```

```c
1、下载https://raw.githubusercontent.com/opencv/opencv_3rdparty/34e4206aef44d50e6bbcd0ab06354b52e7466d26/boostdesc_binboost_064.i

2、修改名称为  34e4206aef44d50e6bbcd0ab06354b52e7466d26-boostdesc_binboost_064.i

3、存放到目录  D:\Software\Programming Software\OpenCV\opencv 4.2.0\opencv\sources\.cache\xfeatures2d\boostdesc
```

```c
1、下载https://raw.githubusercontent.com/opencv/opencv_3rdparty/34e4206aef44d50e6bbcd0ab06354b52e7466d26/boostdesc_binboost_128.i

2、修改名称为  34e4206aef44d50e6bbcd0ab06354b52e7466d26-boostdesc_binboost_128.i

3、存放到目录  D:\Software\Programming Software\OpenCV\opencv 4.2.0\opencv\sources\.cache\xfeatures2d\boostdesc
```

```c
1、下载https://raw.githubusercontent.com/opencv/opencv_3rdparty/34e4206aef44d50e6bbcd0ab06354b52e7466d26/boostdesc_binboost_256.i

2、修改名称为  34e4206aef44d50e6bbcd0ab06354b52e7466d26-boostdesc_binboost_256.i

3、存放到目录  D:\Software\Programming Software\OpenCV\opencv 4.2.0\opencv\sources\.cache\xfeatures2d\boostdesc
```

```c
1、下载https://raw.githubusercontent.com/opencv/opencv_3rdparty/34e4206aef44d50e6bbcd0ab06354b52e7466d26/boostdesc_lbgm.i

2、修改名称为  34e4206aef44d50e6bbcd0ab06354b52e7466d26-boostdesc_lbgm.i

3、存放到目录  D:\Software\Programming Software\OpenCV\opencv 4.2.0\opencv\sources\.cache\xfeatures2d\boostdesc
```

```c
1、下载https://raw.githubusercontent.com/opencv/opencv_3rdparty/fccf7cd6a4b12079f73bbfb21745f9babcd4eb1d/vgg_generated_48.i

2、修改名称为  fccf7cd6a4b12079f73bbfb21745f9babcd4eb1d-vgg_generated_48.i

3、存放到目录  D:\Software\Programming Software\OpenCV\opencv 4.2.0\opencv\sources\.cache\xfeatures2d\vgg
```

```c
1、下载https://raw.githubusercontent.com/opencv/opencv_3rdparty/fccf7cd6a4b12079f73bbfb21745f9babcd4eb1d/vgg_generated_64.i

2、修改名称为  fccf7cd6a4b12079f73bbfb21745f9babcd4eb1d-vgg_generated_64.i

3、存放到目录  D:\Software\Programming Software\OpenCV\opencv 4.2.0\opencv\sources\.cache\xfeatures2d\vgg
```

```c
1、下载https://raw.githubusercontent.com/opencv/opencv_3rdparty/fccf7cd6a4b12079f73bbfb21745f9babcd4eb1d/vgg_generated_80.i

2、修改名称为  fccf7cd6a4b12079f73bbfb21745f9babcd4eb1d-vgg_generated_80.i

3、存放到目录  D:\Software\Programming Software\OpenCV\opencv 4.2.0\opencv\sources\.cache\xfeatures2d\vgg
```

```c
1、下载https://raw.githubusercontent.com/opencv/opencv_3rdparty/fccf7cd6a4b12079f73bbfb21745f9babcd4eb1d/vgg_generated_120.i

2、修改名称为  fccf7cd6a4b12079f73bbfb21745f9babcd4eb1d-vgg_generated_120.i

3、存放到目录  D:\Software\Programming Software\OpenCV\opencv 4.2.0\opencv\sources\.cache\xfeatures2d\vgg
```

```c
1、下载https://raw.githubusercontent.com/opencv/opencv_3rdparty/8afa57abc8229d611c4937165d20e2a2d9fc5a12/face_landmark_model.dat

2、修改名称为  8afa57abc8229d611c4937165d20e2a2d9fc5a12-face_landmark_model.dat

3、存放到目录  D:\Software\Programming Software\OpenCV\opencv 4.2.0\opencv\sources\.cache\data

```

再次点击Configure，没有报错。点击Generate。

打开OpenCV.sln，选择CMakeTargets里面INSTALL， 生成。

![install](/images/window10-cmake-opencv4.2.0/6.PNG)