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
    - 文件上传
---

### 内容简介

​	由于在最近的项目中需要完成多文件的上传功能需求，要求兼容ie6及其以上版本，这里使用**SWFUpload**插件，<font color=red>**要注意此插件是基于flash的**</font>，本文主要介绍了**SWFUpload**的基本用法和注意事项还有工作中遇到的一些问题。

### **SWFUpload**特点

- 允许一次上传多个文件，但会有一个上传队列，队列里文件的上传是逐个进行的，服务器端接收文件时跟普通的表单上传文件是一样的;
- 类似[AJAX](https://baike.baidu.com/item/AJAX)的无刷新上传；
- 可以显示上传进度；
- 良好的浏览器兼容性；
- 兼容其他JavaScript库 (例如：[jQuery](https://baike.baidu.com/item/jQuery), Prototype等)；
- 可以在浏览器端就对要上传的文件进行限制;
- 提供了丰富的事件接口供开发者使用;

### SWFUpload参数设置

这里使用的是V2.2.0版本，首先在页面引入相关js

```html
<script src='/swfupload/swfupload.js'></script>
<script src='/swfupload/swfupload.queue.js'></script>
<script src='/swfupload/fileprogress.js'></script>
<script src='/swfupload/handlers.js'></script>
```

然后配置参数

```html
<!-- Html Code-->
<div id="FileList"></div>
<div id="spanSWFUpload">
	<span id="spanSWFUploadButton"></span>
</div>
<button type="button" id="ubutton">上传</button>
```


```javascript
var swfu;
var settings_object = {
	upload_url: "ServerURL",//服务器地址
	flash_url: "/swfupload/swfupload.swf",//swf文件路径
	button_placeholder_id: "spanSWFUploadButton",
	file_size_limit: "20MB", //设置文件选择对话框的文件大小过滤，该属性可接收一个带单位的数值，可用的单位有B,KB,MB,GB。如果忽略了单位，那么默认使用KB。特殊值0表示文件大小无限制。
	file_upload_limit: 12, //设置SWFUpload实例允许上传的最多文件数量
	file_queue_limit: 12, //设置文件上传队列中等待文件的最大数量限制
	button_width: "60",
	button_height: "18",
	button_text: '<span class="theFont">添加文件</span>',
	button_text_style: ".theFont { font-size: 12;}",
	button_text_left_padding: 5,// 设置Flash Button上文字距离左侧的距离，可以使用负值。
	button_text_top_padding: 0,//设置Flash Button上文字距离顶部的距离，可以使用负值。
	//button_action: SWFUpload.BUTTON_ACTION.SELECT_FILE, //设置每次只能选择1个文件
	post_params: {}, //选填，文件上传时附带的参数，也可以在后续动态添加
	use_query_string: false,//设置post_params是否以GET方式发送。如果为false，那么则以POST形式发送。
	requeue_on_error: false,//当文件对象发生uploadError时，文件的处理方式，如果为true,该文件对象会被重新插入到文件上传队列的前端，而不是被丢弃。
	file_types: "*.doc;*.docx;*.pdf;*.xls;*.xlsx",
	button_cursor: SWFUpload.CURSOR.HAND,//设置鼠标划过Flash Button时的光标状态。
    //下面为一系列的回调函数，在文章的后续部分会一一介绍
	file_queued_handler: fileQueued,
	file_queue_error_handler: fileQueueError,
	upload_start_handler: uploadStart,
	upload_error_handler: uploadError,
	upload_success_handler: uploadSuccess,
	upload_complete_handler: uploadComplete
	};
	swfu = new SWFUpload(settings_object);//创建一个SWFUpload对象，并将配置的参数设置进去
	//绑定上传按钮，点击开始上传
	document.getElementById("ubutton").onclick = function (ev) {
         swfu.startUpload();
    }
```

​	写到这里，我们的上传就已经可以使用了，但是没有任何的提示，需要注意的是，选择完文件<font color=red>**插件会自动开始上传**</font>，如果想自己控制，点击按钮才开始上传需要将 handles.js 中的 `this.startUpload()`注释掉，自己手动调用 `startUpload()`方法。

### 回调函数

   - **file_queued_handler：fileQueued(file object) **  当文件选择对话框关闭消失时，如果选择的文件成功加入上传队列，那么针对每个成功加入的文件都会触发一次该事件（N个文件成功加入队列，就触发N次此事件，我们可以在这里完成已选择文件列表的展示功能。
   - **file_queue_error_handler：fileQueueError(file object, error code, message)**   当选择文件入队失败时触发，每个出错的文件都会触发一次该事件，文件添加队列出错的原因可能有：超过了上传大小限制(-110)，文件为零字节(-120)，超过文件队列数量限制(-100)，设置之外的无效文件类型(-130)。
   - **upload_start_handler:uploadStart(file object) **  在文件往服务端上传之前触发此事件，可以在这里完成上传前的最后验证以及其他你需要的操作，例如添加、修改、删除post数据等。在完成最后的操作以后，如果函数返回false，那么这个上传不会被启动，并且触发uploadError事件，如果返回true或者无返回，那么将正式启动上传。
   - **upload_error_handler：uploadError(file object, error code, message)   **



