# Java调用FFmpeg.exe程序

ffmpeg 是一个可行的视频处理程序，可以通过 Java 调用ffmpeg.exe 完成视频处理。

在 java 中可以使用 `Runtime` 类和 `ProcessBuilder` 类两种方式来执行外部程序，工作中至少掌握一种。

这里使用 `ProcessBuilder` 的方式来调用 ffmpeg 完成视频处理。

## ProcessBuilder 测试

```java
//使用processBuilder来调用本机 ping 命令
@Test
public void testProcessBuilder() throws IOException {
    //创建processBuilder对象
    ProcessBuilder builder = new ProcessBuilder();
    //设置第三方应用程序的命令
    builder.command("ping", "127.0.0.1");
    //将标准输入流和错误流合并
    builder.redirectErrorStream(true);
    Process process = builder.start();
    //通过标准输入流来拿到正常和错误的信息
    InputStream inputStream = process.getInputStream();
    //转成字符流
    InputStreamReader isReader = new InputStreamReader(inputStream, "gbk");
    char[] chars = new char[1024];
    int len = -1;
    while ((len = isReader.read(chars)) != -1) {
        String str = new String(chars, 0, len);
        System.err.println(str);
    }
    inputStream.close();
    isReader.close();
}
```

## ProcessBuilder 调用 FFmpeg

```java
@Test
public void testFFmpeg() throws IOException {
    //创建processBuilder对象
    ProcessBuilder builder = new ProcessBuilder();
    //设置第三方应用程序的命令
    List<String> command = new ArrayList<>();
    command.add("D:\\Develop2\\ffmpeg-20190803-9af8ce7-win64-static\\bin\\ffmpeg.exe");
    command.add("-i");
    command.add("D:\\Develop2\\Test\\lucene.avi");
    command.add("-y");//覆盖输出文件
    command.add("-c:v");
    command.add("libx264");
    command.add("-s");
    command.add("1280x720");
    command.add("-pix_fmt");
    command.add("yuv420p");
    command.add("-b:a");
    command.add("63k");
    command.add("-b:v");
    command.add("753k");
    command.add("-r");
    command.add("18");
    command.add("D:\\Develop2\\Test\\lucene2.mp4");
    builder.command(command);
    //将标准输入流和错误流合并
    builder.redirectErrorStream(true);
    Process process = builder.start();
    //通过标准输入流来拿到正常和错误的信息
    InputStream inputStream = process.getInputStream();
    //转成字符流
    InputStreamReader isReader = new InputStreamReader(inputStream, "gbk");
    char[] chars = new char[1024];
    int len = -1;
    while ((len = isReader.read(chars)) != -1) {
        String str = new String(chars, 0, len);
        System.err.println(str);
    }
    inputStream.close();
    isReader.close();

}
```

## 调用 FFmpeg 将 avi 转成 mp4 视频格式

已封装成 [`Mp4VideoUtil.java`](Mp4VideoUtil.java) 工具类

## 调用 FFmpeg 将 mp4 生成 m3u8 文件

已封装成 [`HlsVideoUtil.java`](HlsVideoUtil.java) 工具类
