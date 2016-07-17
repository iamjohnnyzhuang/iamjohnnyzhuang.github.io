---
layout: post
title: 有效防止F5刷新重复提交
categories: [java]
---

# 前言

我们填写完一个表单后点击submit提交后，假设返回了填写的页面。这个时候无论你的form方法是get还是post，你点击F5刷新，都会重复此前的操作！至于原理为什么，因为比较长这里就不阐述了，网上有一大把。这就存在一个问题了，我们通常是通过form来提交数据到数据库的，但是这样就会引起插入重复的数据！有人说可以通过数据库判断是否有重复，但是毕竟这样的操作能减少数据库操作还是尽量减少况且很多情况下允许重复数据呢。





----



# 解决办法 ——  前端综合后端验证



我们先看看问题的后台代码，这里数据库操作省略了直接写一个System.out.println模拟。

``` java
    @RequestMapping(value = "/demo" , method = RequestMethod.POST)
    public String preventFormCommit(String orderName){
        System.out.println("收到订单:" + orderName +"这里将进行数据库运算");
        return null;
    }

    @RequestMapping(value = "/demoindex")
    public String gotoDemo(){
        return "demo";
    }
```

很简单的Spring MVC代码。

再看看前端。

``` html
<html>
<head>
    <title>防止F5重复提交</title>
</head>
<body>
    <form method="post" action="/demo">
        <input type="text" name="orderName"/>
        <input type="submit"/>
    </form>
</body>
</html>
```

依然很简单，这种情况下F5按下去打印与第一次点击submit一样的信息。说明重复提交了。

ok，现在进入正题讲讲解决办法:



**首先我们在进入目标页面的时候在session内用uuid生成一个token标志并且将其保存在页面的一个hidden input域，然后我们提交表单的时候，后台可以接收这个input值，并且取出session中的token值，比较，如果session的token存在并且与hidden域的相等，那么执行操作，并且在操作完成后删除session中该属性。**

**这样我们再重复提交的时候这个时候虽然input域传值过来，但是session已经没了这个属性，所以我们可以判断什么都不做。** 



好看下代码：

``` java
    @RequestMapping(value = "/demo" , method = RequestMethod.POST)
    public String preventFormCommit(String orderName,String judgeDuplicate,HttpSession session){
        String token = (String)session.getAttribute("token");
        if(StringUtils.isNotBlank(token) && token.equals(judgeDuplicate)) {
            System.out.println("收到订单:" + orderName + "这里将进行数据库运算");
            //操作完成后删除session中的对象
            session.removeAttribute("token");
        }
        return null;
    }

    @RequestMapping(value = "/demoindex")
    public String gotoDemo(HttpSession session){
        UUID uuid = UUID.randomUUID();
        session.setAttribute("token",uuid.toString());
        return "demo";
    }
```

只需在进入页面加一个session，并且在提交订单的时候判断session值是否存在以及是否等于hidden域。

``` html	
<html>
<head>
    <title>防止F5重复提交</title>
</head>
<body>
    <form method="post" action="/demo">
        <input type="text" name="orderName"/>
        <input type="submit"/>
        <input type="hidden" name="judgeDuplicate" value="${token!""}"/>
    </form>
</body>
</html>
```

增加一个隐藏域其值等于session的token。

这样你再怎么刷新，也只能提交一次了！



## 为什么要设置一个隐藏域？

这个问题我一开始没想明白，我认为，进入页面的时候设置一个session并且再token设值，添加的时候把这个值删掉。然后这样我们再按F5的时候就没办法重复提交了（因为这个时候判断session为空）。我觉得这样就ok了，设置hidden域感觉没任何必要。

然而简直是图样图森破，对于一般用户这样当然是可以的。但是对于恶意用户呢？假如恶意用户开两个浏览器窗口(同一浏览器的窗口公用一个session)这样窗口1提交完，系统删掉session，窗口1停留着，他打开第二个窗口进入这个页面，系统又为他们添加了一个session，这个时候窗口1按下F5，那么直接重复提交！

**所以，我们必须得用hidden隐藏一个uuid的token，并且在后台比较它是否与session中的值一致，只有这样才能保证F5是不可能被重复提交的！**



----



# 结论

效率优化的一个重要手段就是减少数据库操作。因此可以的话我们可以将这些校验设置在客户端java代码以及html上面！

