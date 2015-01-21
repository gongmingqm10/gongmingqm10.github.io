---
layout: post
title: "nodejs learnyounode"
date: 2014-10-30 14:44:23 +0800
comments: true
categories: nodejs 
published: false
description: nodejs学习教程 nodejs learnyounode

---

Here are the following steps.


1. console.log("HELLO WORLD")
2. 

```
var sum = 0;
for (var index = 2; index < process.argv.length; index ++) {
	sum += Number(process.argv[index]);
}
console.log(sum);

```

3. Calculate the file line

```
var fs = require('fs');
var str = fs.readFileSync(process.argv[2]).toString();
console.log(str.split('\n').length - 1);

```