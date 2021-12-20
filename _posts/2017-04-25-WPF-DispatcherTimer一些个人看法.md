---
layout:     post
title:      WPF  DispatcherTimer一些个人看法
category: 	blog
---

## WPF  DispatcherTimer一些个人看法

wpf中的DispatcherTimer基本用法，本文不在叙述。主要写一些不同的，来提醒自己不要再犯同样错误。



前几天写代码时发现。当在非UI线程创建DispatcherTimer实例时，程序无法进入Tick事件

```c#
private static System.Windows.Threading.DispatcherTimer timer;

private void Button_Click(object sender, RoutedEventArgs e)
{
    new System.Threading.Thread(CreateTimer).Start();
}

private void CreateTimer()
{
    timer = new System.Windows.Threading.DispatcherTimer();
    timer.Interval = TimeSpan.FromSeconds(1);
    timer.Tick += DispatcherTimer_Click;
    timer.Start();
}

private void DispatcherTimer_Click(object sender, EventArgs e)
{
    Console.WriteLine("DispatcherTimer_Click");
}
```

在DispatcherTimer_Click函数入口设断点，发现程序无法进入。

![](https://img2018.cnblogs.com/blog/1773352/201909/1773352-20190905183659513-729598169.png)





如果这样创建对象

```c#
private static System.Windows.Threading.DispatcherTimer timer;

private void Button_Click(object sender, RoutedEventArgs e)
{
    new System.Threading.Thread(CreateTimer).Start();
}

private void CreateTimer()
{
    timer = new System.Windows.Threading.DispatcherTimer(System.Windows.Threading.DispatcherPriority.SystemIdle, this.Dispatcher);
    timer.Interval = TimeSpan.FromSeconds(1);
    timer.Tick += DispatcherTimer_Click;
    timer.Start();
}

private void DispatcherTimer_Click(object sender, EventArgs e)
{
    Console.WriteLine("DispatcherTimer_Click");
}
```

程序可以进入Tick事件。

![](https://img2018.cnblogs.com/blog/1773352/201909/1773352-20190905183715352-1329255005.png)



或者这样创建对象

```c#
private static System.Windows.Threading.DispatcherTimer timer;

private void Button_Click(object sender, RoutedEventArgs e)
{
    new System.Threading.Thread(CreateTimer).Start();
}

private void CreateTimer()
{
    this.Dispatcher.Invoke(() => 
    {
        timer = new System.Windows.Threading.DispatcherTimer();
    });
    timer.Interval = TimeSpan.FromSeconds(1);
    timer.Tick += DispatcherTimer_Click;
    timer.Start();
}

private void DispatcherTimer_Click(object sender, EventArgs e)
{
    Console.WriteLine("DispatcherTimer_Click");
}
```

原因如下

DispatcherTimer.Tick 集成到按指定时间间隔和指定优先级处理的 Dispatcher 队列中的计时器。

在线程中创建DispatcherTimer对象时，DispatcherTimer的Dispatcher是线程的Dispatcher。

而此时如果线程如果没有操作UI对象，则其Dispatcher==null，[详情见博客](https://www.cnblogs.com/DoNetCoder/p/4369903.html)