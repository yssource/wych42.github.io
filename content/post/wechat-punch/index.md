+++
title = "企业微信远程打卡"
author = ["chi"]
date = 2018-04-23T13:41:00+08:00
lastmod = 2018-10-25T15:07:44+08:00
tags = ["wechat", "work"]
categories = ["无用功"]
draft = false
toc = false
+++

一种简单绕过定位打卡的方法。

<!--more-->

如果你碰到了弹性工作时间却要求定位打卡又经常忘的情况,可以谨慎使用下面的方法。


## 准备 {#准备}

-   安卓设备。（这里用的是 moto g 2nd，老设备发挥余热）
-   安装好 Xposed 框架。(这里用的是 LineageOS 13 刷 Xposed ARM)
-   安装 [模拟位置](https://www.coolapk.com/apk/com.rong.xposed.fakelocation) 模块。 安装完成后，去 Xposed 里启用模块，重启手机使之生效。
-   安装好企业微信/钉钉。
-   可选：PC 安装好 adb。

**有动手能力的读者，如果有越狱的 iOS 设备，也可以做同样的事情**


## 模拟位置设置 {#模拟位置设置}

打开「模拟位置」，看到应用列表，点击需要被模拟位置的。

{{< figure src="/wechat-punch/fakelocation-list.png" >}}

点击「地图」Icon 选择地点, 会自动填入 GPS 信息, 重新选择过地点之后，点击下「更新」按钮，通知对应的应用。

{{< figure src="/wechat-punch/fakelocation-setting-app.png" >}}

重点是，微信、企业微信这些应用可能不(只)使用设备的 GPS 获取位置，也可能通过基站、网络来判断位置，所以如果只设置上面的地点不生效，可以填一下模拟基站信息。把设备带到正确的地点，在拨号盘输入 \`\*#\*#4636#\*#\*\`（Android 有效），在「手机信息」里，查到 mMNC, mCID, mTAC 几项填入保存。再重启需要模拟定位的应用即可。

好用的 GPS、基站工具：[GPSspg](http://www.gpsspg.com/), [cellocation 坐标反查基站接口](http://www.cellocation.com/)
