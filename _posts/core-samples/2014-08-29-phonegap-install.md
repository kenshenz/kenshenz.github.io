---
layout: post
date : 2014-08-29 17:06
title : PhoneGap/Cordova 开发环境配置
category : lessons
tags : [phonegap, cordova, mobile, html5]
postid : 1409396959529
---
{% include JB/setup %}

这里以Windows平台下Cordova3.5的安装为例，Cordova3.X不再直接提供编译好的包，改为以Node.js模块发布。

##安装Node.js 

Node.js是一个Javascript运行环境，我们暂时不需要去了解它，只要知道Cordova是一个Node.js模块，运行Cordova需要Node.js运行环境就行了。  
安装[Node.js](http://www.nodejs.org/)，这个没什么好说，直接下载安装包并执行安装即可。<!--break-->

##安装Cordova

使用Node.js的npm命令（Node.js的模块下载工具）来下载Cordova,通过npm安装的模块会保存到C:\Users\\*username*\AppData\Roaming\npm

    $>npm install -g cordova

安装成功后，就能使用cordova命令了。
如果安装失败，有可能是因为网络代理问题，需要设置npm的代理。  

**npm代理设置：**
    
    $>npm config set https-proxy http://192.168.99.100:80
    $>npm config set proxy http://192.168.99.100:80
        
##安装Git

之所以安装[Git](http://www.git-scm.com/)，是因为创建Cordova项目时需要使用到的模板都是从git仓库上拉下来的。Windows下Git的安装也很简单，直接执行安装即可（安装过程中记得勾选把Git添加到Path路径下，这样方便后面直接在命令行中使用）。  
同样，如果公司不能直接连外网，则需要设置一下git的代理。

**git代理设置：**
    
    $>git config --global http.proxy http://192.168.99.100:80
    $>git config --global https.proxy http://192.168.99.100:80

## * 修改Cordova项目模板下载地址

该步骤不是必须的，但由于cordova默认是从美国的git仓库下载cordova项目模板，所以常常会因为超时导致下载失败。  
修改C:\Users\\*username*\AppData\Roaming\npm\node_modules\cordova\node_modules\cordova-lib\src\cordova目录下的platforms.js文件，具体如下：

    'android' : {
        parser : './metadata/android_parser',
        //url    : 'https://git-wip-us.apache.org/repos/asf?p=cordova-android.git',
        url    : 'https://github.com/apache/cordova-android/archive/3.5.0.tar.gz?',
        version: '3.5.0'
    },
    'www':{
        hostos : [],
        //url    : 'https://git-wip-us.apache.org/repos/asf?p=cordova-app-hello-world.git',
	    url    : 'https://github.com/apache/cordova-app-hello-world/archive/3.5.0.tar.gz?',
        version: '3.5.0'
    },
        
www代表cordova项目，一定要改，其他平台看情况，用到哪个平台就修改哪个。  
（在github上能找到替代路径）

##创建Cordova项目

使用cordova命令创建cordova项目

    $>cordova create HelloCordova com.ksn.cordova HelloCordova
        
其中第一个参数是项目存放路径，第二个参数项目的包路径，第三个参数是项目名称。  
创建完成后可以看到cordova项目的目录结构如下：
    
    |-hooks
      |-README.md
    |-merges
    |-platforms                 #各个平台项目的存放路径，也是cordova的编译目标路径（不是源路径喔！）
    |-plugins                   #项目使用到的插件存放路径
    |-www                       #项目源代码存放路径
      |-css
        |-index.css
      |-img
        |-logo.png
      |-js
        |-index.js
      |-index.html
    |-config.xml                #项目的配置文件，包含主页面路径、使用到的特性（插件）等内容
        
重点关注有解释的目录
    
##添加平台

Cordova的最大特点就是跨平台，所以我们可以很轻松的在cordova项目中创建我们需要的平台，但是前提是安装了必须的平台环境。  
以Android环境为例，我们需要安装JDK、Android SDK、Ant（安装配置过程就略过了）。  
一切就绪之后，进入刚刚创建cordova项目根目录，使用cordova命令添加android平台。

    $>cd HelloCordova
    $>cordova platform add android
    
创建完成之后，就可以在platforms目录下看到完整的一个android项目了。  
接着可以使用cordova命令来编译一下项目运行看看效果（记得连上手机）。
    
    $>cordova run android

##跨平台模板和混合模式  

到了这里，我们需要确定一下项目今后是采取跨平台模式还是混合模式来开发。  
它们的区别在于：  

- 跨平台模式只在cordova项目的www目录下写代码，其他跟平台有关的都通过插件来实现，每次代码修改后需要使用命令进行编译。
- 混合模式是针对某一个平台进行开发，例如android平台，可以直接在Eclipse中导入cordova项目（实际上是platforms下的android项目）。功能可以直接通过java代码来实现。
- 跨平台模式一套代码，直接能打包成各个平台的应用（如果用到插件需要提供各平台的实现版本），但是每次都要用命令行编译比较麻烦。
    

至此，Cordova的开发环境已配置完成了。后面就是coding and coding。




