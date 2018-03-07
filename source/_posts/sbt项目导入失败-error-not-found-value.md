---
title: sbt项目导入失败 error: not found: value
date: 2016-07-28 14:07:58
categories:
- 解决的问题
tags:
- sbt
- bulid
---
导入sbt项目时idea提示如下错误
```
Error:Error while importing SBT project:<br/><pre>G:\temp\attackPlay\build.sbt:5: error: not found: value PlayScala
lazy val `attackplaytemp` = (project in file(".")).enablePlugins(PlayScala)
                                                                 ^
sbt.compiler.EvalException: Type error in expression
[error] sbt.compiler.EvalException: Type error in expression
[error] Use 'last' for the full log.
</pre>
```
PlayScala其实是在sbt-plugin-x.x.x.jar中的，找了一圈发现项目的project文件夹下缺少build.properties和plugins.sbt
build.properties
```
sbt.version=0.13.5
```
plugins.sbt
```
logLevel := Level.Warn

resolvers += "Typesafe repository" at "http://repo.typesafe.com/typesafe/releases/"

addSbtPlugin("com.typesafe.play" % "sbt-plugin" % "2.4.2")
```
重点就是**addSbtPlugin("com.typesafe.play" % "sbt-plugin" % "2.4.2")**

网络配置
/etc/sysconfig/network-scripts/