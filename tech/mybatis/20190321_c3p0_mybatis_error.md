# 使用C3P0实现自定义连接池报`isClosed() is abstract`

## 问题描述

今天在玩Mybatis，使用C3P0实现自定义的连接池报：

> java.lang.AbstractMethodError: Method com/mchange/v2/c3p0/impl/NewProxyPreparedStatement.isClosed()Z is abstract

## 解决方案

修复之前Maven的POM文件部分依赖包：

```XML
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.4.6</version>
</dependency>

<dependency>
    <groupId>com.mchange</groupId>
    <artifactId>c3p0</artifactId>
    <version>0.9.1.2</version>
</dependency>
```

经过一番折腾发现，使用的Mybatis版本是`3.4.6`；而使用的C3P0是`0.9.1.2`版本，两个版本不兼容；后面将C3P0的版本升级到`0.9.5.3`就好。

解决之后的POM文件相关部分：

```XML
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.4.6</version>
</dependency>

<dependency>
    <groupId>com.mchange</groupId>
    <artifactId>c3p0</artifactId>
    <version>0.9.5.3</version>
</dependency>
```