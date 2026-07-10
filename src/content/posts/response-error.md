---
title: ✨Response 响应流遇到的坑
published: 2024-04-26
pinned: false
description: "记录后端通过 Response 输出文件流时前端获取 Blob 大小为 0 的问题排查和响应头处理方式。"
tags: [Java, Response]
category: 技术分享
draft: false
image: ../uploads/20260110/wl.jpg
---

> 前端获取Blob大小为0代码

```java
private void downloadFileByPath(String path, String fileName, HttpServletResponse response) {
    try {
        // 设置内容格式
        response.setContentType("image/png");
				response.addHeader("Content-Disposition", "attachment;filename=" + URLEncoder.encode(sysFileDTO.get().getFileName(), "UTF-8"));
        response.setHeader("Content-Length", sysFileDTO.get().getFileSize() + "");
        // 内容安全策略
        response.setHeader("Content-Security-Policy", "script-src 'self' 'unsafe-inline' 'unsafe-eval';style-src 'self' 'unsafe-inline';img-src 'self' data:;connect-src 'self' <http://127.0.0.1:8080> data:;font-src 'self';object-src 'self';");
    } catch (UnsupportedEncodingException e) {
        log.error("文件编码异常 : {}", e.getMessage());
    }
    // 将文件流写入输出流
    File file = new File(path);
    if (file.exists()) {
        InputStream is = null;
        try {
            is = new FileInputStream(file);
            IOUtils.copy(is, response.getOutputStream());
            response.getOutputStream().flush();
        } catch (FileNotFoundException e) {
            log.error("文件未找到异常 : {}", path);
            throw new RuntimeException("文件未找到");
        } catch (IOException e) {
            log.error("文件读写异常 : {}", path);
            throw new RuntimeException("文件读写异常");
        } finally {
            try {
                is.close();
            } catch (IOException e) {
                throw new RuntimeException("文件关闭异常");
            }
        }
    } else {
        log.error("文件未找到异常 : {}", path);
        throw new RuntimeException("文件未找到");
    }
}
```

> 前端获取Blob大小正常代码

```java
private void downloadFileByPath(String path, String fileName, HttpServletResponse response) {
    try {
        // 设置内容格式
        response.setContentType("image/png");
				response.addHeader("Content-Disposition", "attachment;filename=" + URLEncoder.encode(sysFileDTO.get().getFileName(), "UTF-8"));
        response.setHeader("Content-Length", sysFileDTO.get().getFileSize() + "");
        // 内容安全策略
        response.setHeader("Content-Security-Policy", "script-src 'self' 'unsafe-inline' 'unsafe-eval';style-src 'self' 'unsafe-inline';img-src 'self' data:;connect-src 'self' <http://127.0.0.1:8080> data:;font-src 'self';object-src 'self';");
    } catch (UnsupportedEncodingException e) {
        log.error("文件编码异常 : {}", e.getMessage());
    }
    // 将文件流写入输出流
    File file = new File(path);
    if (file.exists()) {
        InputStream is = null;
        try {
            is = new FileInputStream(file);
            // 必须使用赋值的方式
            ServletOutputStream outputStream = response.getOutputStream();
            IOUtils.copy(is, outputStream);
            response.getOutputStream().flush();
        } catch (FileNotFoundException e) {
            log.error("文件未找到异常 : {}", path);
            throw new RuntimeException("文件未找到");
        } catch (IOException e) {
            log.error("文件读写异常 : {}", path);
            throw new RuntimeException("文件读写异常");
        } finally {
            try {
                is.close();
            } catch (IOException e) {
                throw new RuntimeException("文件关闭异常");
            }
        }
    } else {
        log.error("文件未找到异常 : {}", path);
        throw new RuntimeException("文件未找到");
    }
}
```

> 也可以使用这种方式

```java
private void downloadFileByPath(String path, String fileName, HttpServletResponse response) {
    try {
        // 设置内容格式
        response.setContentType("image/png");
				response.addHeader("Content-Disposition", "attachment;filename=" + URLEncoder.encode(sysFileDTO.get().getFileName(), "UTF-8"));
        response.setHeader("Content-Length", sysFileDTO.get().getFileSize() + "");
        // 内容安全策略
        response.setHeader("Content-Security-Policy", "script-src 'self' 'unsafe-inline' 'unsafe-eval';style-src 'self' 'unsafe-inline';img-src 'self' data:;connect-src 'self' <http://127.0.0.1:8080> data:;font-src 'self';object-src 'self';");
    } catch (UnsupportedEncodingException e) {
        log.error("文件编码异常 : {}", e.getMessage());
    }
    // 将文件流写入输出流
    File file = new File(path);
    if (file.exists()) {
        InputStream is = null;
        ServletOutputStream outputStream = null;
        try {
            is = new FileInputStream(file);
            // 必须使用赋值的方式
            out = response.getOutputStream();
            // 创建缓冲区
            byte buffer[] = new byte[4096];
            int len = 0;
            // 循环将输入流中的内容读取到缓冲区当中
            while ((len = in.read(buffer)) > 0) {
                // 输出缓冲区的内容到浏览器，实现文件下载
                out.write(buffer, 0, len);
            }
        } catch (FileNotFoundException e) {
            log.error("文件未找到异常 : {}", path);
            throw new RuntimeException("文件未找到");
        } catch (IOException e) {
            log.error("文件读写异常 : {}", path);
            throw new RuntimeException("文件读写异常");
        } finally {
            try {
                is.close();
                out.close();
            } catch (IOException e) {
                throw new RuntimeException("文件关闭异常");
            }
        }
    } else {
        log.error("文件未找到异常 : {}", path);
        throw new RuntimeException("文件未找到");
    }
}
```

> 具体因为 ServletOutputStream outputStream = response.getOutputStream(); 就正常的原因还是未知，没有太多的时间排查。

> 其中还遇到使用日志 **log.error("大小:{}", is.readAllBytes().length);**  输出长度也会影响前端Blob的大小为0，同时图片响应也是异常的。
>
