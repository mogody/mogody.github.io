---
title: k8s集群
date: 2024-02-28 20:47:13
tags:
---

1. http协议,超文本传输协议
2. 工作在tcp/ip协议基础上
3. web开发数据的传输都是依赖http协议
4. 通过httpwatch插件来抓取http请求
5. http1.0 短连接 //接受完信息就断开
6. http1.1 长连接 //不会马上断开,有超时限制

###http请求(request)

@(学习笔记)

####1. 基本结构
**请求行**
**消息头**
**消息体**
例子：
```
GET /test/hello.html HTTP/1.1 //请求行,表示是GET请求,请求资源是/test/hello.html
Accept: */* //指定客户端接收那些类型信息,*表示任何类型
Referer: http://localhost:80/test/abc.html //表示来源
Accept-Language: zh-cn //页面语言
User-Agent: Mozilla/4.0 //告诉服务器浏览器内核,操作系统
Accept-Encoding: gzip,defalte //接收什么样的数据压缩格式
Host: localhost: 80 //主机
Connection: Keep-Alive //长连接,表示不要立即断掉请求
```
**1.  HTTP请求方式**
	POST,GET,HEAD,OPTIONS,DELETE,TRACE,PUT (常会用的是GET,POST)
**2. GET与POST的区别**
	1) 安全性,GET请求的数据会显示在地址栏,POST请求的数据放在HTTP协议消息体里面
	2) 从可以提交的数据大小看,
		1. HTTP协议本身并没有限制数据大小
		2. 浏览器在对GET和POST请求做限制,GET请求数据2K+35,POST没有限制
	3) GET请求可以更好的添加到收藏夹
**3. 防盗链**
```
	if(strpos($_SERVER['HTTP_REFERER'],"http:localhost")==0){
		echo "来自本网站";
	}
```
**4. 页面跳转**
header("Location: b.php"); //会向客户端发送一个302状态码,告诉浏览器重新访问b.php。header可以向http响应头写入信息

##HTTP响应
###1.基本结构
***状态行**
***消息头**
***实体信息**
例子：
```
HTTP/1.1 200 OK
//200表示客户端请求成功
Server:Microsoft-IIS/5.0 //高速浏览器服务器的情况
Date:Thu,13 Jul 2000 05:46:53 GMT //告诉浏览器 请求页面的时间
Content-Length: 2291 //表示回送的数据有2291字节
Content-Type: text/html //返回文件类型
Cache-control: private //
```
**状态码说明：**
	- 100-199 表示成功接收请求,要求客户端继续提交下一次请求才能完成整个处理过程
	- 200-299 表示成功接收请求并已完成整个处理过程,常用200
	- 300-399 为完成请求,客户端需进一步细化请求,请求的资源已经移动到一个新的地方,常用302 304
	- 400-499 客户端请求有错误,常用404
	- 500-599 服务器端出现错误,常用505
**细节注意：**
1. 302状态码也可以跳转到外网
header("Location: http:www.baidu.com");
2. 404状态码 //请求不存在
3. 304状态码 //服务器返回304表示资源未修改过
比较详细HTTP响应头：
```
Location： http://baidu.com/index.php
Server:apache
Contnet-Encoding:gzip //内容编码,支持gzip压缩算法
Content-Length:80 //返回数据大小
Content-Language:zh-cn
Content-Type:text/html;charser=GB2312 //返回内容类型
Last-Modified: Tue,11 Jul 2000 18:23:51 GMT //请求页面或图片最近更新的时间
Refresh: 1;url=http://www.baidu.com
Content-Disposition: attachment,filename=aaa.zip
Transfer-Encoding: chunked
Ser-Cookie:SS=5Lb_nQ.path=/search
Expires: -1
Cache-Control: no-cache
Pragma: no-cache
Connection: close/Keep-Alive
Date: Tue,11 Jul 2000 18:23:51 GMT
```
###2.通过HTTP响应,控制浏览器间隔时间跳转
`header("Refresh: 3; url=http:www.xtwind.com");`
###3.通过HTTP响应,控制浏览器缓存(需要三句来控制主要是为了兼容多个浏览器)
```
//通过header来禁用缓存(ajax特别注意需要禁用缓存)
header("Expires: -1");
header("Cache-Control: no-cache");
header("Pragma: no-cache");
```
###4.通过HTTP响应,控制文件下载
1. header("content-type:image/png");
2. heade("content-disposition:attachment;filename=a.txt"); //filename下载的时候文件名
3.`header("content-length:30kb"); //下载的时候显示的文件大小
4.header("Accept-Ranges: bytes"); //无所谓有无,返回文件大小类型
5. readfile('a.txt'); //以附件的形式来读出文件
例子：
```
$file_name="Sunset.jpg";
filesize() //获取文件大小
feof() //判断文件指针是否到了文件结尾
fread() //读取文件内容
```
1. 打开文件
``` php
$file_name="Sunset.jpg";
$fp = fopen($file_name,"r");
header("content-type:image/png");
heade("content-disposition:attachment;filename=a.txt");
header("content-length:30kb");
header("Accept-Ranges: bytes");
$buffer = 1024;
$file_count = 0;
while(!feof($fp) && ($file_size-$fize_count)>0){
$file_Data=fread($fp,$buffer);
$file_count+=$buffer;
echo $file_Data;
}
fclose($fp);
```
2.如果文件是中文名(会提示文件不存在)
原因:php文件系列函数比较老,需要对中文转码,转为gb2312,使用icovn()函数处理文件名
iconv("utf-8","gb2312",$file_name);
