---
layout: post
title: "Android 应用的持续交付"
date: 2015-06-29 11:40:16 +0800
description:
comments: true
categories: Android

---

当我们谈软件质量时，我们一般会谈到测试。测试作为保障软件质量的重要手段正在被开发者逐渐认知。谈到测试时，大部分人都知道Web测试，对于前端JS或者后台，大部分“靠谱”的创业公司也都会用测试来保证软件质量。可是对于起步相对较晚的移动端测试，用的人并不多。

在测试方面，Android早期即存在UIAtomator和Monkey之类的测试，但是用起来实在不方便。2013年，Google开源了针对An ndroid平台的移动测试框架 - Espresso。Espresso可以针对每个页面(Activity)进行测试。开发者可以根据ID获取到页面元素，然后进行点击、长按等操作。结合Junit和Mock等工具，使得移动端测试成为可能。移动端拥有了测试框架的辅助，借助CI平台，持续交付也成为可能。

<!-- more -->

##Android测试的类别

从开发者角度来看，通常的Web平台测试可以分为单元测试、集成测试以及功能测试。移动应用主要用来显示数据，显示框输入数据，相应用户的点击、滑动等操作。所以Android应用的开发工作大部分集中在UI上。因此，App测试大致可分为UI单元测试和功能测试。

UI单元测试覆盖界面显示以及用户与界面的交互。功能测试则是确保功能的正确性。

为了更好的说明App中UI测试和功能测试如何进行的，以一个Story举例如下：

```
Story-101
Summary: 登录模块，用户能够使用用户名和密码登录App。
Acceptance criterions:

Given 作为用户，我打开App进入首页；
When 我点击“登录”按钮；
Then 我跳转到登录页面；

Given 作为用户，我在登录页面
When 我只输入用户名
Then 登录按钮处于Disable状态
When 我输入密码
Then 登录按钮可以点击，点击之后，调用API

Given 作为用户，我在登录页面
When 我输入正确的用户名和密码，并登录
Then 页面跳转到首页

Given 作为用户，我在登录页面
When 我输入错误的用户名和密码，并登录
Then 页面上方显示“用户名或密码错误”提示
And 密码被清空，登录按钮重新Disable

```

###UI单元测试

UI(User interaction)测试又指界面测试。界面是软件与用户交互的最直接的部分，通过UI测试核实用户和软件拥有正常的交互。

UI测试的功能点比较简单：比如点击按钮的行为，按钮是否应该被禁用，页面文字、颜色等信息，是否弹框，点击弹框按钮应该跳转到什么页面等。

在登录页面中，假设登录页面的名字为LoginActivity，这是需要写的UI测试如下：

1.Test login button is not available until username and password are filled   

```
usernameEditText.setText("gongmingqm10")
assertThat(loginButton.isDisabled()).isTrue()

passwordEditText.setText("password")
assertThat(loginButton.isDisabled()).isFalse()
```

2.Test should invoke Api when login button is clicked    

```
usernameEditText.setText("gongmingqm10")
passwordEditText.setText("password")

loginButton.performClick()

verify(loginApi).login(eq("gongmingqm10"), eq("encryptedpassword")

```

3.Test should show error dialog when login failed  

```
usernameEditText.setText("gongmingqm10")
passwordEditText.setText("wrongpassword")
activity.loginFailed()
assertThat(passwordEditText.getText().toString()).isEqualTo("")
assertThat(errorText).isVisibile()
assertThat(errorText.getText().toString()).isEuqalTo("用户名或密码错误")

```

4.Test should navigate to home page when login succeed

```
usernameEditText.setText("gongmingqm10")
passwordEditText.setText("correctpassword")
activity.loginSuccess()
String actualPageName = getNextStartedActivity().getComponentName()
assertThat(actualPageName).isEqualTo(HomeActivity.class.getName())
```

观察上述几个测试，我们可以看到对UI的测试粒度相对较小。主要测试用户输入，按钮点击，服务器返回结果之后，界面应该如何反应。UI测试通常也包括简单的单元测试，Android中需要进行单元测试的内容较少，所以对于一些工具类的单元测试通常和UI测试放在同一个模块中。(PS: 上述测试代码的写法和你选取的测试框架有关)

###功能测试

功能测试主要把测试对象看作一个黑盒子，测试者并不关注具体的功能实现逻辑，只需要关注输入产生期望的输出即可。Android的功能测试和Web的功能测试类似。在真机或模拟器中运行App，模拟输入、点击等操作，识别界面上是否存在某些期望的文字或行为。

功能测试不需要关注细节，只需要关注功能，对登录Story的功能测试如下：

1.With valid username and password, user can successfully login and navigate to home screen

```
tap_when_element_exists "android.widget.EditText hint:'Username'"
keyboard_enter_text 'gongmingqm10'
tap_when_element_exists "android.widget.EditText hint:'Password'"
keyboard_enter_text 'some_correct_password'
@home_page.wait_for_page_to_load
welcome_message = @home_page.get_welcome_message
raise 'Navigate to home page failed' if welcome_message.empty?
```

2.With wrong username and password, user will see an error text

```
tap_when_element_exists "android.widget.EditText hint:'Username'"
keyboard_enter_text 'gongmingqm10'
tap_when_element_exists "android.widget.EditText hint:'Password'"
keyboard_enter_text 'wrong_password'
query "android.widget.TextView id: 'error_box' text:'用户名或者密码错误'"
```

##Android常用测试框架

###针对UI测试
一般项目上都会采用[Robolectric](http://robolectric.org/)框架，Robolectric运行过程中不需要启动模拟器，因此执行过程很快。

除了Robolectric，Google官方推荐使用Espresso来做UI测试。Espresso既可以作UI测试，也可以做简单的功能测试。

###针对功能测试

可以做功能测试的框架也很多，包括Robotium, uiautomator, Espresso, Appium 和 Calabash。对比其主要区别如下：

真实项目过程中功能测试框架使用较多的是Appium和Calabash。这两个框架的优点是能够同时支持Android和iOS，并且不需要注入到工程中(即只需要编译后生成的安装包)，配合响应的测试脚本即可完成测试。

##Android持续交付

CD(Continuous Dilivery)强调能够随时给客户交付有价值的产品。在Web开发上，我们通常运用CI平台持续部署网站到各个环境中。从而实现整个产品的持续交付。在Android平台上也是类似，唯一不同点是Android每发布一个版本需要用户手动更新手机上的App。

**1. 集成你的测试框架**

假定我们已经选定其他的技术栈，这时只需要集成UI测试和功能测试即可。UI单元测试框架我们选择Robolectric，而功能测试框架选择Calabash。参考[Robolectric](http://robolectric.org/)和[Calabash for Android](https://github.com/calabash/calabash-android)官方文档，集成测试框架并编写第一个Dummy Test。

**2. 通过你的第一个Dummy Test**

CI(Continuous Integration)平台，选择Jenkins。因为Jenkins有各种丰富的插件支持。选择Jenkins平台之后，最后能够有一台带界面的Mac Mini(虽然Android可以配置无界面运行，但是配置相对麻烦并且运行太慢)。在Mac Mini上搭建CI环境，配置Android运行环境。此步的目的是你的Dummy Test能够在Jenkins上运行

**3. 自动上传你的应用**

我们希望每次Push之后生成的包能够被QA测试，Showcase时候能够被客户测试。所以我们需要持续的发布新生成的包。我接触到的通常有两种做法：

* 使用Dropbox自动同步上传：这种方法是最省成本的。每次打包新版本的时候只需要给文件带上不同的版本号，方便辨识。包生成之后，只需要存放到本地的Dropbox共享文件夹中。Dropbox会为我们自动上传。手机端则可以直接访问Dropbox下载，安装。
* 使用[HockeyApp](http://hockeyapp.net/features/)管理版本：HockeyApp需要收费，但是可以很方便的通过部署脚本自动上传新生成的包。手机端只需要安装HockeyApp客户端，即可选择更新。

类似的工具还有很多，选择适合自己的才是最好的。

##结语

移动端测试虽然发展时间不长，但是却不断在完善中。我们以前习惯用邮件或U盘发布新版供他人测试。我们以前习惯开发App时不写测试。那些时代应该过去了。运用Robolectric，我们可以很轻松的测试驱动开发Android App；运用Calabash或Appium，我们可以更好的控制App质量，保证App功能；运用Jenkins等CI平台，我们的App可以持续交付啦!
