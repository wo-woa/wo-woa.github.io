---
title: web小技巧
date: 2021-1-15 15:18:11
tags: [Web,Trick]
categories: Web
description: web开发中遇到的小技巧
---



## 快速删除node_modules

[参考](https://blog.csdn.net/robin_star_/article/details/80293512)

解决方法：使用npm的一个名为rimraf的模块进行删除

官方描述：The UNIX command rm -rf for node，即node环境下模拟unix或者linux下的rm -rf（强制删除命令）

安装（推荐全局安装）：  npm install -g rimraf

使用： rimraf node_modules

idea等软件可以在file type中过滤掉node_modules



## 正则使用

**创建**

```javascript
let reg1 = /\d+/g
let reg2 = new RegExp("\\d+")
```

一般使用第一种，一定要带上g，g表示为全局模式

**模式**

g：表示全局（global）模式，即模式将被应用于所有字符串，而非在发现第一个匹配项时立即停止；

i：表示不区分大小写（case-insensitive）模式，即在确定匹配项时忽略模式与字符串的大小写；

m：表示多行（multiline）模式，即在到达一行文本末尾时还会继续查找下一行中是否存在与模式匹配的项。

**常用**

搜索所有符合的值

```javascript
var str='131dsad1231sad1';
var reg=/\d+/g;
var res = str.match(reg);
while( res = reg.exec(str))
{
 console.log(res[0]);
}
```

如果不加上g就会一直进行匹配不会停止









