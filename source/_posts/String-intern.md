---
title: String#intern
date: 2018-01-10 11:40:51
tags:
- String
- jdk
categories:
- 一些有趣的问题
---

*String#inter()是什么？*
*它在jdk6-7-8中是否有什么区别呢？*
*应该使用或是如何呢？*
*这是一个秘密*

未完待续
<!--more-->


# Talk is cheap, show me the code
```Java
    public static void main(String[] args) {
        String s1 = new StringBuilder("计算机").append("软件").toString();
        String s2 = new StringBuilder("ja").append("va").toString();

        System.out.println("Answer One: " + s1 == s1.intern());
        System.out.println("Answer Two: " + s2 == s2.intern());

        System.out.println("Answer Three: " + "计算机麻烦呀" == "计算机麻烦呀".intern());
    }
```

# Console
```bash
# in jdk6
Answer One: false
Answer Two: false
Answer Three: true ? 这个还需要回去在jdk6上验证一下

# in jdk7+
Answer One: true
Answer Two: false
Answer Three: true 
```

# Infomation

