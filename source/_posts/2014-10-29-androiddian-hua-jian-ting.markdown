---
layout: post
title: "Android电话监听"
date: 2014-10-29 11:11:48 +0800
comments: true
categories: android
description: Android电话监听

---
看题名，本篇博文似乎是关于怎么监听别人的电话。其实不然，这里我主要做的是监听自己的电话。监听自己电话的场景来源如下：  
我最近在换手机，当然也在换号。于是面对两个号码两部手机，特别是一些捆绑了银行卡活着以前的快递等情况下，我必须得带两部手机出门，防止旧手机上重要的电话漏接。最开始是想直接进行通过转接，但是面对电信和移动间的通话转移，我放弃了这个想法。于是开始想着用软件提醒。  
在老手机上安装APP，这个APP的作用是监听来电，来电挂断之后会自动给你设定的新手机上发送短信，告诉你谁给你打来了一个未接电话，这样你在新手机上看到短信来电提醒，于是可以直接在新手机上进行回复了。虽然不能直接解决通话的问题，但是确解决了我的问题。考虑到老手机上每个月的短信套餐，发现其实成本也几乎为零。  
  
实现部分主要注册一个CallReceiver，并给我们的Receiver监听电话，发送短信，开机重启等权限。编译通过后便可以安装在手机上使用了。贴出主要代码：  

```
package net.gongmingqm10.callwatcher;

import android.app.Service;
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.telephony.SmsManager;
import android.telephony.TelephonyManager;
import android.util.Log;

public class CallReceiver extends BroadcastReceiver {

    private final static String NUMBER = "Your Phone number"; //我的新手机号
    @Override
    public void onReceive(Context context, Intent intent) {
        if (intent.getAction().equals(Intent.ACTION_NEW_OUTGOING_CALL)) return;
        TelephonyManager tm = (TelephonyManager) context.getSystemService(Service.TELEPHONY_SERVICE);
        SmsManager sms = SmsManager.getDefault();
        Log.i("state", "state = " + tm.getCallState());
        if (tm.getCallState() == TelephonyManager.CALL_STATE_IDLE) {
            String incomingNumber = intent.getStringExtra("incoming_number");
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            sms.sendTextMessage(NUMBER, "", "主人，有人给你附属机打电话了：" + incomingNumber, null, null);
        }
    }
}

```

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="net.gongmingqm10.callwatcher" >

    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
    <uses-permission android:name="android.permission.READ_PHONE_STATE"/>
    <uses-permission android:name="android.permission.PROCESS_OUTGOING_CALLS"/>
    <uses-permission android:name="android.permission.SEND_SMS"/>

    <application
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" >
        <activity
            android:name=".MainActivity"
            android:label="@string/app_name" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <receiver android:name=".CallReceiver">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED"/>
                <action android:name="android.intent.action.PHONE_STATE"/>
                <action android:name="android.intent.action.NEW_OUTGOING_CALL"/>
            </intent-filter>
        </receiver>
    </application>

</manifest>

```

主界面几乎没任何交互。主要实现功能，想更多交互的话可以继续～