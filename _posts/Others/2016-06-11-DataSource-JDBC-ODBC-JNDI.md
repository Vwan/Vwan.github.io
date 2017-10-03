---
title: DataSource-JDBC-ODBC-JNDI
date: 2017-09-30 21:05:13 +0800
layout: post
categories:
  - Others
tags:
  - JDBC
---

最近在学习Java Hibernate，对数据库资源访问这块好多概念模糊，所以在此记录一下, 如有错误 盼指正。

## <font color="#0099ff" face="黑体"> Data Source </font>

### It is
>**physically** a set of data stored somewhere like a database;<br>
>**abstractly** it is DataSource object used to get connection to a database;

### <u>访问这些数据需要什么？</u>

Give the Data Source a **NAME** i.e. **DSN** (Data Source Name) which describes informations to access it:

* location
* access right
* driver
* name
* ...

### <u>都有哪些DSN?</u>
- User DSN
- System DSN
- File DSN

### <u>如何访问Data Source呢？</u>
Data Source是存在数据库中的，访问这些数据要用到sql 语句。<br>
应用程序呢，是用特定编程语言比如Java, C#等写的，程序中需要使用Data Source。<br>
这就存在一个问题，这些编程语言和sql语言之间如何talk呢？<br>
显然需要一个翻译机制来实现。<br>
于是便有了ODBC and JDBC...<br>

## <font color="#0099ff" face="黑体"> ODBC</font>

### <u>ODBC is</u>
>     Open Database Connectivity
>     它统一了对各种不同类型数据库的访问接口
>     是应用程序和数据库之间的中间件
>     Language Indepenent (适用any application to access any database)

类比：JVM 使java 可以跨平台使用，ODBC使应用程序可以跨数据库使用（移植数据库方便）

### <u>配置ODBC数据源</u>
ODBC 数据源得配置是OS level的，是不依赖任何数据库系统的，不直接与数据库打交道，与数据库实际操作都是通过各数据库的ODBC Driver来实现的。

如上面三种DSN的配置，在windows中直接到Control Panel -> Administrator -> ODBC 中配置即可。

## <font color="#0099ff" face="黑体"> JDBC</font>
>      Java Database Connectivity
>      纯Java接口规范
>      Java Language specific
>      

### <u>why jdbc when there is already ODBC?</u>
最简单的一句话是，ODBC是使用C语言接口的，java直接调C...sounds not very good.


### <u>连接数据库</u>

Java 程序员可以使用：

+ JDBC-to-ODBC bridge to talk to any ODBC compliant database.
+ JDBC driver directly to talk to database


## <font color="#0099ff" face="黑体"> JNDI</font>

### <u>Why?</u>
既然有了ODBC, JDBC, 那为什么还要JNDI?
一个简单的原因是，程序员需要在程序中使用odbc, jdbc 连接配置信息来访问数据库。这些信息直接写在程序中好吗？如果要用到多个连接信息呢？

### <u>是什么？</u>
> Java naming and directory Interface (JAVA API)
>
> Java 接口规范。

从根本上讲，JNDI是一个目录服务。比如你在手机里查询某人的电话，输入名字手机返回你这个人的电话。这里的手机就相当于JNDI的作用。

在java中可以这么理解：

>     首先给出一个Name(比如Data source name),
>     java 返回你一个Object对象(比如DataSource object)。
>     然后你可以通过这个DataSource object 来获取与数据库的连接，
>     进而作各种数据库的操作。
>     最后关闭数据库连接等。


### <u>Sample Code: （[参考][1])</u>


	`private static void ConnectDBWithJNDI(String jndiName) throws 	NamingException, SQLException {
        Context ctx=new InitialContext();
        Object obj=ctx.lookup(jndiName);
        DataSource ds=(DataSource)obj;
        conn=ds.getConnection();
        conn.setAutoCommit(false);
        conn.commit();
        conn.close();
    }


	private static void ConnectDBWithoutJNDI(String driverName, String jdbcLink) throws ClassNotFoundException, SQLException {
        Class.forName(driverName, true, Thread.currentThread().getContextClassLoader());
        conn = DriverManager.getConnection(jdbcLink);
        conn.setAutoCommit(false);
        conn.commit();
        conn.close();
    }`


### <u>Extension：</u>

当然JNDI不仅限于数据库目录服务，很多诸如LDAP,文件系统等都支持。

类似的目录服务还有 WebService 中的 **UDDI** (Universal Description, Discovery and Integration)
