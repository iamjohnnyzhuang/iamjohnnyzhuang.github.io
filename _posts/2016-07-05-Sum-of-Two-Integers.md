---
layout: post
title: Sum of Two Integers
categories: [leetcode]
---

# 题目
Calculate the sum of two integers a and b, but you are not allowed to use the operator + and -.
Example:  
Given a = 1 and b = 2, return 3.

题目的意思就是简单的A + B Problem 但是这里要求不能使用 ‘+’ 操作。

# 解法一 使用API （作弊法强烈不推荐）
Leetcode的检查机制并不严格。使用了API: Integer.sum(a,b)即可通过。但是跟进源代码，Integer.sum无非就是 return a + b; 换汤不换药。

{% highlight java%}  
/**  
 * Created by johnny on 16/6/13.  
 */  
public class Solution {  
  
    public int getSum(int a, int b) {  
        String a1 = Integer.toBinaryString(a);  
        String b1 = Integer.toBinaryString(b);  
  
        return Integer.sum(a,b);  
    }  
  
    public static void main(String[]() args) {  
        System.out.println(new Solution().getSum(1,2));  
    }  
}  

{% endhighlight %}


---- 

# 解法二 位运算（推荐）
既然不能使用 ‘+’ 操作符显然就是要使用位运算了。
这里我们必须得先了解一下怎样用位运算来模拟加法。
首先我们观察下面几个一位数的位运算。
* 0^0 = 0
* 0^1 = 1
* 1^0 = 1
* 1^1 = 0
由此可知，位运算的异或操作相当于我们理解的’+’操作。

但是涉及到加法，我们还应该考虑到的是进位操作。考虑 4(100) 加 5(101)的情形。 4 + 5 的最终结果为 9 (1001)。

4 ^ 5 = 001;这相当于是最终加法的结果，但是并没有进位（进位。进位怎么来呢？
我们观察下面几个数字的位运算。
* 0&0=不进位
* 0&1=1不进位
* 1&0=0不进位
* 1&1=1进位
所以可以看出来&操作和我们直观上理解的进位是相符的。
4 & 5 = 100;
因此是进位因此我们再向左移动一位。即 (4 & 5) <<1 = 1000;最后把进位的和结果相加(^)即可(1000 ^ 001 = 1001)。

{% highlight java%}

/**  
 * Created by johnny on 16/6/13.  
 */  
public class Solution {  
  
    public int getSum(int a, int b) {  
        while (b != 0){  
            int c = a ^ b;  
            b = (a&b) << 1;  
            a = c;  
        }  
        return a;  
    }  
  
    public static void main(String[]() args) {  
        System.out.println(new Solution().getSum(1,2));  
    }  
}

{%endhighlight%}

