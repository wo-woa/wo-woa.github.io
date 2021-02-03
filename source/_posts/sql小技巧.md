---
title: sql小技巧
date: 2021-1-30 13:50:11
tags: [Sql,Trick]
categories: sql
description: sql开发中遇到的小技巧
---

## sql 排序时空值排最后面
https://blog.csdn.net/dbsjack/article/details/91575680
oracle中的实现方式：
select * from sys_user ORDER BY update_date NULLS LAST;

## sql_mode=only_full_group_by错误解决方案
https://blog.csdn.net/qq_42175986/article/details/82384160
需修改mysql配置文件，通过手动添加sql_mode的方式强制指定不需要ONLY_FULL_GROUP_BY属性
linux:  my.cnf位于etc文件夹下，vim下光标移到最后，添加如下：
```
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```
window则在my.ini文件中添加这一行后重启服务即可