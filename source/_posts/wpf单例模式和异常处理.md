---
title: wpf 单例模式和异常处理
date: 2017-04-12
categories: wpf
---

###   单例模式

```c#
#region Singletion App

/// <summary>
/// 必须定义此变量
/// </summary>
/// <remarks>
/// <para>
/// 当<see cref="EnsureAssemblySingletion()"/>函数内部定义局部Mutex时，如果先启动软件再调试运行，此时判断单例模式失效。
/// </para>
/// </remarks>
private System.Threading.Mutex mutex;

private const string AssemblyGUID = "Your New Guid";

private bool EnsureAssemblySingletion()
{
    mutex = new System.Threading.Mutex(
        true,
        $"{System.Reflection.Assembly.GetEntryAssembly().GetName().Name} - {AssemblyGUID}",
        out bool ret
    );
    if (ret)
    {
        mutex.ReleaseMutex();
    }
    return ret;
}

#endregion
```

### 异常处理

#### 注册异常处理函数

```c#
/// 设置UI线程发生异常时处理函数
System.Windows.Application.Current.DispatcherUnhandledException +=
    App_DispatcherUnhandledException;

/// 设置非UI线程发生异常时处理函数
AppDomain.CurrentDomain.UnhandledException += App_UnhandledException;

/// 设置托管代码异步线程发生异常时处理函数
TaskScheduler.UnobservedTaskException += App_UnobservedTaskException;

/// 设置非托管代码发生异常时处理函数
callBack = new Unhandled_CallBack(Unhandled_ExceptionFilter);
SetUnhandledExceptionFilter(callBack);
```

#### 异常处理函数

```c#
#region Try catch Exception

    private void App_DispatcherUnhandledException(
    object sender,
    System.Windows.Threading.DispatcherUnhandledExceptionEventArgs exception
)
{
    messageBox.ShowMessage(exception.Exception);

    exception.Handled = true;
}

private void App_UnhandledException(object sender, UnhandledExceptionEventArgs exception)
{
    messageBox.ShowMessage(
        exception
    );

    if (exception.IsTerminating)
    {
        messageBox.ShowMessage(
            "软件出现不可恢复错误，强制关闭软件。"
        );

        Environment.Exit(0);
    }
}

private void App_UnobservedTaskException(
    object sender,
    UnobservedTaskExceptionEventArgs exception
)
{
    if (!exception.Observed && exception.Exception != null)
    {
        foreach (var ex in exception.Exception.Flatten().InnerExceptions)
        {
            messageBox.ShowMessage(ex);
        }
        exception.SetObserved();
    }
}

[System.Runtime.InteropServices.DllImport("kernel32")]
private static extern Int32 SetUnhandledExceptionFilter(Unhandled_CallBack cb);

private delegate int Unhandled_CallBack(ref long a);

private Unhandled_CallBack callBack;

private int Unhandled_ExceptionFilter(ref long a)
{
    messageBox.Show(
        new Exception($"StackTrace = {Environment.StackTrace}")
    );

    return 1;
}

#endregion
```

### Dump

对于一些问题，有时候日志不能完全帮助我们找到问题所在，这时dmp文件就可以帮助到我们。

```c#
class DumpWriter
{
    public enum MiniDumpType
    {
        None = 0x00010000,
        Normal = 0x00000000,
        WithDataSegs = 0x00000001,
        WithFullMemory = 0x00000002,
        WithHandleData = 0x00000004,
        FilterMemory = 0x00000008,
        ScanMemory = 0x00000010,
        WithUnloadedModules = 0x00000020,
        WithIndirectlyReferencedMemory = 0x00000040,
        FilterModulePaths = 0x00000080,
        WithProcessThreadData = 0x00000100,
        WithPrivateReadWriteMemory = 0x00000200,
        WithoutOptionalData = 0x00000400,
        WithFullMemoryInfo = 0x00000800,
        WithThreadInfo = 0x00001000,
        WithCodeSegs = 0x00002000
    }

    [DllImport("DbgHelp.dll")]
    private static extern bool MiniDumpWriteDump(
        IntPtr hProcess,
        Int32 processId,
        IntPtr fileHandle,
        MiniDumpType dumpType,
        ref MiniDumpExceptionInformation excepInfo,
        IntPtr userInfo,
        IntPtr extInfo);

    [DllImport("DbgHelp.dll")]
    private static extern bool MiniDumpWriteDump(
        IntPtr hProcess,
        Int32 processId,
        IntPtr fileHandle,
        MiniDumpType dumpType,
        IntPtr excepParam,
        IntPtr userInfo,
        IntPtr extInfo);

    [StructLayout(LayoutKind.Sequential, Pack = 4)]// Pack=4 is important! So it works also for x64!
    private struct MiniDumpExceptionInformation
    {
        public uint ThreadId;
        public IntPtr ExceptionPointers;
        [MarshalAs(UnmanagedType.Bool)]
        public bool ClientPointers;
    }

    [DllImport("kernel32.dll")]
    private static extern uint GetCurrentThreadId();

    private bool WriteDump(String dmpPath, MiniDumpType dmpType)
    {
        using (FileStream stream = new FileStream(dmpPath, FileMode.Create))
        {
            //取得进程信息
            Process process = Process.GetCurrentProcess();

            MiniDumpExceptionInformation mei = new MiniDumpExceptionInformation();
            mei.ThreadId = GetCurrentThreadId();
            mei.ExceptionPointers = Marshal.GetExceptionPointers();
            mei.ClientPointers = true;

            bool res = false;

            //如果不使用MiniDumpWriteDump重载函数
            //当mei.ExceptioonPointers == IntPtr.Zero => 无法保存dmp文件
            //且当mei.ClientPointers == false时程序直接崩溃(mei.ClientPointers == true程序不崩溃)
            //
            //以上测试信息硬件环境 cpu Pentium(R) Dual-Core CPU T4200 @ 2.00GHz
            //                 vs2013update5
            //在公司服务器上测试(64位系统、vs2013)不会出现上述情况
            /*res = MiniDumpWriteDump(
                process.Handle,
                process.Id,
                stream.SafeFileHandle.DangerousGetHandle(),
                dmpType,
                ref mei,
                IntPtr.Zero,
                IntPtr.Zero);*/

            if (mei.ExceptionPointers == IntPtr.Zero)
            {
                res = MiniDumpWriteDump(
                    process.Handle,
                    process.Id,
                    stream.SafeFileHandle.DangerousGetHandle(),
                    dmpType,
                    IntPtr.Zero,
                    IntPtr.Zero,
                    IntPtr.Zero);
            }
            else
            {
                res = MiniDumpWriteDump(
                    process.Handle,
                    process.Id,
                    stream.SafeFileHandle.DangerousGetHandle(),
                    dmpType,
                    ref mei,
                    IntPtr.Zero,
                    IntPtr.Zero);
            }
            return res;
        }
    }

    public DumpWriter()
    {
        FilePath = Environment.CurrentDirectory + @"\Dump";
        if (!Directory.Exists(FilePath))
            Directory.CreateDirectory(FilePath);
    }

    /// <summary>
    /// 保存dmp文件路径
    /// </summary>
    public string FilePath { get; protected set; }
    /// <summary>
    /// 保存dmp文件名称(包括路径)
    /// </summary>
    public string FileName { get; protected set; }
    /// <summary>
    /// 写dmp文件
    /// </summary>
    /// <param name="dmpType">参数，不同参数保存内容不一样</param>
    /// <returns></returns>
    public bool WriteDumpFile(MiniDumpType dmpType)
    {
        FileName = string.Format("{0}\\{1}_{2}.dmp",
                                 FilePath,
                                 DateTime.Now.ToString("yyyy-MM-dd-HH-mm-ss-fff"),
                                 Process.GetCurrentProcess().ProcessName);
        return WriteDump(FileName, dmpType);
    }
}
```
