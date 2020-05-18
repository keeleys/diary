---
title: linux上部署文档转换的服务
date: 2018-08-08 13:50:29
tags: linux
---


## 介绍

unoconv，全称为 Universal Office Converter ，是一个命令行工具，可在linux平台,对LibreOffice/OpenOffice 支持的任意文件格式之间进行转换

<!-- more -->
## 使用

一般使用这个会需要配置很多运行环境,为了快速开发网上找了一个部署好环境的docker镜像.转换的事情直接docker做好了,我们只需要加工一下,处理消息的传入传出和错误处理就好了

https://hub.docker.com/r/zrrrzzt/docker-unoconv-webservice/
https://github.com/keeleys/docker-unocnv

docker run -d -p 80:3000 --name unoconv docker-unoconv-webservice

## 具体使用

该docker版本的unoconv是已服务的方式进行部署的,我们用的时候,需要将待转换的文旦上传到转换服务,转换服务成功之后返回pdf流,然后我们将pdf流输出成未见.

```s

curl --form "file=@/Users/keeley/Downloads/BP神经网络详解-最好的版本.ppt" http://localhost:3500/unoconv/pdf > "BP神经网络详解-最好的版本.ppt.pdf"
```


## java中的使用

>>将该文档转换直接集成到原来的mp4/ZIP转换项目去了

* sass端上传文档 ,发送上传消息
* file-message-listener接收到消息,将office文件传递给文档转换服务进行转换,
* 判断是否返回200,有就是成功,并将输出流转换成pdf存放,没有就是失败,返回错误

## 代码

### 处理的代码

```java
public class HttpKit {
    public static boolean postFile(String filePath, String url) throws IOException {
        File file = new File(filePath);
        CloseableHttpClient httpclient = HttpClients.createDefault();
        CloseableHttpResponse response = null;
        try {
            HttpPost httpPost = new HttpPost(url);
            MultipartEntityBuilder mEntityBuilder = MultipartEntityBuilder.create();
            mEntityBuilder.addBinaryBody("file", file);
            httpPost.setEntity(mEntityBuilder.build());
            response = httpclient.execute(httpPost);

            int statusCode = response.getStatusLine().getStatusCode();
            if (statusCode == HttpStatus.SC_OK) {
                HttpEntity resEntity = response.getEntity();
                // 消耗掉response
                File destFile = new File(file.getPath()+".pdf");
                FileUtils.copyInputStreamToFile(resEntity.getContent(), destFile);
                return true;
            }
        } catch (ParseException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            HttpClientUtils.closeQuietly(httpclient);
            HttpClientUtils.closeQuietly(response);
        }
        return false;
    }
}

```

### 接受处理的代码(FileBackListener)

将转换过的代码存成新的attachment记录,和原来逻辑保持一致

```java
private void pdfProcess(String fileId, String filePath) {
    Attachment attachment = fileService.get(fileId).orElse(null);

    Attachment attachment1 = new Attachment();
    attachment1.forInsert();
    attachment1.setPath(filePath);
    attachment1.setContentType("application/octet-stream");
    attachment1.setExtention("pdf");
    attachment1.setFilename(attachment.getFilename());
    attachment1.setSize(attachment.getSize());
    fileService.insert(attachment1);

    fileService.update(fileId, Optional.of(attachment1.getId()), Optional.of(successStatus));
}
```

### tips

1. txt格式乱码问题(处理方案是将txt的文本正确读取出来,然后转成docx在进行转换)

```java
package com.zxy.fastdfs.convert.util;
 
import org.apache.commons.io.FileUtils;
import org.apache.poi.xwpf.usermodel.XWPFDocument;
import org.apache.poi.xwpf.usermodel.XWPFParagraph;
import org.apache.poi.xwpf.usermodel.XWPFRun;
 
import java.io.*;
 
public class TextUtil {
    private static String codeString(String fileName) throws IOException {
        BufferedInputStream bin = new BufferedInputStream(new FileInputStream(fileName));
        int p = (bin.read() << 8) + bin.read();
        bin.close();
        String code;
 
        switch (p) {
            case 0xefbb:
                code = "UTF-8";
                break;
            case 0xfffe:
                code = "Unicode";
                break;
            case 0xfeff:
                code = "UTF-16BE";
                break;
            default:
                code = "GBK";
        }
 
        return code;
    }
 
    public static boolean parseTextToDocx(String source, String desc) throws IOException {
        FileOutputStream out = null;
        try {
            String str = FileUtils.readFileToString(new File(source), codeString(source));
            System.out.println(str);
            XWPFDocument document = new XWPFDocument();
            out = new FileOutputStream(new File(desc));
            XWPFParagraph paragraph = document.createParagraph();
            XWPFRun run = paragraph.createRun();
            run.setText(str);
            document.write(out);
            return true;
        } catch (Exception e){
            return false;
        }finally {
            if(out!=null){
                out.close();
            }
        }
    }
}
```

### 软件下载
* [libreoffice](https://downloadarchive.documentfoundation.org/libreoffice/old/4.3.7.2/mac/x86_64/LibreOffice_4.3.7.2_MacOS_x86-64.dmg)