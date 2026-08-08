---
title: ✨Java 获取以太网IP
published: 2024-04-21
pinned: false
description: "记录 Java 遍历网卡并获取以太网 IP 的实现方式，适用于多网卡环境下筛选指定网络地址。"
tags: [Java, IP]
category: 技术分享
draft: false
image: ../uploads/20260110/yy.png
---

# Java 获取以太网IP

> 遍历获取以eth开头的网卡

```java
package com.lds.utils;

import cn.hutool.log.StaticLog;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import javax.servlet.http.HttpServletRequest;
import java.net.InetAddress;
import java.net.NetworkInterface;
import java.net.SocketException;
import java.util.Collections;
import java.util.Enumeration;

/**
 * <strong style='color:purple;'>Created with IntelliJ IDEA.<hr>
 * <strong style='color:orange;'>Author: lds<hr>
 * <strong style='color:yellow;'>Date: 2021/9/1 10:01<hr>
 * <strong style='color:blue;'>Class: com.lds.util<hr>
 * <strong style='color:indigo;'>Project: demo<hr>
 * <strong style='color:red;'>Description: 获取当前IP及端口<hr>
 */
@Slf4j
@Component
public class IPUtils {

    private static String serverPort;
    @Value("${server.port}")
    private String port;

    @PostConstruct
    public void setServerPort() {
        serverPort = port;
        StaticLog.info("port:" + serverPort);
    }

    /**
     * 获取当前服务的ip和端口
     *
     * @param
     * @author: lds
     * @date: 2021/9/1 10:03
     * @return: {@link String}
     */
    public static String getServerIpAndPort() {

        InetAddress address = null;
        try {
            Enumeration<NetworkInterface> interfaces = NetworkInterface.getNetworkInterfaces();
            for (NetworkInterface ni : Collections.list(interfaces)) {
                if (ni.isUp() && !ni.isLoopback() && ni.getName().startsWith("eth")) {
                    Enumeration<InetAddress> addresses = ni.getInetAddresses();
                    for (InetAddress inetAddress : Collections.list(addresses)) {
                        if (inetAddress.isSiteLocalAddress()) {
                            log.info("Ethernet IP Address: {}", inetAddress.getHostAddress());
                            address = inetAddress;
                        }
                    }
                }
            }
        } catch (SocketException e) {
            log.error("", e);
        }
        return address.getHostAddress() + ":" + IPUtils.serverPort;
    }

    /**
     * 获取当前服务的ip和端口
     *
     * @param
     * @author: lds
     * @date: 2021/9/1 10:03
     * @return: {@link String}
     */
    public static String getServerIpAndPort(InetAddress address) {
        return address.getHostAddress() + ":" + IPUtils.serverPort;
    }

    public static String getRealIp(HttpServletRequest request) {
        String ipAddress = request.getHeader("X-Real-IP");
        if (ipAddress == null || ipAddress.isEmpty() || "unknown".equalsIgnoreCase(ipAddress)) {
            ipAddress = request.getHeader("X-Forwarded-For");
        }
        if (ipAddress == null || ipAddress.isEmpty() || "unknown".equalsIgnoreCase(ipAddress)) {
            ipAddress = request.getHeader("Proxy-Client-IP");
        }
        if (ipAddress == null || ipAddress.isEmpty() || "unknown".equalsIgnoreCase(ipAddress)) {
            ipAddress = request.getHeader("WL-Proxy-Client-IP");
        }
        if (ipAddress == null || ipAddress.isEmpty() || "unknown".equalsIgnoreCase(ipAddress)) {
            ipAddress = request.getHeader("HTTP_CLIENT_IP");
        }
        if (ipAddress == null || ipAddress.isEmpty() || "unknown".equalsIgnoreCase(ipAddress)) {
            ipAddress = request.getHeader("HTTP_X_FORWARDED_FOR");
        }
        if (ipAddress == null || ipAddress.isEmpty() || "unknown".equalsIgnoreCase(ipAddress)) {
            ipAddress = request.getRemoteAddr();
        }
        return ipAddress;
    }

}

```
