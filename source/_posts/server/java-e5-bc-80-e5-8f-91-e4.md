---
title: 服务器开发之---Java问题汇总
categories:
  - Server
url: 1444.html
id: 1444
comments: true
date: 2018-08-01 21:11:05
tags:
---

工作十余载，常憾浅尝辄止于各种技术，亦曾屡试专攻于一技，怎奈现实总不能遂人意，为了生活，只能不断前进再前进....

*   游戏服务器开发掌握的技能： 1）语言：Java(Lua,Python Linux Shell etc.) 教程：[Java 教程](http://www.runoob.com/java/java-tutorial.html); 《Java程序员修炼之道》 2)  网络：Netty(Mina etc.) ;SpringMVC(HTTP) Protobuf,Json;教程：[Spring MVC教程](https://www.yiibai.com/spring_mvc/) ;    [Netty教程](https://www.yiibai.com/netty/); [Netty4.0学习笔记系列](https://blog.csdn.net/u013252773?t=1);《Netty实战》 3)  数据库：Mysql(Redis etc.);MyBatis框架 教程：[菜鸟教程（Redis）](http://www.runoob.com/redis/redis-tutorial.html);  [菜鸟教程](http://www.runoob.com/mysql/mysql-tutorial.html)(MySql) 4)  其他：Java NIO;Java多线程；Java并发；书：《Java 7并发编程实战手册》
*   术语： 1）JDK 开发环境 2)  JRE  运行环境 3)  SE    Standard Edition 标准版 用于桌面或简单的服务器应用平台 4)  ME 移动端 5)  J2(Java 2) 过时的术语 用于面述1998~2006的Java版本 6)  SDK 过时的术语 用于面述1998~2006的JDK版本 7)  u(Update) Oracle的术语 8)  NetBeans Oracle的集成开发环境
*   POJO(Plain Ordinary Java Object):简单的Java对象，实际就是普通JavaBeans，是为了避免和EJB混淆所创造的简称。有时可以作为[VO](https://baike.baidu.com/item/VO)(value -object)或[dto](https://baike.baidu.com/item/dto/16016821)(Data Transform Object)来使用
*   PO(Persistent Object)：持久对象,其属性是跟数据库表的字段一一对应的
*   VO(Value Object)：值对象,其属性是根据当前业务的不同而不同的，也就是说，它的每一个属性都一一对应当前业务逻辑所需要的数据的名称。
*   BO(business object)：主要作用是把业务逻辑封装为一个对象。这个对象可以包括一个或多个其它的对象。
*   DTO(Data Transfer Object)：主要用于远程调用等需要大量传输对象的地方
*   transient：类型修饰符，只能用来修饰字段。在对象序列化的过程中，标记为transient的变量不会被序列化。
*   volatile：变量修饰符，只能用来修饰变量。volatile修饰的成员变量在每次被线程访问时，都强迫从共享内存中重读该成员变量的值。
*   NIO:非阻塞输入输出 Java 4 NIO.1     Java 7 NIO.2
*   ORM:Object Relational Mapping 对象关系映射模型（Hibernate)
*   IoC:控制反转，可以把其看做运行时环境，Java中为依赖注入提供的窗口有Guice,Spring和PicoContainer
*   DI:依赖注入，IoC实现的一种方式。代码解耦并增强其可测试性和易读性的通用技术
*   字符串比较不能使用==，要使用strSource.equals(strDest)，不区分大小写比较使用strSource.equalsIgnoreCase(strDest)
*   ide 报错：
    
    objc\[823\]: Class JavaLaunchHelper is implemented in both 
    解决办法：
    点击Ide最上面菜单的Help-Edit Custom Properties，没有这个properties文件的话，会提示创建，然后在里面加上
    idea.no.launcher=true