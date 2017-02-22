---
layout: post
title: "如何上传 AAR 到 Jcenter"
date: 2017-02-17 16:38:01 +0800
description: Android, aar, 上传JCenter
comments: true
categories: Android

---

自我进入到代码这个行业开始，就有前辈早早告诉我要积极活跃在开源社区，于是认识了 github, 然后也见识到了开源社区的魅力之处。看到大神们都会积极贡献自己的开源仓库，自己也被这种开源精神感动到了。过程中我也尝试写过一些小框架，但是发布到 jcenter 上，所以知名度大大降低。

相比于几年前，现在直接拷贝源代码到自己工程中的做法越来越少了。得益于 Android 使用 Gradle 管理依赖，更多的人选择将开源仓库发布到 jcenter 中，然后被更多的人使用。今天我们将尝试从零开始，一步步上传自己的应用到 Jcenter，然后优雅的通过 gradle `compile 'x.y.z:a:0.0.1'` 引用。

<!-- more -->

## 理解 Android Gradle 构建

以 BottomTabAndroid 工程为例，在一个项目中通常至少有三个 gradle 文件，分别为：

* BottomTabAndroid/settings.gradle
* BottomTabAndroid/build.gradle
* BottomTabAndroid/app/build.gradle

分析各 gradle 文件的具体内容，我们可以发现位于最外层的 settings.gradle 定义项目中的各个不同 module，这时候指定的 module 才能被我们的工程识别。识别之后，如果 A module 需要引用 B module，那么在 A 的 build.gradle 文件中，则需要 `compile project(':B')`。

{% img /images/android_gradle.png Gradle 示例工程 %}

在这个工程中，存在 bottomtab 和 app 两个 module, 而 bottomtab 被 app 所引用。

settings.gradle 内容如下:

```
include ':app', ':bottomtab'
```

build.gradle 内容如下：

```
buildscript {
    repositories {
    	 // 指定构建脚本的下载库
        jcenter()
    }
    dependencies {
    	 // 指定引入何种构建脚本，比如注释中引入了上传到 jecneter 和 maven 构建的脚本
        classpath 'com.android.tools.build:gradle:2.2.3'

//        classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.7.3'
//        classpath 'com.github.dcendents:android-maven-gradle-plugin:1.5'
    }
}

allprojects {
    repositories {
    	 // 指定从何处下载子项目的库依赖
        jcenter()
//        maven {
//            url 'https://dl.bintray.com/gongmingqm10/maven/'
//        }
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}

```
build.gradle 文件中主要定义了 buildscript和 allprojects.repositories 两部分内容。确定了当前项目中使用的构建脚本库，以及项目中依赖库的下载仓库。

app/build.gradle 文件则定义了当前 module 的配置信息，比如 Android 工程中需要对 API，签名，项目依赖等信息进行配置。本文暂且略过其详细介绍。


认识了 Gradle 的基本配置，我们再看看如下这句熟悉的 Gradle 语言：

```
compile 'net.gongmingqm10.uikit:bottomtab:0.2.0'
```

所有的仓库都是按照 `Group_ID:ARTIFACT_ID:VERSION` 定义，所以对于 bottomtab 这个库而言， groupID = net.gongmingqm10.uikit， artifactId = bottomtab，version = 0.2.0。这种方式的好处是，我不需要在项目中导入众多的子 module，只需要通过这一句 DSL (领域特定语言) 引入仓库，并且能够随时更改版本。

下面我们一步步构建出这个 `net.gongmingqm10.uikit.bottomtab` 依赖库，上传到 jcenter，以在项目中直接使用。

## 发布开源库

本阶段我们将从零开始一步步构建并发布自己的第一款开源库。我们假设你已经在使用 Android Studio 开发应用，并且对 Gradle 有一定的了解。Android 仓库通常有两种格式，aar 和 jar，其中 aar 格式中包含 Android 自身相关的资源文件。而 jar 则主要包含 .class 类文件。考虑到我们希望做出一款 Bottom Tab 底部栏的库，因此我们最后使用的是 aar 格式。

### 1. 配置 bintray

首先注册 [Bintray OpenSource](https://bintray.com/signup/oss)，如果之前有账号请跳过此步骤。通过 OpenSource 途径注册的 bintray 属于公开免费版，这正和我的期望相符。注意不要直接在首页注册，那样默认注册的是收费版本。

访问个人主页，点击 "Add New Repository" 跳转到创建 repository 页面，这里我们需要创建一个 name = maven，type = Maven 的 Repository。其中的 license 以及 description 按照个人偏好自行填写。

{% img /images/bintray_1.png 创建 maven 仓库%} 

点击 `Create` 之后，我们可以顺利在首页看到 repository 中多了一个 maven 的仓库。

{% img /images/bintray_2.png 首页 maven 仓库%}

点击刚创建的 maven 仓库，如果是第一次创建，应该看不到任何的 package。我们需要 `Add New Package`。

{% img /images/bintray_3.png maven 仓库%}

创建 package 时，我们填写 packageName = ‘bottom-tab’，这个属性比较重要，在后续步骤的 gradle 中需要用到。注意到需要填写 versionControl，我提前在 github 上创建了一个 repository bottomtab，然后把URL填入即可。

创建完成之后，bintray 的配置基本告一段落。但是正式开始之前我们还需要拿到 API_KEY，要不然所有人都可以更新你的库，岂不是一件很恐怖的事情。

总结这一步，我们所需要完成的事情：

* 注册bintray，获得 bintray 用户名和 API KEY;
* 在 GITHUB 新建 bottom-tab 仓库；
* 在 bintray 新建 maven/bottom-tab 仓库；

### 2. 配置 GRADLE 工程

为了发布一款 “接地气” 的 bottom-tab 库，我们先构建一个 BottomTabAndroid 应用，然后添加 bottom-tab 子模块，然后将 bottom-tab 子模块打包并发布到 jcenter 上，而 app 模块则是一个 demo。展示 bottom-tab 的实际效果。

具体代码请参考：[https://github.com/gongmingqm10/BottomTabAndroid](https://github.com/gongmingqm10/BottomTabAndroid)。项目截图如下：

{% img /images/android_gradle.png BottomTabAndroid %}

开发过程中，我先让 app 直接引用子模块 bottom-tab，功能开发完成之后，开始准备最重要的打包上传。

**Step 1: 配置 local.properties**

我们需要将 bintray 账户相关的 username 和 apikey 存放在此文件中。在提交代码时，此文件被自动忽略，从而保证了其安全性。默认的 local.properties 文件中保存了 sdk 路径。我们在其后添加上 uername 和 apikey:

```
sdk.dir=/Users/mingong/Project/sdk

bintray.user=gongmingqm10
bintray.apikey=thisissomesecretkeyanddonotcopyme

```

请将你自己的 username 和 apikey 复制替换到此文件。

**Step 2: 配置 $projectDir/build.gradle**

为了上传 bintray，我们需要使用 [gradle-bintray-plugin](https://github.com/bintray/gradle-bintray-plugin) 和 [android-maven-gradle-plugin](https://github.com/dcendents/android-maven-gradle-plugin) 两款构建插件。

```
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.2.3'

		 // 添加如下插件以上传 AAR 至 bintray
        classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.7.3'
        classpath 'com.github.dcendents:android-maven-gradle-plugin:1.5'
    }
}
...
```
请关注 buildscript{dependecies{}} 中配置，其余的配置项已省略。

**Step 3: 配置 $projectDir/bottomtab/build.gradle**

```
apply plugin: 'com.android.library'
apply plugin: 'com.github.dcendents.android-maven'
apply plugin: 'com.jfrog.bintray'

android {
    compileSdkVersion 23
    buildToolsVersion "23.0.3"

    defaultConfig {
        minSdkVersion 17
        targetSdkVersion 23
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"

    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:23.3.0'
    testCompile 'junit:junit:4.12'
}

if (project.hasProperty("android")) { // Android libraries
    task sourcesJar(type: Jar) {
        classifier = 'sources'
        from android.sourceSets.main.java.srcDirs
    }

    task javadoc(type: Javadoc) {
        source = android.sourceSets.main.java.srcDirs
        classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
    }
} else { // Java libraries
    task sourcesJar(type: Jar, dependsOn: classes) {
        classifier = 'sources'
        from sourceSets.main.allSource
    }
}

task javadocJar(type: Jar, dependsOn: javadoc) {
    classifier = 'javadoc'
    from javadoc.destinationDir
}

artifacts {
    archives javadocJar
    archives sourcesJar
}

// 你只需要把这里的参数修改为自己的库名称即可
ext {
    siteUrl = 'https://github.com/gongmingqm10/BottomTabAndroid'
    gitUrl = 'https://github.com/gongmingqm10/BottomTabAndroid.git'
    currentVersion = '0.2.0'
    groupId = 'net.gongmingqm10.uikit'
    artifactId = 'bottom-tab'
    desc = 'Android bottom tab'
}


group = groupId
version = currentVersion

install {
    repositories.mavenInstaller {
        pom {
            project {
                name artifactId
                description desc
                url siteUrl

                packaging 'aar'
                groupId groupId
                artifactId artifactId
                version currentVersion

                licenses {
                    license {
                        name 'The Apache Software License, Version 2.0'
                        url 'http://www.apache.org/licenses/LICENSE-2.0.txt'
                    }
                }
                developers {
                    developer {
                        id 'gongmingqm10'
                        name 'Ming Gong'
                        email 'gongmingqm10@foxmail.com'
                    }
                }
                scm {
                    connection gitUrl
                    developerConnection gitUrl
                    url siteUrl
                }
            }
        }
    }
}

Properties properties = new Properties()
properties.load(project.rootProject.file('local.properties').newDataInputStream())
bintray {
    user = properties.getProperty("bintray.user")
    key = properties.getProperty("bintray.apikey")
    configurations = ['archives']
    pkg {
        repo = "maven"
        name = artifactId
        websiteUrl = siteUrl
        vcsUrl = gitUrl
        licenses = ["Apache-2.0"]
        publish = true
        publicDownloadNumbers = true
    }
}

```

注意，其中大部分配置都是很通用的，对于不同的 library，我们只需要修改如下不同的配置即可：

```
// 你只需要把这里的参数修改为自己的库名称即可
ext {
    siteUrl = 'https://github.com/gongmingqm10/BottomTabAndroid'
    gitUrl = 'https://github.com/gongmingqm10/BottomTabAndroid.git'
    currentVersion = '0.2.0'
    groupId = 'net.gongmingqm10.uikit'
    artifactId = 'bottom-tab'
    desc = 'Android bottom tab'
}
```

### 3. 正式发布 jcenter

如上配置完成后，我们开始正式发布 AAR 到 jcenter。在项目根目录下，运行 gradle 命令：

```
./gradlew clean install bintrayUpload

```

如果一切顺利的话，这一步将直接成功。如果出错，请根据错误信息重新诊断并修复。再次访问 bintray/gongmingqm10/maven/bottom-tab 项目首页，你可以看到刚上传的新版本。

{% img /images/bintray_6.png 上传新版本成功 %}

上传到个人仓库中，可以看到右上方有一个地址 `https://dl.bintray.com/gongmingqm10/maven`。这是个人的 maven 仓库地址，如果我们在在 $projectDir/build.gradle 添加此地址：

```
...

allprojects {
    repositories {
        jcenter()
        maven {
            url 'https://dl.bintray.com/gongmingqm10/maven/'
        }
    }
}

...
```
然后直接在 app/build.gradle 中，添加如下语句，则可以直接引用刚才上传的 bottom-tab。如果这种引用方式成功，说明我们的 `./gradlew install bintrayUpload` 这一步没有问题。

```
dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:23.3.0'
    testCompile 'junit:junit:4.12'
 
    compile 'net.gongmingqm10.uikit:bottomtab:0.2.0'
//    compile project(':bottomtab')
}
```

然而，事情到这里并没有完。因为我们使用的大部分的库是不需要在 $projectDir/build.gradle 中做配置的。重新观察 bottom-tab 这个主页面，右下角 `linked to` 可以点击链接 `Add to JCenter`。这一步才是真正的发布 library 到 JCenter 中。然后我们需要做的就是等待 JCenter 官方团队的审核。一般两三个工作日就可以审核通过。然后下一次引用，你就不需要添加私有的 maven 地址了。当然如果你对这个库有更新，也需要重新点击 Add to JCenter。

所以最后的效果是  `compile 'net.gongmingqm10.uikit:bottomtab:0.2.0'` 可以一键引用我们新建的 Android 底部栏组件，试用效果如下：

{% img /images/bintray_demo.png 底部栏效果 %}

具体使用方法，请参考 [BottomTab README](https://github.com/gongmingqm10/BottomTabAndroid/blob/master/README.md)。
## 参考

* [How to distribute your own Android library through jCenter and Maven Central from Android Studio](https://inthecheesefactory.com/blog/how-to-upload-library-to-jcenter-maven-central-as-dependency/en)




