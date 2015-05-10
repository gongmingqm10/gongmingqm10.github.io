---
layout: post
title: "Config spring mvc web project"
date: 2015-04-12 23:33:50 -0500
description: 
published: false
comments: true
categories: web

---

到目前为止，接触过两到三个小型的Spring Web项目，前几日朋友问起怎么搭建Spring Web MVC项目，才想好要写一篇博客来记载如何从零开始搭建一个完整的Spring项目，这里使用的Spring主要还是3.x，关于4.x的我们会在其他文章中再详细叙述。

## Spring Web Content with Spring MVC

既然从零开始，那我们就从空文件夹开始，首先还是习惯在Git上创建一个[gongmingqm10/spring-mvc-sample](http://github.com/gongmingqm10/spring-mvc-sample)的项目。clone到本地之后就是一个空的文件夹。一般来说，官网上都有很多参考的资料，可以告诉我们怎么使用这个框架搭建完整的项目。来到[Spring IO](http://spring.io/guides)，才发现资料真的多，于是就选择了[Spring Web Content with Spring MVC]作为Guide从零开始搭建。还是使用Gradle构建。

###1. 创建文件夹结构
```
mkdir -p src/main/java/hello

XiaoMing:spring-mvc-sample minggong$ tree .
.
├── README.md
└── src
    └── main
        └── java
            └── hello

```
###2. 创建build.gradle文件
`build.gradle`

```
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.2.3.RELEASE")
    }
}

apply plugin: 'java'
apply plugin: 'idea'
apply plugin: 'spring-boot'

jar {
    baseName = 'gs-serving-web-content'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.7
targetCompatibility = 1.7

dependencies {
    compile("org.springframework.boot:spring-boot-starter-thymeleaf")
    testCompile("junit:junit")
}

task wrapper(type: Wrapper) {
    gradleVersion = '2.3'
}
```
###3. 创建 Web Controller

`src/main/java/hello/GreetingController.java`

```
package hello;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

@Controller
public class GreetingController {
    @RequestMapping("/greeting")
    public String greeting(@RequestParam(value="name", required=false, defaultValue="World") String name, Model model) {
        model.addAttribute("name", name);
        return "greeting";
    }

}
```
###4. 创建View文件

`src/main/resources/templates/greeting.html`

```
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Getting Started: Serving Web Content</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
    <p th:text="'Hello, ' + ${name} + '!'" />
</body>
</html>
```
###5. 创建可运行的Application

