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

## maven打包命令
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