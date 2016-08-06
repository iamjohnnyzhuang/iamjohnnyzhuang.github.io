---
layout: post
title: 排序算法看这篇就够
categories: [algorithm]
---

# 前言

简单总结下思路。推荐个很好的可视化算法学习网站，具体的思想我觉得看那么多文字不如看一个可视化动画，[点击跳转算法可视化](http://www.cs.usfca.edu/~galles/visualization/)。



----



# 冒泡排序

冒泡排序是最简单的排序方法，通过比较并且交换相邻两个元素即可。时间复杂度为O(n^2)。

``` java
 public static void sort(int[] t){
    for(int i=0;i < t.length;i++){
        for(int j=0;j < t.length-1;j++){
            if(t[j] > t[j+1]){
                int temp = t[j];
                t[j] = t[j+1];
                t[j+1] = temp;
            }
        }
    }
}
```



---



# 选择排序法

简单的排序算法之一，在一系列数字中找出最小的放在第0个位置，然后继续在剩余的元素中找出最小的放在第一个位置…… 时间复杂度O(n^2)

``` java
    public static void sort(int[] a){
        for(int i=0;i<a.length;i++){
            int minIndex = 0;
            int min = Integer.MAX_VALUE;

            for(int j=i;j<a.length;j++){
                if(a[j] < min){
                    min = a[j];
                    minIndex = j;
                }
            }

            int temp = a[i];
            a[i] = min;
            a[minIndex] = temp;
        }
    }
```



----



# 插入排序算法

对于每个元素假设其前面的n-1个元素都是已经排好序的。那么对于这个元素的任务就是去找出他自己适合的那个位置。从第一个元素开始查找。时间复杂度O(n^2)

``` java 
  public static void sort(int[] t){
    for(int i=1;i<t.length;i++){
        int temp  = t[i];
        int j = i-1;
        for(;j>=0 && t[j]> temp;j--){
            t[j+1] = t[j];
        }
        t[j+1] = temp;
    }
}
```



----



# 快速排序

速度几乎是最快的快速排序。时间复杂度为O(nlogn)。利用分治思想，虽然理解不难，但是要默写出来倒是有点难度。 理解问题推荐[这篇博客的『挖空』思想](http://blog.csdn.net/morewindows/article/details/6684558)：

``` java
    public static void sort(int l,int r,int[] s){
    if(l<r){
    int i = l,j = r,x = s[l];
    while(i<j){
        while(i<j&&s[j]>=x) //从右向左查找比x小的数
            j--;
        if(i<j) //找到一个比X小的数
            s[i++] = s[j];

        while(i<j && s[i] < x) //从左向右查找比x大的数字
            i++;
        if(i<j) //找到一个比x大的数字
            s[j--] = s[i];
    }
    s[i] = x;
    sort(l,i-1,s);//递归调用
    sort(i+1,r,s);
    }
}
```



----



# 归并排序

假设待排序列为

1,13,2,24,26,28,27,15.

我们考虑把它分成两部分并且已排序：

前一部分：1，13，24，26

后一部分：2，15，27，38

所谓归并就是要把这两部分合并成一部分。那么我们的目标很显然就是要依次比较两部分对应的元素并且排列，这就要借助另外一个临时数组了。下面是比较的顺序：

第一次，比较1,2  取小者，那么结果集为1

第二次，比较13，2 取小者，那么结果集为1，2

第三次，比较13，15 取小者，那么结果集为1，2，13

……

第七次，前一部分已用完，直接将后部分剩余的加入即可，结果集为

1,2,13,15,24,26,27,38

ok,那么排序就完成了。现在的问题是，我们前面说的的前一部分和后一部分是已经排过序的，那么他们是怎样排序呢？显然，方法和上面的是一样的，即对于前一部分我们再把它分成两部分，然后依次归并，对于前一部分的前一部分我们依次这样递归执行就可以了，后一部分也是。这就利用了算法中的分治思想，即分而治之，将一个整体划分成小部分依次处理。

代码如下：



``` java
	public static void sort(int[] a,int[] temp,int left,int right){
		if(left < right){ 
			int center = (left + right) /2; 
			sort(a,temp,left,center); //对左半部分排序
			sort(a,temp,center+1,right); //对右半部分排序
			merge(a,temp,left,center + 1,right); //合并排序好的两部分
		}
	}
	
	public static void merge(int[] a,int[] temp,int leftBeg,int rightBeg,int rightEnd){
		int leftEnd = rightBeg - 1; //左半部分结束index
		int tempBeg = leftBeg; //标志本轮merge结果对tempBeg的填充位置
		int numElements = rightEnd - leftBeg + 1; //元素总个数
		
		while(leftBeg <= leftEnd && rightBeg <= rightEnd){
			//对前半部分的每个元素 依顺序 和后半部分对应的比较选择小的
			if(a[leftBeg] <= a[rightBeg])
				temp[tempBeg ++] = a[leftBeg++];
			else
				temp[tempBeg ++] = a[rightBeg++];
		}
		
		//将剩余的前（或者后）半部分copy进去结果集
		while(leftBeg <= leftEnd) //左半部分集剩余 
			temp[tempBeg++] = a[leftBeg++];
		while(rightBeg <= rightEnd) //右半部分集剩余
			temp[tempBeg++] = a[rightBeg++];
		
		//将结果集copy回a
		for(int i=0;i<numElements;i++,rightEnd--)
			a[rightEnd] = temp[rightEnd];
		
	}
```





