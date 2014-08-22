---
layout: post
category : lessons
tagline: "Supporting tagline"
tags : [intro, beginner, jekyll, tutorial]
---
{% include JB/setup %}

#PhoneGap/Cordova������������

������Cordova3.5�İ�װΪ����Cordova3.X����ֱ���ṩ����õİ�����Ϊ��Node.jsģ�鷢����

1. ��װ[Node.js](http://www.nodejs.org/)�����ûʲô��˵��ֱ��ִ�а�װ���ɡ�
2. ʹ��Node.js��npm���Node.js��ģ�����ع��ߣ�������Cordova,ͨ��npm��װ��ģ��ᱣ�浽C:\Users\<i>username</i>\AppData\Roaming\npm

        $>npm install -g cordova
    
    �����װʧ�ܣ��п�������Ϊ����������⣬��Ҫ����npm�Ĵ���  
    ��װ�ɹ��󣬾���ʹ��cordova�����ˡ�
    
    **npm�������ã�**
    
        $>npm config set https-proxy http://192.168.99.100:80
        $>npm config set proxy http://192.168.99.100:80
        
3. ��װ[Git](http://www.git-scm.com/)����������Cordova����Ŀģ�壩��Git�İ�װҲ�ܼ򵥣�ֱ��ִ�а�װ���ɣ���װ�����мǵù�ѡ��Git��ӵ�Path·���£�������������ʹ�ã���  
ͬ���������˾����ֱ��������������Ҫ����һ��git�Ĵ���

    **git�������ã�**
    
        $>git config --global http.proxy http://192.168.99.100:80
        $>git config --global https.proxy http://192.168.99.100:80

4. *�޸�Cordova��Ŀģ������ص�ַ���ò��費�Ǳ���ģ�������cordovaĬ���Ǵ�������git�ֿ�����cordova��Ŀģ�壬���Գ�������Ϊ��ʱ��������ʧ�ܣ�  
�޸�C:\Users\<i>username</i>\AppData\Roaming\npm\node_modules\cordova\node_modules\cordova-lib\src\cordovaĿ¼�µ�platforms.js�ļ����������£�

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
        
    www����cordova��Ŀ��һ��Ҫ�ģ�����ƽ̨��������õ��ĸ�ƽ̨���޸��ĸ���  
    ����github�����ҵ����·����
5. ����Cordova��Ŀ  
ʹ��cordova�����cordova��Ŀ

        $>cordova create HelloCordova com.ksn.cordova HelloCordova
        
    ���е�һ����������Ŀ���·�����ڶ���������Ŀ�İ�·������������������Ŀ���ơ�  
    ������ɺ���Կ���cordova��Ŀ��Ŀ¼�ṹ���£�
    
        |-hooks
          |-README.md
        |-merges
        |-platforms                 #����ƽ̨��Ŀ�Ĵ��·����Ҳ��cordova�ı���Ŀ��·��������Դ·��ร���
        |-plugins                   #��Ŀʹ�õ��Ĳ�����·��
        |-www                       #��ĿԴ������·��
          |-css
            |-index.css
          |-img
            |-logo.png
          |-js
            |-index.js
          |-index.html
        |-config.xml                #��Ŀ�������ļ���������ҳ��·����ʹ�õ������ԣ������������
        
   �ص��ע�н��͵�Ŀ¼
    
6. ���ƽ̨  
Cordova������ص���ǿ�ƽ̨���������ǿ��Ժ����ɵ���cordova��Ŀ�д���������Ҫ��ƽ̨������ǰ���ǰ�װ�˱����ƽ̨������  
��Android����Ϊ����������Ҫ��װJDK��Android SDK��Ant����װ���ù��̾��Թ��ˣ���  
һ�о���֮�󣬽���cordovaĿ¼��ʹ��cordova�������androidƽ̨��

        $>cordova platform add android
    
    �������֮�󣬾Ϳ�����platformsĿ¼�¿���������һ��android��Ŀ�ˡ�  
    ���ſ���ʹ��cordova����������һ����Ŀ���п���Ч����
    
        $>cordova run android

7. ��ƽ̨ģ��ͻ��ģʽ  
�������������Ҫȷ��һ����Ŀ����ǲ�ȡ��ƽ̨ģʽ���ǻ��ģʽ��������  
���ǵ��������ڣ�  

    - ��ƽ̨ģʽֻ��cordova��Ŀ��wwwĿ¼��д���룬������ƽ̨�йصĶ�ͨ�������ʵ�֣�ÿ�δ����޸ĺ���Ҫʹ��������롣
    - ���ģʽ�����ĳһ��ƽ̨���п���������androidƽ̨������ֱ����Eclipse�е���cordova��Ŀ��ʵ������platforms�µ�android��Ŀ�������ܿ���ֱ��ͨ��java������ʵ�֡�
    - ��ƽ̨ģʽһ�״��룬ֱ���ܴ���ɸ���ƽ̨��Ӧ�ã�����õ������Ҫ�ṩ��ƽ̨��ʵ�ְ汾����
    
���ˣ�Cordova�Ŀ�����������������ˡ��������coding and coding��



