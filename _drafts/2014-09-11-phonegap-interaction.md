---
layout: post
date : 2014-09-11 13:56
title : PhoneGap/Cordova Web与Native的交互
category : lessons
tags : [phonegap, cordova, mobile, html5]
postid : 1410414980593
---
{% include JB/setup %}

当我们的Cordova项目需要调用设备的一些功能，例如照相，打电话等，又或者我们想实现文件的续点上传下载等功能，脚本语言无法满足我们的需求，这时候就需要Web与Native进行交互，使用原生API实现。  
在这里我们有两种方式实现交互，第一种是Cordova插件，第二种是通过WebView组件提供的API。

##Cordova插件

Cordova通过插件来扩展自身的功能，我们可以把这部分功能单独抽离出来作为插件，供不同的项目使用，Cordova官方已经提供了很多有用的插件，像Camera,Contacts,Media等等，具体可以到[这里](http://docs.phonegap.com/en/3.5.0/cordova_plugins_pluginapis.md.html#Plugin%20APIs)看看。

###Cordova插件的添加

针对跨平台开发模式（即直接在Cordova项目下编码），添加插件是一件很容易的事情，Cordova已经提供了命令来完成这一操作。

以添加org.apache.cordova.dialogs插件（dialogs提供了alert,confirm等函数）为例

	$>cd HelloCordova
	$>cordova plugin add https://github.com/apache/cordova-plugin-dialogs.git
	
由于国内网络问题，这里请使用github上的插件地址。  
插件下载完成后，Cordova会自动把插件添加到当前所有平台下，一个插件的添加就这么简单的完成了。

接着可以直接调用dialog插件的函数  

```
navigator.notification.confirm('是否确认退出?',console.log('应用推出日志.'),'提示',['确认','取消']);
```

可能有同学会疑问Cordova到底帮我们做了些什么呢？在搞清楚这个问题之前，我们先来看看插件的开发，学会插件的开发，以及配置之后，就自然明白Cordova到底做了些什么。

###Cordova插件的开发

由于插件是平台相关的，这里以Android平台为例，开发一个用于打开Activity的插件。  
首先，在IDE里创建一个项目，其主要的目录结构如下：

	|-src				#插件的原生代码，各个平台下的插件代码都要单独开发的
	  |-android
	  |-ios
	  |-wp
	  ...
	|-www
	  |-activity.js		#插件的JS接口，通过接口来调用原生代码
	|-plugin.xml		#插件的配置文件，Cordova需要该文件来配置插件

接着，编写插件的业务逻辑，源文件放到src/android目录下，代码如下：

```java
package org.hyena.cordova.activity;		//包路径是插件的唯一标识

public class ActivityPlugin extends CordovaPlugin {	//插件必须继承CordovaPlugin类

	@Override
	public void initialize(CordovaInterface cordova, CordovaWebView webView) {
		// TODO Auto-generated method stub
		super.initialize(cordova, webView);
	}
	
	/**
	 * 重写execute方法，这就是插件的统一接口
	 * 参数action：接口方法名
	 * 参数args：传递的参数
	 * callbackContext：插件调用的上下文，可以用来返回处理结果到页面
	 */
	@Override
	public boolean execute(String action, JSONArray args,
			CallbackContext callbackContext) throws JSONException {
		
		if("startActivity".equals(action)) {
			String activityPkg = args.getString(0);
			
			try {
				Class activityClass = Class.forName(activityPkg);
				Intent intent = new Intent(this.cordova.getActivity(), activityClass);
				
				this.cordova.getActivity().startActivity(intent);
			} catch (ClassNotFoundException e) {
				e.printStackTrace();
			}  
		}
		
		return true;
	}
}
```

然后，编写插件的JS接口activity.js，代码如下：

```javascript
/**
 * @fileOverview activity插件
 * @author cf.chen
 * @version 1.0.0
 */

//这是cordova提供的执行器
var exec = require('cordova/exec');

//插件定义
function ActivityPlugin(){

	//插件的接口方法
	this.startActivity = function(activity){
		//成功回调，失败回调，插件（特性）名称，接口方法，参数
		exec(null, null, "ActivityPlugin", "startActivity", [activity]);
	}
	
	this.onSuccess = function(rtn){
	}
	
	this.onError = function(){
	}
}

var me = new ActivityPlugin();
module.exports = me;
```

下一步，编写插件的配置文件plugin.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<plugin xmlns="http://apache.org/cordova/ns/plugins/1.0" 
    id="org.hyena.cordova.activity"
	version="1">
	
	<name>Activity Plugin</name>
	<description>Activity Plugin</description>
	<keywords>cordova,heartbeat</keywords>
	
	<!-- 插件模块，一个插件可以有多个模块 -->
	<js-module src="www/activity.js" name="activity">
		<!-- 这里指定插件的js对象名称，以后就可以直接在页面使用window.activity了 -->
		<clobbers target="activity" />
	</js-module>
	
	
	<platform name="android">
		<!-- 针对Android平台，设置cordova特性 -->
        <config-file target="res/xml/config.xml" parent="/*">
            <feature name="ActivityPlugin">
                <param name="android-package" value="org.hyena.cordova.activity.ActivityPlugin"/>
            </feature>
        </config-file>
		<!-- 把插件源代码复制到平台的目标路径下 -->
        <source-file src="src/android/ActivityPlugin.java" target-dir="src/org/hyena/cordova/activity" />
    </platform>
	
</plugin>
```

最后，把我们的插件添加到Cordova项目中去

	$>cd HelloCordova
	$>cordova plugin add '插件项目根目录'

编译生成后，我们的Android项目发生了几样变化：

- 增加了一个源文件src/org/hyena/cordova/activity/ActivityPlugin.java

- 增加了一个插件接口文件assets/www/plugins/org.hyena.cordova.activity/www/activity.js，代码发生了一点变化，如下：

```javascript
//在原来的代码外边包了一层cordova的脚本，这里的org.hyena.cordova.activity.activity表示插件名为org.hyena.cordova.activity，模块名为activity。
cordova.define("org.hyena.cordova.activity.activity", function(require, exports, module) {
/**
 * @fileOverview activity插件
 * @author cf.chen
 * @version 1.0.0
 */

var exec = require('cordova/exec');


function ActivityPlugin(){
	...
}

var me = new ActivityPlugin();
module.exports = me;

});
```

- 修改了assets\www\cordova_plugins.js文件，cordova启动时会读取该文件，来初始化插件。修改如下：

```
cordova.define('cordova/plugin_list', function(require, exports, module) {
module.exports = [
	...
	//增加了activity插件的配置说明
	{
        "file": "plugins/org.hyena.cordova.activity/www/activity.js",
        "id": "org.hyena.cordova.activity.activity",
        "clobbers": [
            "activity"
        ]
    },
	...
];
```	

- 修改了res\xml\config.xml文件，应用启动时会读取该文件，来配置特性。修改如下：

```xml
<?xml version='1.0' encoding='utf-8'?>
<widget id="org.hyena.mobileofficeapp" version="0.0.1" xmlns="http://www.w3.org/ns/widgets" xmlns:cdv="http://cordova.apache.org/ns/1.0">
	...
	<!-- 增加ActivityPlugin特性 -->
	<feature name="ActivityPlugin">
        <param name="android-package" value="org.hyena.cordova.activity.ActivityPlugin" />
    </feature>
	...
</widget>
```

总结一下，Cordova做了如下几样事情：

1. 复制java源文件到项目src下（如果需要）

2. 修改AndroidManifest.xml，增加Activity、Service、Receiver、Permission等（如果需要）

3. 添加类库到项目libs下（如果需要）

4. 添加资源文件到res下（如果需要）

5. 复制js源文件到plugins下，并修改

6. 修改cordova_plugin.js文件，增加插件模块的描述

7. 修改config.xml文件，增加特性的描述


明白了插件怎么开发后，我们就可以直接在混合模式的项目下编写插件，手动配置插件，而不需要cordova plugin命令了。  
可以以[极光推送插件](https://github.com/jpush/jpush-phonegap-plugin)来练练手，该插件包含了上述7样事情。

##WebView API

每个平台都会提供Web组件，Android下是WebView组件。通过WebView API，我们可以实现Web与Native的交互。以Android平台为例，通过webView.loadUrl()方法在原生代码中调用执行javascript脚本，通过webView.addJavascriptInterface()方法在原生代码中监听javascript脚本。

下面是一个简单的例子

```java
public class MainActivity extends Activity {
	
	private WebView webView;
	private Handler handler = new Handler();

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		super.setContentView(R.layout.main);
		
		webView = (WebView) this.findViewById(R.id.webView);
		WebSettings setting = webView.getSettings();
		setting.setJavaScriptEnabled(true);						//启动javascript
		
		webView.setWebChromeClient(new WebChromeClient());		//WebChromeClient用于渲染界面
		
		//监听javascript的调用
		webView.addJavascriptInterface(new Object() {
			
			@JavascriptInterface
			public void showDate() {
				//执行javascript脚本
				webView.loadUrl("javascript:alert(new Date());");
				/*handler.post(new Runnable() {
					@Override
					public void run() {
						webView.loadUrl("javascript:alert(new Date());");
					}
				});*/
			}

		}, "tool");
		
		webView.loadUrl("file:///android_asset/demo.html");
	}
}
```

而页面的代码如下：

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
<button onclick="window.tool.showDate()">显示当前时间</button>
</body>
</html>
```

通过WebView API可以很轻松，高效的实现Web与Navtive的交互，但其不能重用的缺点也是不言而喻。

what is the github favor markdown.
i think is good thing.
