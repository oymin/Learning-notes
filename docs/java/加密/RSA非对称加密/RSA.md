# RSA非对称加密

RSA 生成一对密钥：公钥（publicKey）和私钥（privateKey）

## 加密流程：

1、加密签名：将加密内容与 privateKey 对加密内容进行签名

2、验签：使用 publicKey 对加密内容验签，解密

## 验签过程

调用方在请求头中传递 authId , sign

使用 AOP 在执行实际方法前根据 authId 获取到公钥，进行验签

验签通过就继续执行

## pom 中加密的依赖

```xml
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.12</version>
</dependency>
```











