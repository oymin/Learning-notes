# FFmpeg 介绍

FFmpeg 是一套可以用来记录、转换数字音频、视频，并能将其转化为流的开源计算机程序。采用 LGPL 或 GPL 许可证。它提供了录制、转换以及流化音视频的完整解决方案。它包含了非常先进的音频/视频编解码库 libavcodec，为了保证高可移植性和编解码质量，libavcode c里很多 code 都是从头开发的。

FFmpeg 在 Linux 平台下开发，但它同样也可以在其它操作系统环境中编译运行，包括 Windows、Mac OS X 等。这个项目最早由 Fabrice Bellard 发起，2004年至2015年间由 Michael Niedermayer 主要负责维护。许多 FFmpeg 的开发人员都来自 MPlayer 项目，而且当前 FFmpeg 也是放在 MPlayer 项目组的服务器上。项目的名称来自 MPEG 视频编码标准，前面的 "FF "代表 "Fast Forward"。

FFmpeg 被许多开源项目采用，QQ 影音、暴风影音、VLC 等。

windows下载地址：[https://ffmpeg.zeranoe.com/builds/win64/static/ffmpeg-20190805-5ac28e9-win64-static.zip](https://ffmpeg.zeranoe.com/builds/win64/static/ffmpeg-20190805-5ac28e9-win64-static.zip)

---

## FFmpeg 下载与安装

1、下载并解压

解压后目录结构：

```text
ffmpeg
    ├─bin
    ├─doc
    └─presets
```

2、将 'ffmpeg\bin' 目录配置到 path 环境变量中

3、输入命令检测是否安装成功

```bash
ffmpeg - version
```

## FFmpeg 基本使用

### 生成 m3u8 、ts 文件

#### 1、cmd 输入命令将 `.avi` 视频转换成 `mpt`

```bash
ffmpeg.exe -i D:/lucene.avi -c:v libx264 -s 1280x720 -pix_fmt yuv420p -b:a 63k -b:v 753k -r 18 D:/lucene.mp4
```

参数解析：

- `-c:v` : 视频编码为 x264 ，x264 编码是 H264 的一种开源编码格式。

- `-s` : 设置分辨率

- `-pix_fmt yuv420p` ：设置像素采样方式，主流的采样方式有三种，YUV4:4:4，YUV4:2:2，YUV4:2:0，它的作用是根据采样方式来从码流中还原每个像素点的 YUV（亮度信息与色彩信息）值。

- `-b` : 设置码率，-b:a 和 -b:v 分别表示音频的码率和视频的码率，-b 表示音频加视频的总码率。码率对一个视频质量有很大的作用，后边会介绍。

- `-r` ：帧率，表示每秒更新图像画面的次数，通常大于 24 肉眼就没有连贯与停顿的感觉了。

#### 2、cmd 输入命令将 `mp4` 生成 `m3u8`

```bash
ffmpeg -i lucene.mp4 -hls_time 10 -hls_list_size 0 -hls_segment_filename ./hls/lucene_%05d.ts ./hls/lucene.m3u8
```

参数解析：

- `-hls_time` : 设置每片的长度，单位为秒

- `-hls_list_size n` : 保存的分片的数量，设置为0表示保存所有分片

- `-hls_segment_filename` ：段文件的名称，%05d 表示5位数字

生成的效果是：将 lucene.mp4 视频文件每 10 秒生成一个 ts 文件，最后生成一个 m3u8 文件，m3u8 文件是 ts 的索引文件。

## 码率的设置

码率又叫比特率即每秒传输的 bit 数，单位为 bps(Bit Per Second)，码率越大传送数据的速度越快。

码率的计算公式是：`文件大小（转成bit）/ 时长（秒）/1024 = kbps` 即每秒传输千位数

例如一个 1M 的视频，它的时长是 10s，它的码率等于:

```text
1*1024*1024*8/10/1024 = 819 kbps
```













