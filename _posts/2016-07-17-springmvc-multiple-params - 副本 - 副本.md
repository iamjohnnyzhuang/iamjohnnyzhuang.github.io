---
layout: post
title: 通用的Spring MVC Bean自动装配机制
categories: [java]
---

# 前言

我们经常在写项目的时候提交表单，表单信息可能有非常多个。这样我们在Controller里面就要一个一个在方法参数里面罗列出来非常不方便。怎么办呢？

马上有人说，写个Java bean呗。

可是你有没有想过，写一个还好，难道你一有需求就要写一个bean？而且很多的我们只是为了自动装配目的，并没有实际的需求。这样泛滥的bean无论是对于架构还是代码可读性都是非常不友好的。



----



# 解决办法 —— 通用的Query Bean

既然bean是一个键值对，那么我们是不是可以写一个类似的Map，String对应bean的Name，value对应相应的值，这样在后台使用的时候一个一个取出来不就可以了吗？

好的，我们下面看下代码。

前端代码：

``` html
<body> <form action="autoFill" method="post"> 
<label>userName:</label><input type="text" name="params[userName]"/> 
<labele>passWord:</labele>
<input type="text" name="params[passWord]"/> 
<input type="submit"/> </form> </body> </html>
```

这段代码非常的简单，一个有账号和密码的提交表单。注意这里的name。我们给它这样的表述: params[userNmae] 其中这里的params可以是任意的字段，因为我们这里是要告诉后台这是一个Map参数，userName对应一个Key，而input的Value对应一个值。

好，现在再看下后台代码，我截取了最主要的代码，以下是最关键的全能bean代码：

``` java
package com.wskj.controller;

import java.util.HashMap;
import java.util.Map;

/**
* Created by zhuangjy on 2015/12/10.
*/
public class BaseQuery {
protected Map<String,Object> params = new HashMap<String,Object>();

public Map<String, Object> getParams() {
    return params;
}

public void setParams(Map<String, Object> params) {
    this.params = params;
}
}
```

我们用一个Map来获取所有前端的键值对，并且将其存储。

最后看一下Spring Controller中的代码：

``` java
@RequestMapping(value = "/autoFill" , method = RequestMethod.POST)
  public void autoFill(BaseQuery query){
  System.out.println(query.getParams());
  }
```



----



# 结论

 这个小技巧说起来很简单，但是却着实是非常实用，可以避免写上很多不需要的bean，所以以后的项目中还是要多多使用哟！

