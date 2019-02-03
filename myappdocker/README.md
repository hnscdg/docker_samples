这个例子的目标是为了向大家展示如何在Docker的container里运行Node.js程序。我会先创建一个简单的Node.js web

app，来构建一个镜像。然后基于这个Image运行一个container。从而实现快速部署。

　　由于网络的原因我的Node.js镜像从国内的镜像库下载，而不是Docker Hub。

　　先从国内的镜像网站上pull下一下nodejs镜像。 
　　
> docker pull hub.c.163.com/nce2/nodejs:0.12.2

## 创建Node.js 程序

### 创建package.json,并写入相关信息和依赖
　
> vi package.json

``` docker
{
  "name": "webtest",
  "version": "1.0.0",
  "description": "Node.js on Docker",
  "author": "lpxxn",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.13.3"
  }
}
```
### 创建server.js

> vi server.js

写一个最简单web 这个web基于express框架,返回Hello word.注意我们监听的是8888端口

``` docker
'use strict';

var express = require('express');

var PORT = 8888;

var app = express();
app.get('/', function (req, res) {
  res.send('Hello world\n');
});

app.listen(PORT);
console.log('Running on http://localhost:' + PORT);
```
### 创建Dockerfile

接下来主角上场了创建Dockerfile文件 这个文件是创建镜像所必须的文件

> vi Dockerfile

Docker会依照Dockerfile的内容来构建一个镜像。我先给出完整的代码，再一行一行的给出解释

``` docker
FROM hub.c.163.com/nce2/nodejs:0.12.2

# Create app directory
RUN mkdir -p /home/Service
WORKDIR /home/Service

# Bundle app source
COPY . /home/Service
RUN npm install

EXPOSE 8888
CMD [ "npm", "start" ]
```
我们来一句一句的解释

> FROM hub.c.163.com/nce2/nodejs:0.12.2

FROM是构建镜像的基础源镜像，hub.c.163.com/nce2/nodejs:0.12.2 这个是镜像的名称，也就是我们一开始从国内服务器上拉下来的那个Image。如果本地没有Docker 会自己pull镜像。

```
# Create app directory
RUN mkdir -p /home/Service
WORKDIR /home/Service
```
第一句RUN 用于在Image里创建一个文件夹，将来用于保存我们的代码。

第二句WORKDIR是将我们创建的文件夹做为工作目录。

```
# Bundle app source
COPY . /home/Service
RUN npm install
```

第一句的COPY是把本机当前目录下的所有文件拷贝到Image的/home/Service文件夹下。

第二句的RUN 使用npm 安装我们的app据需要的所有依赖。

```
EXPOSE 8888
```
由于我们的web app监听的是8888端口，我们把这个端口暴露给主机，这样我就能从外部访问web了。

```
CMD [ "npm", "start" ]
```
这个我相信我不用解释你也能看出来他是做什么的。运行npm start命令，这个命令会运行 node service.js来

启动我们的web app。

### 构建Image

在你Dockerfile文件所在的目录下运行下面的命令来构建一个Image.

> docker build -t mynodeapp .

别忘了最的的那个点

###  运行镜像

> docker run -d -p 8888:8888 [镜像ID的前三个字符就可以了，如ac5]

-d 表明容器会在后台运行，-p 表示端口映射，把本机的8888商品映射到container的8888端口这样外网就能通过本机的8888商品访问我们的web了。

后面的ac5是我们Image的ID因为前3个就已经能定位出这个Image所以我就没有把后边的再写出来。

打印log  7370就是我们的Container ID，和Image ID一样，你也可以全写出来，我比较懒就写前4位，已经足够标识出这个Container了 

> docker logs 7350

如果你想到Container里可以执行下面的命令，进入到里边后就可以像操作普通的linux 一样。如果想退出可执行exit命令。

> docker exec -i -t 7350 /bin/bash

###  测试

我们先通过curl 看能不能访问我们的web。

> curl -i localhost:8888