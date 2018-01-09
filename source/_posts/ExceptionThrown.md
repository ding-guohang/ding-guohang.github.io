---
title: Exception thrown in catch and finally clause 
date: 2018-01-09 15:41:40
tags:
- exception
- 栈
categories:
- 一些有趣的问题
---

*since: 2016-12-14 21:32:51* 

*一个有趣的问题，如果在catch或是finally里面抛出异常，会发生什么呢*
<!--more-->

# Question
```Java
   public static void main(String[] args) throws Exception {
        try {
            System.out.println("begin");
            tryThrow();
        } catch (Exception e) {
            throw new Exception("ex3");
        } finally {
            System.out.println("main throw");
            throw new Exception("ex4");
        }
    }

    private static void tryThrow() throws Exception {
        try {
            throw new Exception("ex1");
        } catch (Exception e) {
            //ignore
        } finally {
            System.out.println("try throw");
            throw new Exception("ex2");
        }
    }
```

# Answer
```bash
begin
try throw
main throw
Exception in thread "main" java.lang.Exception: ex4
	at line;
```
* 每当抛出一个异常，都会离开当前代码块，例如 try -> catch -> finally
* 每当抛出一个异常，上一个异常会立刻结束，并展开新异常的堆栈

# Resources
[Exception thrown in catch and finally clause -- stackOverFlow](http://cache.baiducontent.com/c?m=9f65cb4a8c8507ed4fece76310579135480ddd276b97844b22918448e435061e5a25a4ec66644b598f84616604aa425ce0f72b257d5277f5dd93d516cabbe46e75c87a712c0b8637498f0eafbd17768771ca4de9d94abbece732e5fa898f8c0a0d8144050cd1aed81d4143de2aad5467ece0c308481a07ba9c6b39be02217e882331e21ca5b36c304c&p=82759a46d0c450f20be29666166486&newp=9f66c115d9c047a819be9b7c455790231601d13523808c0a3b8fd12590655a55113d8eff7062515f8e99736300a84856e8f33371300327b19bc38c4dd8be866e42c970767f4bda1753&user=baidu&fm=sc&query=throw+inside+finally&qid=8249427500027d5c&p1=1)
[Oracel docs]( http://docs.oracle.com/javase/specs/jls/se8/html/jls-14.html#jls-14.20.2)
