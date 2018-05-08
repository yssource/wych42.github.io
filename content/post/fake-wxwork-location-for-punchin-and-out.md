---
title: "无用功|企业微信自动打卡"
subtitle: "使用 Xposed 模拟位置，定时运行 adb shell 模拟动作"
slug: "fake-wxwork-location-and-auto-punch-in_and_out"
date: 2018-04-23T14:11:22+08:00
tags:
  - 无用功
  - Android
  - 微信
  - 工作
draft: false
toc: true
---

一种简单绕过定位打卡制度的方法。For Freedom!

> 做人要诚信。 -- 鲁迅.

但是，如果你碰到了弹性工作时间却要求定位打卡又经常忘的情况,可以谨慎使用下面的方法。

### 准备

- 安卓设备。（这里用的是 moto g 2nd，老设备发挥余热）
- 安装好 Xposed 框架。(这里用的是 LineageOS 13 刷 Xposed ARM)
- 安装 [模拟位置](https://www.coolapk.com/apk/com.rong.xposed.fakelocation) 模块。 安装完成后，去 Xposed 里启用模块，重启手机使之生效。
- 安装好企业微信/钉钉。
- 可选：PC 安装好 adb。

*有动手能力的读者，如果有越狱的 iOS 设备，也可以做同样的事情*

### 模拟位置设置

打开「模拟位置」，看到应用列表，点击需要被模拟位置的。

![App List](/images/fakelocation-list.png)

点击「地图」Icon 选择地点, 会自动填入 GPS 信息, 重新选择过地点之后，点击下「更新」按钮，通知对应的应用。

![Settings](/images/fakelocation-setting-app.png)

重点是，微信、企业微信这些应用可能不(只)使用设备的 GPS 获取位置，也可能通过基站、网络来判断位置，所以如果只设置上面的地点不生效，可以填一下模拟基站信息。把设备带到正确的地点，在拨号盘输入 `*#*#4636#*#*`（Android 有效），在「手机信息」里，查到 mMNC, mCID, mTAC 几项填入保存。再重启需要模拟定位的应用即可。

一个好用的 GPS、基站工具：[GPSspg](http://www.gpsspg.com/), [cellocation 坐标反查基站接口](http://www.cellocation.com/)

### 自动打卡

**Note:** 因为本人对 Android/iOS 开发知识了解甚少，这里用各式工具拼凑了一个自动化方案，如果哪位读者有内嵌在设备内的方案，很希望能传授一下思路。

Mac 上安装 [replykit](https://github.com/appetizerio/replaykit),我们需要用到其中的 appetizer 工具。

Andoird 设备连接上 Mac，打开 usb 调试（或者网络调试），用 `adb devices` 确定已经连接好的设备名，这里用网络连接，名字为 `172.19.xx.xx:5555`.

设备解锁亮屏的情况下，在终端里运行 `path/to/appetizer trace record --device 172.19.xx.xx:5555 mytrace.trace`，在设备上按照正常的打卡逻辑操作。完成录制后，在终端里用 `exit` 退出录制。

用 `path/to/appetizer trace replay mytrace.trace 172.19.xx.xx:5555` 回放。

两个建议：

- 企业微信放在首屏固定位置。
- 录制结束时，杀掉应用进程，回到首屏，方便下次回放可以保持开头一致。

#### 定时打卡

如果有 Mac（或者其他可以跑定时任务、有 adb 的设备）闲着，可以配置一个 cron job，定时用 adb 解锁 Andoird 设备，执行回放动作。


### 闲话

- 好的工程（团队）文化和组织结构（沟通模式），一定是以实现组织的目标为导向的。
- 规章制度应该为上面的目标服务。背离上面目标的行为应该果断放弃。


