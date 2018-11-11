---
title: swagger入门
categories:
  - 笔记
tags:
  - swagger
  - 接口
date: 2018-11-06 10:51:43
---
 使用注解注解接口,可以自动生成接口ui文档页面
 <!-- more -->

## 注解使用
* @API
表示一个开放的API，可以通过description简明的描述API的功能，produce指定API的mime类型
* @ApiOperation
一个@API下，可以有多个@ApiOperation,表示针对该API的CRUD操作，在ApiOperation Annotation中还可以通过value，notes描述该操作的作用，response描述正常情况下该请求返回的对象类型
在一个@ApiOperation下，可以通过@ApiResponses描述API操作可能出现的异常情况
* @ApiParam
用于描述该API操作接收的参数类型，value用于描述参数，required指明参数是否为必须。
* @ApiModelProperty
value指定描述，required指明是否为必须
