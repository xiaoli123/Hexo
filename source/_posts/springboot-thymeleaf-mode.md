---
title: springboot下thymeleaf运行在非严格模式下
date: 2016-10-24
tags:
    - springboot
    - thymeleaf
categories: 
	- springboot
	- thymeleaf
---

> 今天在整合springboot和thymeleaf，于是在下载了hui的hui-admin模板准备写个人员的增删改查。整合过程可以说是举步维艰！其中一个问题就是thymeleaf的语法太严格了。标签不关闭页面就报错。hui算国内出色的ui框架了，但是不关闭的标签页到处都是。当然，严格点是好的，但我们还是要探索下怎么在非严格模式下运行。

google找到了这个问题的答案：
http://stackoverflow.com/questions/28624768/thymeleaf-strict-html-parsing-issue

具体做法是修改：

```
spring.thymeleaf.mode=LEGACYHTML5
```
默认是：
```
spring.thymeleaf.mode=HTML5
```
然后加上依赖：
```
<dependency>
     <groupId>net.sourceforge.nekohtml</groupId>
     <artifactId>nekohtml</artifactId>
     <version>1.9.21</version>
 </dependency>
```
按说这样就应该大功告成了，但我运行还是报错：java.lang.ClassNotFoundException: org.w3c.dom.ElementTraversal

于是找到了如下答案：
https://github.com/spring-projects/spring-boot/issues/3263

具体做法是添加依赖：
```
<dependency>
    <groupId>xml-apis</groupId>
    <artifactId>xml-apis</artifactId>
    <version>1.4.01</version>
</dependency>
```
大功告成！！！