# log4j2.xml 日志配置文件

## 文件用途

- 配置输出日志的格式

- 配置输出日志的级别

## 配置标签

### `<Pattern>` 配置 MyCAT 日志格式

```xml
<PatternLayout>
    <Pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %5p [%t] (%l) - %m%n</Pattern>
</PatternLayout>
```

| &emsp; 属性 &emsp; | 说明 |
| :----: | ---- |
| %d   | 日志的输出时间格式 |
| %5p  | 标识日志输出的级别，5：级别名称5个固定字符 |
| %t   | 日志中记录线程名称 |
| %m   | 日志输出的具体消息 |
| %n   | 输出一个回车换行符，windws：/r/n ，liunx：/n |

### level 属性配置日志级别

```xml
<AsyncLogger level="info"> </AsyncLogger>
```

日志级别：低 < 高

`all` < `trace` < `debug` < `info` < `warn` < `error` < `fatal` < `off`




