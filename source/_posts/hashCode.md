---
title: hashCode
date: 2018-01-09 16:19:26
tags:
- hashCode
- jdk
categories:
- 一些有趣的问题
---

*since: 2016-12-18 10:22:20*
*hashCode一般如何实现？为什么要这么实现？其实我也不知道......*
<!--more-->

# xxx
其实一直觉得31这个挺碍眼的，不知道是处于什么考虑

```Java
// String中使用了这样的方式来生成hashCode
    public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            char val[] = value;

            for (int i = 0; i < value.length; i++) {
                h = 31 * h + val[i];
            }
            hash = h;
        }
        return h;
    }

// IDEA自动生成的hashCode方法也使用了类似的实现 
    public int hashCode() {
        int result = status;
        result = 31 * result + (message != null ? message.hashCode() : 0);
        result = 31 * result + (data != null ? data.hashCode() : 0);
        return result;
    }


//"s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]"
```


# yyy
其实都是在使用31作为基数，把值转换为31进制，来获得更大的唯一性

1、 31 * N 会被编译器优化为=> 左移5位 - N
2、 质数能更好地保证与其他数相乘之后的唯一性
3、 研究人员发现，31能更好的分配key，减少碰撞，但是不知道为什么

# zzz
[参考文章](https://computinglife.wordpress.com/2008/11/20/why-do-hash-functions-use-prime-numbers/) => 这个人废话了一大堆，然后说他也不知道为什么使用31。。。

{% cq %}Researchers found that using a prime of 31 gives a better distribution to the keys, and lesser no of collisions. 
No one knows why, the last i know and i had this question answered by Chris Torek himself, who is generally credited with coming up with 31 hash, on the C++ or C mailing list a while back. {% endcq %}

不知道是不是我翻译有问题。。反正这句话我没找到答案
