---
layout: post
title: "Begin to use SpringBoot"
date: 2015-01-22 18:00:09 +0800
description: How to use spring boot, config spring boot project
comments: true
categories: web

---

The time since I worked as a web developer, I use spring 3.x for most of my personal project. Because java is mostly
been used. Those who worked with me can easily be participant in the project. While the more I use it, the more I'm suffered
with the complex config of the spring project. Additionally it's not easy to create a scaffolding Spring project, especially
when I'm so accustomed to rails scaffold.
 
It's time for us to trial a new development pattern to make our work be more graceful. Luckily spring4.x appeared, and I
began to knew the [Spring Boot](http://projects.spring.io/spring-boot/) project.

So, do not wait any more, just try it. I following the [official guides](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/)

### Say hello to Spring Boot
I worked in a Mac system. It's easy to install SpringBoot CLI by brew:

```
$ brew tap pivotal/tap
$ brew install springboot
```
Homebrew will install spring to `/usr/local/bin`, when I try to use spring to start a simple spring boot project. Some trick
error stuck me until I realized the name `spring` is collided with the gem `spring`. But `/usr/local/bin/spring` exists in my mac,
thus I use `/usr/local/bin/spring` to start the demo project. The following shows how to say hello to spring boot.

* Create the app.groovy first:

```
@RestController
class ThisWillActuallyRun {

    @RequestMapping("/")
    String home() {
        "Hello World!"
    }

}
```

* Use spring to run it: `/usr/local/bin/spring run app.groovy`
* Open `http://localhost:8080/` in your browser. You can see the `Hello World!` in the page.

### Build a real project
After we finished the hello-world project, we can move to next step to build a real web project.
I use gradle to manage my dependency in our project. This is my build.gradle file:

```
buildscript {
    repositories { jcenter() }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.2.2.BUILD-SNAPSHOT")
    }
}

apply plugin: 'java'
apply plugin: 'spring-boot'

repositories { jcenter() }
dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    testCompile("org.springframework.boot:spring-boot-starter-test")
}
```

