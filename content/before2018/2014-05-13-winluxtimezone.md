---
layout: post
title: "Windows 和 Linux共存时时间不一致"
tagline: "让Windows把硬件时间当作UTC"
description: "Windows Linux 时时间不一致 相差8小时"
category: windows
tags: [windows, linux, timezone]
---


让Windows把硬件时间当作UTC，与Linux/Unix/Mac保持一致。

在 注册表项：HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation\
下中添加一项数据类型为REG_DWORD，名称为RealTimeIsUniversal，值设为1 的键值。

或者在windows下运行下列代码

	@echo off
	color 0a
	Reg add HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation /v RealTimeIsUniversal /t REG_DWORD /d 1
	echo.
	echo 已让Windows识别存贮在主板CMOS内的时间为格林威治标准时间（GMT）,即系统根据CMOS时间和设置的时区来确定当前系统的时间。
	echo.
	pause
	