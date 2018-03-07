---
title: idea下MybatisPlugin的使用test3
date: 2016-04-07 16:46:46
categories:
- 生产力工具
tags:
- idea
- MybatisPlugin
- MBG
---
换用mybatis后这个mapper当字段多的时候真的是挺烦,作为先进生产力的代表idea当然有方法那就是MybatisPlugin。
### MybatisPlugin安装
这个插件现在收费了,这里有一个破解版的[链接](http://yun.baidu.com/share/link?shareid=3257745653&uk=3005471020)。下下来从本地安装插件就行了。
安装好重启idea后新建文件就会有mybatis_generaator_config文件了。
{% asset_img 1.png mybatis_generaator_config %}
### MBG 配置
配置文件可以看这个[文档](http://mbg.cndocs.tk/index.html)
放一下我测试的配置：
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE generatorConfiguration PUBLIC
        "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd" >
<generatorConfiguration>

    <!-- !!!! Driver Class Path !!!! -->
    <classPathEntry location="E:\apache-maven-3.3.1\repository\mysql\mysql-connector-java\5.1.34\mysql-connector-java-5.1.34.jar"/>

    <context id="context" targetRuntime="MyBatis3">
        <commentGenerator>
            <property name="suppressAllComments" value="false"/>
            <property name="suppressDate" value="true"/>
        </commentGenerator>

        <!-- !!!! Database Configurations !!!! -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver" connectionURL="jdbc:mysql://192.168.10.13:3306/test?characterEncoding=utf-8" userId="root" password="root"/>

        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>

        <!-- !!!! Model Configurations !!!! -->
        <javaModelGenerator targetPackage="com.vncnliu.persistence.model" targetProject="src/main/java">
            <property name="enableSubPackages" value="false"/>
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>

        <!-- !!!! Mapper XML Configurations !!!! -->
        <sqlMapGenerator targetPackage="com.vncnliu.persistence.mapper" targetProject="src/main/resources">
            <property name="enableSubPackages" value="false"/>
        </sqlMapGenerator>

        <!-- !!!! Mapper Interface Configurations !!!! -->
        <javaClientGenerator targetPackage="com.vncnliu.persistence.mapper" targetProject="src/main/java" type="XMLMAPPER">
            <property name="enableSubPackages" value="false"/>
        </javaClientGenerator>

        <!-- !!!! Table Configurations !!!! -->
        <!-- !!!! domainObjectName用于指定生成的类名,如不设置则是驼峰原则的tableName !!!! -->
        <table tableName="mapper_test" domainObjectName="test" enableCountByExample="false" enableDeleteByExample="false"
               enableSelectByExample="false" enableUpdateByExample="false">
            <generatedKey column="id" sqlStatement="MySql" identity="true"/>
        </table>

    </context>
</generatorConfiguration>
```
在创建的mybatis_generaator_config文件上右键Run As Mybatis Generator
{% asset_img 2.png Run As Mybatis Generator %}
就会在指定的文件加下创建对应的文件，我测试的在idea下用这种方式targetProject并不起作用。而使用maven的方式是可以的。
### maven 执行 MBG
如果要用maven的方式现在pom里加上
```xml
<plugin>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-maven-plugin</artifactId>
    <version>1.3.2</version>
    <configuration>
        <verbose>true</verbose>
        <overwrite>true</overwrite>
    </configuration>
</plugin>
```
然后在run configura里添加一个maven配置
working directory: 为项目文件所在的路径
command line: mybatis-generator:generate -e
{% asset_img 3.png maven configura %}
然后run一下搞定
可能会报以下异常，创建一下config目录就行了
```java
Caused by: java.lang.RuntimeException: Cannot resolve classpath entry
```
