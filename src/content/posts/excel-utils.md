---
title: ✨Excel 导出工具类
published: 2024-06-24
pinned: false
description: ""
tags: [Excel, 工具类]
category: 技术分享
draft: false
image: ./images/firefly1.avif
---

# **Excel 导出**

> **ExportQueue队列**

```java
package com.example.system.config;

import com.example.system.api.domain.ExportUser;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

import java.util.LinkedList;

@Slf4j
@Component
public class ExportQueue {

 private final int MAX_CAPACITY = 10; // 队列最大容量
 private LinkedList<ExportUser> queue; // 用户队列

 public ExportQueue(LinkedList<ExportUser> queue) {
 this.queue = new LinkedList<>();
 }

 /**
 * 排队队列添加
 * @param sysUser
 */
 public synchronized LinkedList<ExportUser> add(ExportUser sysUser) {
 while (queue.size() >= MAX_CAPACITY) {
 try {
 log.info("当前排队人已满，请等待");
 wait();
 } catch (InterruptedException e) {
 e.getMessage();
 }
 }
 queue.add(sysUser);
 log.info("目前导出队列排队人数：" + queue.size());
 notifyAll();
 return queue;
 }

 /**
 * 获取排队队列下一个人
 * @return
 */
 public synchronized ExportUser getNextSysUser() {
 while (queue.isEmpty()) {
 try {
 wait();
 } catch (InterruptedException e) {
 e.printStackTrace();
 }
 }
 ExportUser sysUser = queue.remove();
 notifyAll(); //唤醒
 return sysUser;
 }
}
```

> **AbstractExport导出类**

```java
package com.example.system.config;

import cn.hutool.core.bean.BeanUtil;
import cn.hutool.core.util.PageUtil;
import com.alibaba.excel.EasyExcel;
import com.alibaba.excel.ExcelWriter;
import com.alibaba.excel.write.metadata.WriteSheet;
import com.example.system.api.domain.ExportUser;
import lombok.extern.slf4j.Slf4j;

import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.net.URLEncoder;
import java.util.List;

@Slf4j
public abstract class AbstractExport<T, K> {

 public abstract void export(ExportUser sysUser) throws InterruptedException;

 /**
 * 导出
 *
 * @param response 输出流
 * @param pageSize 每页大小
 * @param t        导出条件
 * @param k        Excel内容实体类
 * @param fileName 文件名称
 */
 public void export(HttpServletResponse response, int pageSize, T t, Class<K> k, String fileName) throws Exception {
 ExcelWriter writer = null;
 try {
 writer = getExcelWriter(response, fileName);
 //查询导出总条数
 int total = this.countExport(t);
 //页数
 int loopCount = PageUtil.totalPage(total, pageSize);
 BeanUtil.setProperty(t, "pageSize", pageSize);
 for (int i = 0; i < loopCount; i++) {
 //开始页
 BeanUtil.setProperty(t, "pageNum", PageUtil.getStart(i + 1, pageSize));
 //获取Excel导出信息
 List<K> kList = this.getExportDetail(t);
 WriteSheet writeSheet = EasyExcel.writerSheet(fileName).head(k).build();
 writer.write(kList, writeSheet);
 }
 } catch (Exception e) {
 String msg = "导出" + fileName + "异常";
 log.error(msg, e);
 throw new Exception(msg + e);
 } finally {
 if (writer != null) {
 writer.finish();
 }
 }
 }

 public com.alibaba.excel.ExcelWriter getExcelWriter(HttpServletResponse response, String fileName) throws IOException {
 response.setContentType("application/vnd.ms-excel");
 response.setCharacterEncoding("utf-8");
 // 这里URLEncoder.encode可以防止中文乱码 当然和easyexcel没有关系
 String fileNameUtf = URLEncoder.encode(fileName, "UTF-8").replaceAll("\\\\+", "%20");
 response.setHeader("Content-disposition", "attachment;filename*=utf-8''" + fileNameUtf + ".xlsx");
 return EasyExcel.write(response.getOutputStream()).build();
 }

 /**
 * （模版导出）
 *
 * @param t
 * @param fileName
 * @param response
 */
 public abstract void complexFillWithTable(T t, String fileName, HttpServletResponse response);

 /**
 * 查询导出总条数
 *
 * @param t
 * @return
 */
 public abstract int countExport(T t);

 /**
 * 查询导出数据
 *
 * @param t
 * @return
 */
 public abstract List<K> getExportDetail(T t);
}

```

> **ExportImpl导出实现方法**

```java
package com.example.system.service.impl;

import com.alibaba.excel.ExcelWriter;
import com.example.system.api.domain.ExportUser;
import com.example.system.config.AbstractExport;
import com.example.system.config.ExportQueue;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.LinkedList;
import java.util.List;

@Service
@Slf4j
public class ExportImpl extends AbstractExport {

 @Autowired
 private ExportQueue exportQueue;

 @Override
 public void export(ExportUser sysUser) throws InterruptedException {

 //导出
 log.info("导出文件方法执行～～～～～～～～～");
//        export(response,pageSize,t,k,fileName);
 LinkedList<ExportUser> queue = exportQueue.add(sysUser);
 log.info("导出队列：" + queue);
 //休眠时间稍微设置大点，模拟导出处理时间
 Thread.sleep(20000);
 //导出成功后移除当前导出用户
 ExportUser nextSysUser = exportQueue.getNextSysUser();
 log.info("移除后获取下一个排队的用户: " + nextSysUser.getUserName());

 }

 @Override
 public void export(HttpServletResponse response, int pageSize, Object o, Class k, String fileName) throws Exception {
 super.export(response, pageSize, o, k, fileName);
 }

 @Override
 public ExcelWriter getExcelWriter(HttpServletResponse response, String fileName) throws IOException {
 return super.getExcelWriter(response, fileName);
 }

 @Override
 public void complexFillWithTable(Object o, String fileName, HttpServletResponse response) {

 }

 @Override
 public int countExport(Object o) {
 return 0;
 }

 @Override
 public List getExportDetail(Object o) {
 return null;
 }
}
```

> **测试controller**

```java
package com.example.system.controller;

import com.example.system.api.domain.ExportUser;
import com.example.system.api.domain.SysUser;
import com.example.system.service.impl.ExportImpl;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/export")
@Slf4j
public class ExportController {

 @Autowired
 private ExportImpl export;

 @PostMapping("/exportFile")
 public void exportFile() {
 new Thread(new Runnable() {
 @SneakyThrows
 @Override
 public void run() {
 Thread thread1 = Thread.currentThread();
 ExportUser sysUser =new ExportUser();
 sysUser.setUserName(thread1.getName());

 export.export(sysUser);
 }
 }).start();
 }
}

```
