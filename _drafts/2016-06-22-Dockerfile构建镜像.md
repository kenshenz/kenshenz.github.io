---
layout: post
date : 2016-06-22 09:45
title : ［转］Docker本地私有仓库
category : lessons
tags : [Docker]
postid : 1466559932204
---
{% include JB/setup %}

通过Dockerfile，构建自己的Docker镜像。

### 编写Dockerfile

新建Dockfile文件

```
vi Dockerfile
```

Dockerfile内容如下

```
FROM centos:7
MAINTAINER kenshenchan@gmail.com
CMD /usr/bin/bash
```

在Dockerfile目录下执行构建

```
docker build -t centos:7 .
```





