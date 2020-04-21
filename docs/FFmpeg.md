# FFmpeg的简单使用

## 发送H.264裸流至RTMP协议的流媒体服务器

```bash
ffmpeg -re -i test.h264 -vcodec copy -f flv rtmp://localhost/application/livestream
```
## 参数注解:  

> `-re` 代表按照帧率发送，负责会按照最高的效率发送;
> `-i filename`  指定输入文件
> `-vcodec codec` 等价于`-codec:V codec`、`-c:v codec`, 如果使用`copy`表示完全拷贝源编解码
> `-f fmt`: 视频封装格式

## 通用参数

> `-y` 覆盖输出文件，为输入参数  
> `-t duration` 输出视频的总时长,以秒为单位  
> `-ss position` 输入视频的起始时间，以秒为单位  

## 视频参数

> `-b bitrate` 默认200kb/s 码率(每秒传送的二进制数据量)越高清晰度越好，同时文件体积增大  
> `-r fps` 帧频，默认25，影响画面流畅度  
> `-s size` 帧大小，视频的画面分辨率，默认160X128, `Sqcif:128x96`,`qcif:176x144`,`cif:252x288`,`4cif:704x576`  
> `aspect aspect` 画面比例,`4:3`,`16:9`等
> `-vn` 不对视频处理  

## 从视频中分离音频流
下面这行命令,可以用来只导出视频中的音频内容.
```bash
ffmpeg -i input_file -acode copy -vn out_file_audio
```

> `-maxrate bitrate` 最大码率  
> `-minrate bitrate` 最小码率

## 音频参数

> `-acode codec` 音频编码
> `-ab bitrate` 音频码率  
> `ar freq` 音频采样率  
> `-an` 不处理音频

## 从视频中分离视频流

利用下面这行命令,可以很方便地制作无声视频.
```bash
ffmpeg -i input_file -vocde copy -an out_file_video
```

## 视音频捕获选项

> `-vd device` 视频捕获设备
> `-ad device` 音频捕获设备
