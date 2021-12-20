---
layout:     post
title:      vs创建qt dll，并使用qt控制台测试
category: 	blog
---

### 创建qt dll项目

1、vs新建，创建Qt Class Library

![1](/images/vs创建qt-dll并使用qt控制台测试/1.PNG)

#### 编写界面

1、Qt Creator创建 一个Main Window文件，界面编写如下：

![2](/images/vs创建qt-dll并使用qt控制台测试/2.PNG)

2、编译项目，添加MainWindow.h/.cpp文件、ui_mainwindow.h文件添加到vs项目下；

![3](/images/vs创建qt-dll并使用qt控制台测试/3.PNG)

#### 配置环境

1、

```c
右键属性 -> 常规 ->输出目录 设置  $(SolutionDir)$(Platform)\$(Configuration)\

右键属性 -> 常规 ->中间目录 设置  $(SolutionDir)$(Platform)\$(Configuration)\$(ProjectName).Dir\
```

2、

```c
右键属性 -> Qt Project Settings -> Qt Modules 添加  gui;widgets
```

![4](/images/vs创建qt-dll并使用qt控制台测试/4.PNG)

#### 生成

1、生成项目，无报错。

![5](/images/vs创建qt-dll并使用qt控制台测试/5.PNG)

### 创建 Qt 控制台程序

![6](/images/vs创建qt-dll并使用qt控制台测试/6.PNG)

#### 配置环境

1、

```c
右键属性 -> 常规 ->输出目录 设置 $(SolutionDir)$(Platform)\$(Configuration)\

右键属性 -> 常规 ->中间目录 设置 $(SolutionDir)$(Platform)\$(Configuration)\$(ProjectName).Dir\
```

2、

```c
C/C++ -> 常规 -> 附加包含目录 添加 $(SolutionDir)$(SolutionName);
```

3、

```c
链接器 -> 常规 -> 附加库目录 添加 $(SolutionDir)$(Platform)\$(Configuration)\
```

4、

```c
链接器 -> 输入 -> 附加依赖项 添加  Qt5Cored.lib
							  Qt5Guid.lib
							  Qt5Widgetsd.lib
(此处配置Debug信息，Release去掉d后缀 eg：Qt5Core.lib、Qt5Gui.lib、Qt5Widgets.lib)
```

#### 编写界面

#### 代码设置

1、添加mainwindow.h，内容如下

```c++
#pragma once

#include "ui_mainwindow.h"

namespace Ui {
	class MainWindow;
}

class MainWindow : public QMainWindow
{
	Q_OBJECT

public:
	explicit MainWindow(QWidget *parent = 0);
	~MainWindow();

private:
	Ui::MainWindow *ui;
};
```

2、main.cpp文件内容修改如下

```c++
#include <QtCore/QCoreApplication>

#include "mainwindow.h"
#pragma comment(lib, "QtExercise7.lib")

int main(int argc, char *argv[])
{
	QApplication a(argc, argv);

	MainWindow *hehe = new MainWindow();
	hehe->show();

    return a.exec();
}
```

3、编译运行，效果图

![7](/images/vs创建qt-dll并使用qt控制台测试/7.PNG)