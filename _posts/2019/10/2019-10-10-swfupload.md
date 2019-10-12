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
- 兼容其他JavaScript库 (例如:[jQuery](https://baike.baidu.com/item/jQuery), Prototype等)；
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

​	写到这里，我们的上传就已经可以使用了，但是没有任何的提示，需要注意的是，默认情况下，你选择了文件并关闭了文件选择对话框文件并不会自动开始上传，你需要调用SWFUpload的startUpload()方法，该方法接收一个file_id作为参数，如果参数为空，则自动开始上传文件队列中的第一个文件。选择完文件后<font color=red>**如果文件会自动开始上传**</font>，可以将 handles.js 中的 this.startUpload()注释掉即可。<font color=red>**Swfupload虽支持批量上传，但本质仍是单个文件依次上传，也就是说，每次上传文件都是一次独立的请求**</font>。

### 回调函数

   - **file_queued_handler:fileQueued(file object) **  当文件选择对话框关闭消失时，如果选择的文件成功加入上传队列，那么针对每个成功加入的文件都会触发一次该事件（N个文件成功加入队列，就触发N次此事件，我们可以在这里完成已选择文件列表的展示功能。

```javascript
	//File Object包含了一组可用的文件属性
	{
		id : string, // SWFUpload控制的文件的id,通过指定该id可启动此文件的上传、退出上传等
		index : number, // 文件在选定文件队列（包括出错、退出、排队的文件）中的索引，getFile可使用此索引
		name : string, // 文件名，不包括文件的路径。
		size : number, // 文件字节数
		type : string, // 客户端操作系统设置的文件类型
		creationdate : Date, // 文件的创建时间
		modificationdate : Date,	// 文件的最后修改时间
		filestatus : number // 文件的当前状态
	}
	
```
   - **file_queue_error_handler:fileQueueError(file object, error code, message)**   当选择文件入队失败时触发，每个出错的文件都会触发一次该事件，文件添加队列出错的原因可能有:超过了上传大小限制(-110)，文件为零字节(-120)，超过文件队列数量限制(-100)，设置之外的无效文件类型(-130)。

   - **upload_start_handler:uploadStart(file object) **  在文件往服务端上传之前触发此事件，可以在**这里完成上传前的最后验证**以及其他你需要的操作，例如**添加、修改、删除post数据**等。在完成最后的操作以后，如果函数返回false，那么这个上传不会被启动，并且触发uploadError事件，如果返回true或者无返回，那么将正式启动上传。

   - **upload_error_handler:uploadError(file object, error code, message)   **只要上传被终止或者没有成功完成，那么该事件都将被触发。具体的错误代码如下:
```javascript
     SWFUpload.UPLOAD_ERROR = {
     HTTP_ERROR : -200,
     MISSING_UPLOAD_URL : -210,
     IO_ERROR : -220,
     SECURITY_ERROR : -230,
     UPLOAD_LIMIT_EXCEEDED : -240,
     UPLOAD_FAILED : -250,
     SPECIFIED_FILE_ID_NOT_FOUND	: -260,
     FILE_VALIDATION_FAILED : -270,
     FILE_CANCELLED : -280,
     UPLOAD_STOPPED : -290
     };
```

- **upload_success_handler:uploadSuccess(file object, server data)**   当文件上传的处理已经完成（这里的完成只是指向目标处理程序发送了Files信息，只管发，不管是否成功接收），并且<font color=red>**服务端返回了200的HTTP状态**</font>时，触发此事件。在window平台下，服务端的处理程序在处理完文件存储以后，<font color=red>**必须返回一个非空值，否则此事件不会被触发**</font>

- **upload_complete_handler:uploadComplete(file object)**   当上传队列中的<font color=red>**一个文件**</font>完成了一个上传周期，无论是成功(uoloadSuccess触发)还是失败(uploadError触发)，此事件都会被触发，这也标志着一个文件的上传完成，可以进行下一个文件的上传了。

- **upload_progress_handler:uploadProgress(file object, bytes complete, total bytes)**   该事件由flash定时触发，提供三个参数分别访问上传文件对象、已上传的字节数，总共的字节数。因此可以在这个事件中来定时更新页面中的UI元素，以达到及时显示上传进度的效果。<font color=red>**注意:**</font>该事件在Linux版本的Flash Player中存在问题，目前还无法解决。

  这些回调函数可以帮助我们做很多事情，比如在文件加入队列之前做的一些校验，比如我们可以在文件**fileQueued()**方法中完成文件入队前的一些校验，并通过**cancelUpload(fileId)**（会触发uploadError事件）方法来将文件移除队列。
  
### 有关参数的传递

在SWFUpload的配置参数中的post_params参数是用来向后台传递参数的，如果无法传递，请尝试将另一个配置参数use_query_string设置为true之后就可以传递参数了。该参数接收一个JS对象:

```javascript
post_params: {  
    param1: 'Hello',  
    param2: '你好'  
} 
```

但是如果参数值中含有中文的话，那么后台会报错，也取不到值，可以这样解决:

```javascript
post_params: {  
    param1: encodeURI('你好',"utf-8")  
} 
```

然后在后台再decode回来，以Java为例:

```java
URLDecoder.decode(request.getParameter("param1"), "utf-8");  
```

### 其他问题

- **文件上传完成后的页面跳转:**   可以通过监听uploadSuccess(file object, server data)方法，通过JS控制挑转，<font color = "red" >**该事件在每个文件上传成功均会触发**</font>如果同时在上传多个文件，那么第一个文件上传完成后页面就直接跳转了，后面的文件将不会上传，可以通过SWFUpload.getStats().files_queued是否为0来判断是否还有未上传的文件。
- **在非IE浏览器中Session不同的问题:**   这是由于Flash Player在非IE浏览器下一个Bug引起的，导致用户登录的Session和文件上传产生的Session不同，也就是说文件上传另生成了一个新的Session。解决办法是手动将SessionID传到后台服务端，在上传路径URL里加上jsessionid变量即可。
```java
upload_url: "ServerURL;jsessionid=<%=request.getSession().getId() %>"
```

### 常用方法

```javascript
//指定file_id来启动该文件的上传，如果file_id被忽略了，那么默认开始上传第一个文件。
void startUpload(file_id) 
//指定file_id来退出文件的上传，从上传队列中删除该文件。
//如果忽略file_id，那么默认文件上传队列中的第一个文件将被退出上传。
//如果取消的文件是正在上传，那么会触发uploadError事件。
//如果将可选参数trigger_error_event设置为false,那么uploadError事件不会触发。
void cancelUpload(file_id, trigger_error_event) 
//如果当前有文件上传，那么停止上传，并且将文件还原到上传队列中。
//停止了正在上传的文件，uploadError事件会被触发。如果此时没有正在上传文件，那么不会发生任何操作，不会触发任何事件。
void stopUpload() 
//获取当前状态的统计对象,具体见Stats Object
object getStats() 
Stats Object:{
	in_progress : number // 值为1或0，1表示当前有文件正在上传，0表示当前没有文件正在上传
	files_queued : number // 当前上传队列中存在的文件数量
	successful_uploads : number	// 已经上传成功（uploadSuccess触发）的文件数量
	upload_errors : number // 已经上传失败的文件数量 (不包括退出上传的文件)
	upload_cancelled : number	// 退出上传的文件数量
	queue_errors : number // 入队失败（fileQueueError触发）的文件数量
}
//动态修改post_params，以前的属性全部被覆盖。param_object必须是一个JavaScript的基本对象，所有属性和值都必须是字符串类型。
void setPostParams(param_object)

```

更多详细的功能请参考[SWFUpload V2.2.0 API 说明文档](http://www.leeon.me/upload/other/swfupload.html)
