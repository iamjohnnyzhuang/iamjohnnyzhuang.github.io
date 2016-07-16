---
layout: post
title: SQL注入看这篇就够
categories: [database]
---

# 前言

SQL注入是一种非常出名的安全防范问题。原理很简单，但是问题是在客户端是怎么做到操作/修改后台SQL语句的呢?



# 快速理解SQL注入原理

我们以登录账号为起始点。

通常，登录账号要求我们输入的是账号(userName)、密码(passWord)

输入完点击确定后后台这里用的java将产生一条sql语句。

假如我们用Java原声的产生sql语句方式，而不是用 PreparedStatement，代码是这样写的：

```java
String sql = "select * from table where user_name='" + userName +"' and pass_word='" + passWord + "'";
```

将会生成一条语句（这里加入我输入账号:zhanghao 密码:123)

```sql
select * from table where user_name ='zhanghao' and 'pass_word=123'
```

这样看起来一切很ok。

但是如果用户输入的不是zhanghao，而是这样的’ or 1=1 –呢？

这样产生的sql语句就应该是这样的

```sql
select * from table where user_name ='' or 1=1 -- 'and pass_word='123'
```

这样前面的user_name判断条件无效。 并且加入– 注释后面的所有语句！攻击者堂皇的进入系统。

可能此人更心怀不轨，不是要进入系统而是破坏。所以他可能会写：’;DROP DATABASE (DB NAME) —

这样翻译的sql语句就是：

```sql
select * from table where user_name='';DROP DATABASE (DB NAME) --  'and pass_word='123'
```

**一条语句被拆解为两条数据库瞬间爆炸。**



---



# 防止SQl注入手段

很多语言已经帮我们做好了对应的API。比如Java我们用PreparedStatement

```java
   String sql= "select * from users where username=? and password=?;
   PreparedStatement preState = conn.prepareStatement(sql);
   preState.setString(1, userName);
   preState.setString(2, password);
   ResultSet rs = preState.executeQuery();
```

或者，我们可以用正则，把 有 单引号(‘)，分号(;) 和 注释符号(–)的语句给替换掉来防止SQL注入

Java

```
   return str.replaceAll(".*([';]+|(--)+).*", " ");
```



----



# 结论

最近看学校人做的网站说是被黑了。数据库全没了，但是被黑了是怎么定义呢？

显然这是仅仅是简单的SQL注入，清空数据库。

SQL注入被称作永不过时的黑客技术，虽然很简单但是却一直盛行，所以我们在做项目的时候一定要考虑到SQL注入！