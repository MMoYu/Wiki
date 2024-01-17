---
title: method-of-adb
date: 2023-11-21 14:51:14
description: "Android Debug Bridge"
sticky: 998
tags: [Android,ADB,Linux]
---

#### 常用ADB命令整理

--- Android Debug Bridge（安卓调试桥）

- 连接/重启/安装/应用管理

  ```shell
  adb devices
  
  # 多设备连接
  adb devices
  adb -s 192.168.xxx.xxx shell
  
  # adb wifi
  adb tcpip 5555 //让设备端的 adbd 重启，并在 TCP 端口 5555 处监听
  adb connect 192.168.xxx:5555 //远程连接设备，设备的 IP 地址是 192.168.xxx
  adb disconnect 192.168.xxx:5555 // 断开连接
  
  # adb重新挂载
  adb root
  adb remount
  
  # adb重启
  adb reboot				//普通重启 
  adb reboot recovery		//重启到Recovery界面 
  adb reboot fastboot		//重启到fastboot 
  adb reboot bootloader	//重启到bootloader 
  adb reboot ed 重启到紧急下载（高通only）
  
  # 安装apk
  adb install xxx.apk 路径
  adb install -r xxx.apk 路径 (强制安装)
  
  # 文件导出/上传
  adb push xxx.txt 本地路径 
  adb pull xxx.txt 本地路径
  
  # 截图导出
  adb shell screencap /xxx.jpg //截图
  adb pull xxx //导出
  
  # 录制视频
  adb shell screencord /xxx.mp4
  
  # 查看CPU架构
  adb shell getprop ro.product.cpu.abi
  
  # 屏幕常亮
  adb shell settings put system screen_off_timeout 600000
  ```

- Android-调试/Debug

  --- adb logcat 安卓系统专用指令，打印内容只与应用程序相关，即只打印用户态log信息

  ```shell
  # 常用
  adb logcat > log.txt 
  adb logcat -b(特定类型)  kernel > k.txt 
  adb logcat -b all -d(一次性输出后退出) >log.txt  //这个好用，有一次性退出 
  adb shell logcat -b all > log.txt     //kernel log
  
  # 实时查看音量级别
  logcat  | grep storeVolume
  
  # 实时输出
  logcat -b all | grep input_report_key
  
  # 清空日志信息，适用于复现前清除无用log
  logcat -c
  
  radio：查看包含无线装置/电话相关信息的缓冲区 
  events：查看已经过解释的二进制系统事件缓冲区消息
  main：查看系统日志缓冲区(默认) 
  crash：查看崩溃日志缓冲区（默认） 
  all：查看所有缓冲区 
  default：报告main、system、crash缓冲区
  ```

---

#### getprop/setprop/watchprops

--- 在<mark>Android</mark>系统中，使用getprop命令可以从系统中读取一些设备信息，属性的文件

- getprop

  ```c
  # 从系统的配置中读取信息
  
  adb shell getprop > p.txt (导出所有属性)
  
  getprop ro.build.type（看版本 Userdebug版本/UserD版本/熔断版本）
  
  getprop | grep efuse 熔断判断
  
  getprop ro.build.fingerprint（特定事件）
  
  getprop persist.vendor.framebuffer.main(看分辨率)
  ```

- setprop

  ```c
  # setprop <prop-name> <value>
  //例如，修改进程默认分配的可以使用堆内存大小：
  adb shell setprop dalvik.vm.heapgrowthlimit 128m
  ```

- 一些常用参数说明

  ```c
  dalvik.vm.heapgrowthlimit：默认给进程分配的可使用堆内存
  dalvik.vm.heapsize：设置了android:largeHeap以后可使用的内存大小
  ro.product.brand：手机品牌
  ro.product.device：设备名称
  ro.product.model：设备内部代号
  ro.product.name：设备名称
  ro.product.manufacturer：设备制造商
  ro.serialno：设备序列号
  ro.sf.lcd_density：设备屏幕密度
  ro.config.ringtone：默认来电铃声
  ro.config.notification_sound：默认通知铃声
  ro.config.alarm_alert：默认闹钟铃声
  dalvik.vm.stack-trace-file：trace文件放置目录
  ```

---

#### User版本开启ADB

- Android

  ```diff
  Z:\work\xxx\build\make\core\main.mk
  修改：ro.debuggable=1   ro.adb.secure=0
  
  diff --git a/core/main.mk b/core/main.mk
  index c5a0baeef..ab5b9e22a 100644
  --- a/core/main.mk
  +++ b/core/main.mk
  @@ -393,7 +393,7 @@ ifneq (,$(user_variant))
     ADDITIONAL_SYSTEM_PROPERTIES += security.perf_harden=1
   
     ifeq ($(user_variant),user)
  -    ADDITIONAL_SYSTEM_PROPERTIES += ro.adb.secure=1
  +    ADDITIONAL_SYSTEM_PROPERTIES += ro.adb.secure=0
     endif
   
     ifeq ($(user_variant),userdebug)
  @@ -423,7 +423,7 @@ ifeq (true,$(strip $(enable_target_debugging)))
     ADDITIONAL_SYSTEM_PROPERTIES += dalvik.vm.lockprof.threshold=500
   else # !enable_target_debugging
     # Target is less debuggable and adbd is off by default
  -  ADDITIONAL_SYSTEM_PROPERTIES += ro.debuggable=0
  +  ADDITIONAL_SYSTEM_PROPERTIES += ro.debuggable=1
   endif # !enable_target_debugging
   
   ## eng ##
  ```

  
