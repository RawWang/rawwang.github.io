# 使用FFmpeg推流

## 特别声明

有关`FFmpeg`的学习，主要参考[大神-雷晓骅](http://blog.csdn.net/leixiaohua1020)。只恨天妒英才，逝者安息!

## 简介

这篇在`Windows`平台下调用`ffmpeg.exe`可执行文件,然后采用`RTMP`协议将本地文件转化为流媒体推送至本地搭建的`nginx-rtmp`服务器。

> [FFmpeg](https://ffmpeg.zeranoe.com/builds/)。   
> [nginx-rtmp](https://github.com/illuspas/nginx-rtmp-win32)。  

## 常用命令

使用FFmpeg通常也可以帮助我们将视频转成自己需要的格式,真的非常方便!
```bash
ffmpeg [options] [[infile options] -i infile]... {[outfile options] outfile}...
```
> 通用选项 输入文件选项 输入文件 输入文件... 输出文件选项 输出文件 输出文件...


## 推流

将本地文件转换成流推送至流媒体服务器
```bash
ffmpeg -re -i localFile.mp4 -c copy -f flv rtmp://server/live/streamName  
```

## 核心代码
```C#
ProcessStartInfo info = new ProcessStartInfo(FFmpegFilePath);
info.Arguments = parameters;   //运行参数
info.UseShellExecute = false;
info.CreateNoWindow = true;
info.RedirectStandardError = true;
try
{
  Process ffmpegProcess = Process.Start(info);
  //监听输出回调
  ffmpegProcess.ErrorDataReceived += new DataReceivedEventHandler(Output);
  //监听退出回调
  ffmpegProcess.Exited += new EventHandler(OnExit);
  ffmpegProcess.EnableRaisingEvents = true;
  //异步执行
  ffmpegProcess.BeginErrorReadLine();
}
catch (System.Exception ex)
{
  throw new ArgumentException(ex.Message);
}

//处理输出
void Output(object send, DataReceivedEventArgs output)
{
  if (!String.IsNullOrEmpty(output.Data))
  {
    UnityEngine.Debug.Log("> FFMPEG Log <: " + output.Data);
  }
}

//处理退出
static void OnExit(object send, EventArgs output)
{
  Process process = (Process)send;
  UnityEngine.Debug.Log("code" + process.ExitCode);
}
```
