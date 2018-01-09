---
title: SPI
date: 2018-01-09 15:52:59
tags:
- SPI
- dubbo
categories:
- 一些有趣的问题
---

*since: 2016-12-14 21:16:05*

## 什么是SPI
**SPI：Service Provider Interface**
<!--more-->

**扩展点接口**

普通开发人员可能不熟悉，因为这个是针对厂商或者插件的。在java.util.ServiceLoader的文档里有比较详细的介绍。究其思想，其实是和"Callback"差不多。“Callback”的思想是在我们调用API的时候，我们可以自己写一段逻辑代码，传入到API里面，API内部在合适的时候会调用它，从而实现某种程度的“定制”。

典型的是Collections.sort（List list,Comparator c）这个方法，它的第二个参数是一个实现Comparator接口的实例。我们可以根据自己的排序规则写一个类，实现此接口，传入此方法，那么这个方法就会根据我们的规则对list进行排序。

把这个思想扩展开来，我们用SPI来重新实现上面的例子。客户把自己的排序规则写成一个类，并且打包成Jar文件，这个Jar文件里面必须有META-INF目录，其下又有services目录，其下有一个文本文件，文件名即为接口的全名：java.util.Comparator
```
--META-INF
--services
--java.util.Comparator

//文件内容只有一行：
com.company1.ComparatorProvider
```

## API & SPI

### *really cool description*

* the *API* is the description of classes/interfaces/methods/... that you +call and use+ to achieve a goal and
the *SPI* is the description of classes/interfaces/methods/... that you +extend and implement+ to achieve a goal

* Put differently, the API tells you what a specific class/method does for you and the SPI tells you what you must do to conform.

* Usually API and SPI are separate. For example in JDBC the Driver class is part of the SPI: If you simply want to use JDBC, you don't need to use it directly, but everyone who implements a JDBC driver must implement that class.

* Sometimes they overlap, however. The Connection interface is both SPI and API: You use it routinely when you use a JDBC driver and it needs to be implemented by the developer of the JDBC driver.

### *deference between SPI and API*

![API and SPI](/img/SPIandAPI.jpg)

### *example for SPI and API as JNDI*

![SPI](/img/SPI.jpg)


## JDK的SPI加载类

ServiceLoader.java

## Dubbo的SPI注解

### 注解
```java
/*
 * Copyright 1999-2011 Alibaba Group.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package com.alibaba.dubbo.common.extension;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 扩展点接口的标识。
 * <p />
 * 扩展点声明配置文件，格式修改。<br />
 * 以Protocol示例，配置文件META-INF/dubbo/com.xxx.Protocol内容：
 * 由 com.foo.XxxProtocol com.foo.YyyProtocol
 * 改成使用KV格式 xxx=com.foo.XxxProtocol yyy=com.foo.YyyProtocol
 *
 * 原因：
 * 当扩展点的static字段或方法签名上引用了三方库，
 * 如果三方库不存在，会导致类初始化失败，
 * Extension标识Dubbo就拿不到了，异常信息就和配置对应不起来。
 *
 * 比如:
 * Extension("mina")加载失败，
 * 当用户配置使用mina时，就会报找不到扩展点，
 * 而不是报加载扩展点失败，以及失败原因。
 *
 * @author william.liangf
 * @author ding.lid
 * @export
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
public @interface SPI {

    /**
     * 缺省扩展点名。
     */
    String value() default "";

}
```

### 加载类

ExtensionLoader.java
