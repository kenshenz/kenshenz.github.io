---
layout: post
category : lessons
tagline: "Supporting tagline"
tags : [intro, beginner, jekyll, tutorial]
---
{% include JB/setup %}

#PhoneGap/Cordova开发环境配置

这里以Cordova3.5的安装为例，由于Cordova3.X不再直接提供编译好的包，所以只能从源码编译（这也是Cordova推荐的方式）

1. 安装[Node.js](http://www.nodejs.org/)，这个没什么好说，直接执行安装即可。
2. 安装[Git](http://www.git-scm.com/)，安装Cordova，下载Cordova的项目模板都是通过Git来下载的。Git的安装也很简单，直接执行安装即可（安装过程中记得勾选把Git添加的Path路径下，这样子方便后面的使用）
3. 使用Node.js的npm命令（Node.js的模块下载工具）来下载Cordova

	$>npm install -g cordova

通过npm安装的模块会保存到C:\Users\<i>username</i>\AppData\Roaming\npm
