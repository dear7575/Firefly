---
title: ✨Lebara UK Wi-Fi Calling 排查总结
published: 2026-08-08
pinned: false
description: "记录小米 15 Pro 上 Lebara UK Wi-Fi Calling 无法稳定拉起的原因、状态判断、恢复方式和切卡操作规范。"
tags: [Lebara, Wi-Fi Calling, eSIM, VoWiFi, 小米]
category: 技术分享
draft: false
image: ../uploads/20260110/mt.png
---

# Lebara UK Wi-Fi Calling 排查总结

这次排查的核心结论是：Lebara UK 在小米 15 Pro 上无法稳定拉起 Wi-Fi Calling，并不是 APN 配置问题，也不是单纯的代理出口问题，而是 eSIM / SIM profile 被系统识别成了不同的运营商身份。

成功时应进入英国 Lebara 路径：

```text
23487 / gb / Lebara_UK_Commercial
```

失败时会掉到荷兰 Vodafone 路径：

```text
20404 / nl / Netherlands-VoLTE-Vodafone
```

一旦变成 `20404/nl`，手机会走荷兰 ePDG 和荷兰 IMS 配置，Wi-Fi Calling 通常无法注册成功。

## 已观察到的现象

GG 卡切换后可以很快拉起 Wi-Fi Calling，因为它稳定识别为英国 O2 / giffgaff 配置：

```text
23410 / gb / Telefonica_UK_Commercial
O2 WiFiCall
```

Lebara 卡失败时，小米读到的是：

```text
gsm.sim.operator.numeric=20404
gsm.sim.operator.iso-country=nl
ril.mcc.mnc1=20404
persist.radio.mbn_sw_sub1=Netherlands-VoLTE-Vodafone
carrierId=-1
```

此时日志里会出现：

```text
isImsRegistered=false
isVowifiEnabled=false
WiFi_Error08-Unable to connect
```

## 为什么关闭飞行模式后会变成荷兰卡

关闭飞行模式会重新启动蜂窝基带。基带会重新读取 eSIM、重新选择 PLMN / IMSI、重新匹配运营商配置。

Lebara 这类卡存在多 IMSI / 多运营商承载特征。在中国环境下，一旦蜂窝基带开始完整找网或驻网，设备很容易选择到 `20404` 这条国际/荷兰 Vodafone 承载身份。Android 随后按 `20404/nl` 加载荷兰 Vodafone 的 MBN 和 IMS 配置。

结果就是：

```text
20404/nl -> Netherlands-VoLTE-Vodafone -> 荷兰 ePDG -> IMS 注册失败
```

而正确路径应该是：

```text
23487/gb -> Lebara_UK_Commercial -> 英国 ePDG -> IMS 注册成功
```

## 正确成功状态

Lebara Wi-Fi Calling 成功时，目标状态应接近：

```text
gsm.sim.operator.numeric=23487
gsm.sim.operator.iso-country=gb
persist.radio.mbn_sw_sub1=Lebara_UK_Commercial
carrierId=2564
isImsRegistered=true
getImsRegistrationTechnology=1
isVowifiEnabled=true
```

其中 `getImsRegistrationTechnology=1` 表示 IMS 注册在 WLAN / IWLAN 上。

## 拉起 Wi-Fi Calling 的前置条件

1. 飞行模式开启。
2. 只打开 Wi-Fi。
3. 路由器级或热点级英国出口。
4. 使用支持 UDP / XUDP 的英国节点。
5. DNS 不能污染 `*.3gppnetwork.org`，不要解析到 fake-ip。
6. Lebara 必须被识别为 `23487/gb/Lebara_UK_Commercial`。
7. 再打开 WLAN 通话并等待 IMS 注册。

UDP / XUDP 的重点是确保 Wi-Fi Calling 所需的 IKE / IPsec 流量可以通过，尤其是 UDP 500 和 UDP 4500。节点如果不支持 UDP，或者 UDP 被中转/丢包，Wi-Fi Calling 可能无法建立。

## 切卡操作规范

### 从 Lebara 切到 GG 或其他卡

1. 当前如果 Lebara Wi-Fi Calling 已经拉起，保持飞行模式开启。
2. 在 EasyEUICC 中先禁用 Lebara 卡。
3. 确认 Lebara 已禁用后，再切换到 GG 或其他卡。
4. 如确实需要关闭飞行模式，必须在 Lebara 已禁用后再关闭。

### 从 GG 或其他卡切回 Lebara

1. 先切到英国 UDP / XUDP 节点。
2. 开启飞行模式。
3. 只打开 Wi-Fi。
4. 在 EasyEUICC 中启用或重新下载 Lebara。
5. 读取状态，确认是否进入：

```text
23487 / gb / Lebara_UK_Commercial
```

6. 确认进入 UK 配置后，再打开 WLAN 通话。
7. 等待 Wi-Fi Calling / IMS 注册成功。
8. 成功后不要关闭飞行模式。

## 禁止操作

不要在 Lebara 启用状态下关闭飞行模式。

原因是关闭飞行模式会触发基带重新读卡，Lebara 很容易从英国配置变成荷兰配置：

```text
23487/gb -> 20404/nl
```

一旦变成 `20404/nl`，通常无法靠开关 WLAN 通话、开关 IMS、改 APN 恢复。需要在 EasyEUICC 中删除 Lebara 卡，再重新下载。

## 失败后的恢复方式

如果已经变成以下状态：

```text
20404 / nl / Netherlands-VoLTE-Vodafone
```

不要继续反复开关 WLAN 通话，也不要优先改 APN。建议直接：

1. 保持英国出口。
2. 开启飞行模式。
3. 只打开 Wi-Fi。
4. 使用 EasyEUICC 删除 Lebara profile。
5. 重新下载 Lebara eSIM。
6. 立刻检查是否进入 `23487/gb/Lebara_UK_Commercial`。
7. 如果状态正确，再打开 WLAN 通话。

## 判断是否正常的 ADB 检查项

```powershell
$adb = "C:/Users/dear7575/Documents/Codex/2026-08-07/ni-z/work/android-platform-tools/platform-tools/adb.exe"

& "$adb" shell getprop | Select-String -Pattern "gsm.sim.operator|gsm.operator|ril.mcc.mnc|persist.radio.mbn|ro.miui.region"
& "$adb" shell dumpsys isub
& "$adb" logcat -b radio -d -v time -t 1500 | Select-String -Pattern "Phone-1 .*isImsRegistered|Phone-1 .*getImsRegistrationTechnology|handleFeatureCapabilityChanged|isVowifiEnabled|WiFi_Error|CODE_REGISTRATION_ERROR|IWLAN|epdg|3gppnetwork"
```

## 关键记忆点

GG 卡能秒起，说明 Wi-Fi、英国出口、UDP 基本没有问题。

Lebara 的问题是 eSIM profile / carrier config 匹配不稳定。只要它掉到 `20404/nl`，Wi-Fi Calling 就会失败。恢复的关键不是改 APN，而是让卡重新进入 `23487/gb/Lebara_UK_Commercial`。
