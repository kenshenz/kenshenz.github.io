---
layout: post
date : 2014-09-06 11:56
title : [PhoneGap/Cordova]Web与Native的交互
category : lessons
tags : [phonegap, cordova, mobile, html5]
postid : 1409975827483
---
{% include JB/setup %}

当我们的Cordova项目需要与设备做交互，例如照相，打电话等，就需要交互。  
在这里我们有两种方式实现交互，第一种是Cordova插件，第二种是通过WebView组件提供地API。

##Cordova插件

Cordova通过插件来扩展自身的功能，我们可以把这部分功能单独抽离出来作为插件，就可以供不同的项目使用，Cordova官方已经提供了很多有用的插件，像Camera,Contacts,Media等等，具体可以到[这里](http://docs.phonegap.com/en/3.5.0/cordova_plugins_pluginapis.md.html#Plugin%20APIs)看看，下面以Android为例来说说Cordova插件的开发

###Cordova插件的开发






