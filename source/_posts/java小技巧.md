---
title: java小技巧
date: 2021-1-14 17:14:11
tags: [Java,Trick]
categories: Java
description: 开发中遇到的小技巧
---



### **快速创建List带初始值**

**1. 使用Collections.addAll()方法，前提还是需要手动 new ArrayList**

ArrayList<String> s = new ArrayList(); Collections.addAll(s,"1","2","3")

**2. 使用Arrays.asList(...args) 直接返回一个List**

List<String> s = Arrays.asList("1","2","3")

**3. 如果引入了Guava的工具包，可以使用他的Lists.newArrayList(...args)方法**

List<String> list = Lists.newArrayList("1","2","3")

**4. 如果是Java9，可以使用自带的List类**

List<String> s = List.of("1","2","3")

推荐使用第一种，第二种在java8有bug

使用addAll方法合并两个



### 快速创建Map带初始值

Map<String, Integer> map = new LinkedHashMap<String, Integer>() {{
    put("city", 1);
    put("stationNum", 1);
}}



### parseInt和valueOf的区别

parseInt方法返回的是int基本类型，valueOf方法返回的是Integer的包装类型

valueOf方法实际上是调用了parseInt方法，也就是说，如果我们仅仅只需要得到字符串类型字符数值对应的整数数值，那我们大可不必调用valueOf，因为这样得到整形数值之后还要做一个装箱的操作，将int封装为Integer。



### springboot依赖的版本号统一管理

[参考](https://www.cnblogs.com/ld-mars/p/11714151.html)

第一种常用的方式

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>1.5.4.RELEASE</version>
</parent>
```

第二种使用dependencyManagement

```
<dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-dependencies</artifactId>
        <version>1.5.4.RELEASE</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
</dependencyManagement>
```

其中的spring-boot-dependencies可以替换为spring-boot-starter-parent

