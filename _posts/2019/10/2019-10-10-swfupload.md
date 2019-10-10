---
layout:     post
title:      使用 SWFUpload 插件完成多文件上传
subtitle:   SWFUpload
date:       2019-10-10
author:     MrSunflowers
header-img: img/post-bg-debug.png
keywords_post:  "使用 SWFUpload 插件完成多文件上传"
catalog: true
tags:
    - SWFUpload
    - FileUpload
---

## SWFUpload简介

由于在最近的项目中需要完成多文件的上传功能需求，要求兼容ie6及其以上版本，之前尝试了**WebUploader**插件，但效果并不理想，无奈之下再次寻找其他插件，功夫不负有心人，在百般尝试后找到了**SWFUpload**插件，相比较能于**WebUploader**来说**SWFUpload**更多的功能需要自定义完成，话不多说，先来看下吧。

**SWFUpload**是一个客户端文件上传工具，最初由Vinterwebb.se开发，它通过整合[Flash](https://baike.baidu.com/item/Flash)与[JavaScript](https://baike.baidu.com/item/JavaScript)技术为WEB开发者提供了一个具有丰富功能继而超越传统<input type="file" />标签的文件上传模式。

### **SWFUpload**特点

- 可以同时上传多个文件；

- 类似[AJAX](https://baike.baidu.com/item/AJAX)的无刷新上传；

- 可以显示上传进度；

- 良好的浏览器兼容性；

- 兼容其他JavaScript库 (例如：[jQuery](https://baike.baidu.com/item/jQuery), Prototype等)；

