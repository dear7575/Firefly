---
title: ✨SpringBoot Apache CXF 配置
published: 2024-09-10
pinned: false
description: ""
tags: [SpringBoot, Apache CXF]
category: 技术分享
draft: false
image: ./images/firefly1.avif
---

# SpringBoot Apache CXF 配置

> docker-compose.yml 文件配置

```java
public String sendWebServiceApi(String method, String xml, String message) {
        StopWatch stopWatch = new StopWatch();
        try {
            stopWatch.start();
            //  获取接口密钥
            String ztbKey = ConstantContextHolder.getSysConfig("ZTB_KEY", String.class, true);
            //  获取接口地址
            String ztbWebServiceUrl = ConstantContextHolder.getSysConfigWithDefault("ZTB_WEB_SERVICE_URL", String.class, "<http://127.0.0.1/WebService.asmx?wsdl>");
            //  获取是否开启请求超时
            String ztbWebHttpTimeOut = ConstantContextHolder.getSysConfigWithDefault("ZTB_WEB_REQUEST_TIME_OUT", String.class, "N");
            //  获取请求超时时间
            Integer ztbWebRequestTimeOutTime = ConstantContextHolder.getSysConfigWithDefault("ZTB_WEB_REQUEST_TIME_OUT_TIME", Integer.class, 3);
            //  获取请求超时时间
            Integer ztbWebConnectionTimeOutTime = ConstantContextHolder.getSysConfigWithDefault("ZTB_WEB_CONNECTION_TIME_OUT_TIME", Integer.class, 3);

            // 创建根元素
            Element root = new Element("Request_Packet");

            // 创建并添加Header元素
            Element header = new Element("Header");
            header.addContent(new Element("Key").setText(ztbKey));
            header.addContent(new Element("from").setText("请求者"));
            header.addContent(new Element("instruct").setText(method));

            // 添加Header到根元素
            root.addContent(header);

            // 创建Content元素并添加CDATA内容
            Element content = new Element("Content");
            content.addContent(new CDATA(xml));

            // 添加Content到根元素
            root.addContent(content);

            // 将根元素转换为XML文档
            Document document = new Document(root);

            // 使用XMLOutputter输出XML字符串
            XMLOutputter xmlOutput = new XMLOutputter();
            // 格式化输出
            xmlOutput.setFormat(Format.getPrettyFormat());
            String xmlString = xmlOutput.outputString(document);

            JaxWsDynamicClientFactory dynamicClient = JaxWsDynamicClientFactory.newInstance();
            Object[] objectArray;
            try (Client client = dynamicClient.createClient(ztbWebServiceUrl)) {
                // 设置超时单位为毫秒
                HTTPConduit conduit = (HTTPConduit) client.getConduit();
                HTTPClientPolicy policy = new HTTPClientPolicy();
                policy.setConnectionTimeout((1000 * 60) * ztbWebConnectionTimeOutTime);
                policy.setAllowChunking(false);
                policy.setReceiveTimeout((1000 * 60) * ztbWebRequestTimeOutTime);
                conduit.setClient(policy);
                // 强制设置URL，确保端口号没有丢失
                // conduit.getTarget().getAddress().setValue("<http://127.0.0.1/WebService.asmx>");
                objectArray = client.invoke("ProcessData", xmlString);
            }
            stopWatch.stop();
            log.error("sendWebServiceApi - 执行请求耗时: {} 秒", stopWatch.getTotalTimeSeconds());
            log.info("请求返回结果: {}", objectArray[0].toString());
            return formatXml(objectArray[0].toString());
        } catch (Exception e) {
            stopWatch.stop();
            log.error("WebService接口异常 - 执行请求耗时: {} 秒", stopWatch.getTotalTimeSeconds());
            log.error("WebService接口异常, {}, {},", message, e.getMessage(), e);
            return null;
        }
    }
```

> 以下重点，遇到的坑

```java
// 1.在使用代理的情况下可能导致请求地址无端口号
// 强制设置URL，确保端口号没有丢失
conduit.getTarget().getAddress().setValue("<http://127.0.0.1/WebService.asmx>");

// 2.ProcessData 必须和公布的WebService 一致，大小写不一样都不行
objectArray = client.invoke("ProcessData", xmlString);

```
