---
layout: post
title: 利用Alfred实现jekyll自动化
categories: [other]
---

# 前言

用jekyll来创建自己的技术博客想必是很多程序老兄共同的喜好吧。但是用了jekyll发现有一个问题，就是当我要写一篇文章的时候我还得自己跑到category目录下去创建一个md文件并且要写一个YAML头。更要命的是还得给这个文件按一定规则命名。这不就失去了jekyll“免管理”的初衷了吗！除此之外，我们知道jekyll里面要代码高亮的话必须得这样写{% highlight java %} {% endhighlight %}相当的繁琐。还很有可能出错

作为一名程序猿，遇到这种情况怎么能沉得住气。

于是我就借助Mac OS上的神器Alfred自己写了两个简单的脚本，分别解决上面两个问题。至于Alfred是什么，如果有朋友还不清楚，[可以点击此处先进行了解]。


## 一、利用Alfred自动生成highlight语句

先来个简单一点的。我写了一段简单python代码，输入框让用户输入高亮的语言比如javascript，然后回车后自动生成highlight语句并且黏贴到黏贴板上。

如图所示：

![](https://iamjohnnyzhuang.github.io/public/upload/1.png)

这样我在文章中直接黏贴就ok了。代码实现非常简单一段简单的python代码。

{% highlight python %}

# coding:utf-8

import sys

reload(sys)
sys.setdefaultencoding('utf8')
sys.path.append("/Users/johnny/Library/Application Support/Alfred 2/Alfred.alfredpreferences/workflows/")
from workflow import Workflow

def main(wf):
    query = wf.args[0].strip()
    if not query:
    	wf.add_item('generating..')
    	wf.send_feedback()
    	return 0
    #这里百分号没有写出来，否则jekyll会报错。
    res = "{ highlight " + query + " } { endhighlight }"
    wf.add_item(
    	title="highlight text",subtitle=res,arg=res,valid=True)
    wf.send_feedback()

if __name__ == '__main__':
	wf = Workflow()
	sys.exit(wf.run(main))
 {% endhighlight %}



具体代码可以参考我的[GitHub](https://github.com/iamjohnnyzhuang/my-alfred-workflow/blob/master/jekyllHighLight/highlight.py)

----



## 二、利用Alfred实现自动创建md文件并且命名

这个功能思路也很简单，首先根据用户输入，要求输入文章名以及类别名。怎样判断是否输入结束了呢？我们让输入文章名与类别名中间需要一个空格比如：学习Alfred alfred

同时，系统去遍历category这个目录获取所有的目录名称。如果检测到输入存在空格同时最后一个单词存在于category目录里面那么可以判定为输入结束。

输入结束后，脚本会在用户设置的目录根据输入的文件名、类别创建文件并且填充如YAML头信息。同时调用shell打开文件。
![](https://iamjohnnyzhuang.github.io/public/upload/2.png)

代码也是挺简单的。

{% highlight python %}

#coding:utf-8
import sys
import os
import time
reload(sys)
sys.setdefaultencoding('utf8')
sys.path.append("/Users/johnny/Library/Application Support/Alfred 2/Alfred.alfredpreferences/workflows/")
from workflow import Workflow

def main(wf):
    categoriesPath = '/Users/johnny/GitHub/jekyllBlog/category/'
    src = "/Users/johnny/GitHub/jekyllBlog/_posts/"
    ISOTIMEFORMAT='%Y-%m-%d'
    categories = os.listdir(categoriesPath) 
    query = wf.args[0].strip()
    if not query or ' ' not in query or query.split( )[1] + '.md' not in categories:
        #创建类别提示
        tip = ''
        for c in categories:
            if c.split('.')[0]:
                tip = tip + c.split('.')[0] + ' '
        wf.add_item(tip)
    	wf.send_feedback()
    	return 0
    #获取当前日期
    cur = time.strftime(ISOTIMEFORMAT,time.localtime())
    input = query.split( )
    name = input[0]
    category = input[1]
    filePath = src + cur + '-' + name + '.md'
    file = open(filePath,'w')
    head = "---\nlayout: post\ntitle: " + name + "\ncategories: [" + category + "]\n---\n"
    file.write(head)
    file.close()
    shell="open " + filePath
    os.system(shell)
    wf.add_item(
    	title="创建成功",subtitle="successful",arg="successful",valid=True)
    wf.send_feedback()

if __name__ == '__main__':
	wf = Workflow()
	sys.exit(wf.run(main))



 {% endhighlight %}

具体代码可以参考我的[GitHub](https://github.com/iamjohnnyzhuang/my-alfred-workflow/blob/master/JekyllPostGenerate/JekyllPostGenerate.py)



# 写在最后

实现都比较简单，原来我写的很生疏的Python哈毕竟很少写。

至于Alfred怎么用，怎么设置Script这里暂时就不详细讲啦，毕竟不是这篇blog的主题。当然如果有人看的话再写哈。



