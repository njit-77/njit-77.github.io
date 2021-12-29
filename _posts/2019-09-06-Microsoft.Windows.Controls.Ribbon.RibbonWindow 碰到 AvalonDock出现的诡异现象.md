---
layout:     post
title:      Microsoft.Windows.Controls.Ribbon.RibbonWindow 碰到 AvalonDock出现的诡异现象
category: 	blog
---

### 部分一

14年底进入目前公司时，领导准备开发一款新软件平台以取代原有平台。原平台采用C++Build开发界面(window c/s客户端) 、Visual Studio(封装dll模块)。过完年，领导已把框架搭建完毕(过年期间领导加班了 ^_^ )。当时菜鸟一个(目前老鸟了)，新框架用wpf模式，RibbonWindow(界面功能按钮) + AvalonDock(界面布局，灵活好用)，工具当然是最新的Visual Studio 2013了。虽然使用wpf开发，但是上位机软件客户不关心UI美观与否，只看功能是否强大。也因此本人只是把checkbox，Text等常用控件按照网上资料封装供自己调用，自定义控件方向没再继续往下研究。用的比较多的是用MVVM模式开发一些业务逻辑。绑定真的好用，View与ViewModel之间数据传输不需要在编写代码了。需求改变时，工作量也大大减轻了，从此步入新社会了。废话不多说，进入正题

16年中测试人员反馈，软件莫名其妙情况下，界面上一些输入控件无法响应键盘输入。接到反馈后，我们立即排查分析。
1、怀疑键盘消息没有传递到软件，但是现象出现时一部分控件可以输入，一部分无法输入，分分钟立刻打脸；
2、因无法输入的控件不是wpf本身提供控件，是一些mfc封装的窗口中的控件或者引用的winform控件如DataGridView，此时怀疑兼容性原因，看了[微软官方资料](https://docs.microsoft.com/zh-cn/dotnet/framework/wpf/advanced/wpf-and-windows-forms-interoperation)，在窗口前添加

```c#
System.Windows.Forms.Integration.ElementHost.EnableModelessKeyboardInterop(this)
```

还是无法解决问题。最终因能力不够只能临时搁浅此问题，采用临时方案，增加一个键盘输入窗口，当问题出现时，调用键盘输入窗口输入信息。

随着时间流失，我从小菜鸟变成了老菜鸟，但是该问题一直徘徊在我脑海中，无情嘲笑我，菜逼！！！



### 部分二

18年4月份，我重新回头查看此问题，主要看一下[开源源码](https://referencesource.microsoft.com/) 以期能解决问题。无意中我发现了造成此现象的原因。

1、点击RibbonSplitButton按键一次，下拉菜单弹出，再点击一次下拉菜单收回。此时无法输入信息

2、点击RibbonSplitButton按键一次，下拉菜单弹出，再点击其它地方。此时无法输入信息

3、点击RibbonSplitButton按键一次。此时可以输入信息。

请看下图，操作步骤按照1、2、3进行

![](/images/Microsoft.Windows.Controls.Ribbon.RibbonWindow/1.gif)



能重现问题，顿时感觉希望就在眼前。难道是RibbonSplitButton控件造成的，替换成RibbonMenuButton，结果还是一样。难不成重写一个控件？此方案放到最后吧，再想想其它思路。新在公司测试代码没法考出来，周末在自己电脑重新新建了一个工程。然而不管怎么点击，一直可以输入。难不成是我们平台其它bug造成的？天啊，平台开发3年多了，怎么去找问题，一个一个模块排除吗？终于，苦心人天不负，付出是有回报的。我发现自己新建工程中的RibbonWindow与公司不是同一个。公司引用**Microsoft.Windows.Controls.Ribbon.RibbonWindow**，而我引用**System.Windows.Controls.Ribbon.RibbonWindow**。

​	

### 部分三

	胜利就在眼前了，赶紧重新新建两个工程，分别引用**Microsoft.Windows.Controls.Ribbon.RibbonWindow**和**System.Windows.Controls.Ribbon.RibbonWindow**，其余代码不变。结果引用**Microsoft.Windows.Controls.Ribbon.RibbonWindow**出现键盘无法输入现象，另一个正常。



### 笔记

该问题只能说临时解决了，**Microsoft.Windows.Controls.Ribbon.RibbonWindow**库出现原因不清楚。也没在继续跟踪下去。
