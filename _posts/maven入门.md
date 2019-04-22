---
title: maven入门
categories:
  - 笔记
tags:
  - maven
date: 2018-08-18 09:46:59
---
 这是摘要
 <!-- more -->


## maven插件
1. 统计项目代码行数插件开发
>参考地址:https://blog.csdn.net/zjf280441589/article/details/53044308/

## maven Archetype的使用
### idea中创建方式
1. 创建项目骨架模板
2. 在pom文件中添加插件,将resource路径下的空目录打包进archetype,需要注意的是,src/main/java路径下的空目录不会被打包进archetype
```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-resources-plugin</artifactId>
    <version>3.0.2</version>
    <configuration>
        <includeEmptyDirs>true</includeEmptyDirs>
    </configuration>
</plugin>
```
3. 在根目录运行`mvn archetaye:create-from-project`
4. `cd target/generated-sources/archetype`执行`mvn install`命令安装到本地
5. 在IDEA新建项目时使用groupId,artifactId和version添加新的archetype,如果要删除自定义的archetype,可以`/Users/huangning/Library/Caches/IntelliJIdea2018.2/Maven/Indices/UserArchetypes.xml`中删除



## maven打包命令
### 打包跳过测试
1. `mvn clean package -Dmaven.test.skip=true`

2. 在pom.xml文件中配置
```
<plugin>  
    <groupId>org.apache.maven.plugin</groupId>  
    <artifactId>maven-compiler-plugin</artifactId>  
    <version>2.1</version>  
    <configuration>  
        <skip>true</skip>  
    </configuration>  
</plugin>  
<plugin>  
    <groupId>org.apache.maven.plugins</groupId>  
    <artifactId>maven-surefire-plugin</artifactId>  
    <version>2.5</version>  
    <configuration>  
        <skip>true</skip>  
    </configuration>  
</plugin> 

```

**使用以上两种方式,不但跳过单元测试的运行，也跳过测试代码的编译。**
3. `mvn clean package -DskipTests`
只是跳过测试,但是还是会编译测试代码

### 多模块打包,单独打包一个模块
```
-pl, --projects
        Build specified reactor projects instead of all projects
-am, --also-make
        If project list is specified, also build projects required by the list
-amd, --also-make-dependents
        If project list is specified, also build projects that depend on projects on the list
```
1. 首先到根目录
2. 单独构建模块 connection，同时会构建 connection 模块依赖的其他模块**注意点:这里使用的是模块在pom文件中的相对路径,而不仅仅是模块名**
```
mvn install -pl ../connection -am
```
3. 单独构建模块 base，同时构建依赖模块 base 的其他模块
```
mvn install -pl base -am -amd
```

## 变量
```
<properties>
	<!-- 使用的时候 ${swagger.version} 就行 -->
	<swagger.version>2.2.2</swagger.version>
</properties>
```