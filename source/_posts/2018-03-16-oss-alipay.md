---
title: 阿里云oss上传文件和文件夹
date: 2018-03-16 14:17:10
tags: java
---

### 配置
```
endpoint // oss的上传服务器的配置
accessKeyId // 阿里云的key
accessKeySecret // 阿里云的
bucketName // oss的空间
```

### 引入依赖

```xml
<dependency>
    <groupId>com.aliyun.oss</groupId>
    <artifactId>aliyun-sdk-oss</artifactId>
    <version>2.8.3</version>
</dependency>
```

### 编写工具类

```java
package com.zxy.keeley.filetocdn.kit;

import com.aliyun.oss.ClientException;
import com.aliyun.oss.OSSClient;
import com.aliyun.oss.OSSException;
import com.aliyun.oss.model.CompleteMultipartUploadResult;
import com.aliyun.oss.model.UploadFileRequest;
import com.aliyun.oss.model.UploadFileResult;
import com.zxy.keeley.filetocdn.config.Content;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;

import java.io.*;

/**
 *
 * @author keeley
 * @date 2018/2/7
 */
@Component
public class OSSKit {
    public static final Logger logger = LoggerFactory.getLogger(OSSKit.class);
    public static final String separator = "/";
    /**
     * 上传文件夹
     * @param dir
     * @param key
     * @throws Exception
     */
    public void updateDirectory(String dir, String key) throws Exception{
        if(StringUtils.isEmpty(dir) || StringUtils.isEmpty(key)) {
            return ;
        }
        if(key.startsWith(separator)) {
            key = key.substring(1);
        }
        File[] files = new File(dir).listFiles();
        for(File a:files){
            String fileName = a.getName();
            if(a.isDirectory()){
                updateDirectory(a.getAbsolutePath(), key +separator+ fileName);
            } else {
                uploadFile(a.getAbsolutePath(),key+separator+a.getName());
            }
        }

    }

    /**
     *
     * @param filePath
     */
    private void uploadFile(String filePath,String key) {
        OSSClient client = null;
        try {
            client = new OSSClient(Content.ossEndpoint, Content.ossAccessKeyId, Content.ossAccessKeySecret);
            // 上传文件流
            InputStream inputStream = new FileInputStream(filePath);
            client.putObject(Content.ossBucketName, key, inputStream);
        } catch (FileNotFoundException e) {
            logger.info("FileNotFoundException:",e);
        } finally {
            if (client != null) {
                client.shutdown();
            }
        }
    }

    /**
     * 普通上传
     * @param filePath
     */
    public void uploadFile(String filePath) {
        this.uploadFile(filePath, new File(filePath).getName());
    }

    /**
     * 分段上传
     * @param uploadFile
     * @return
     * @throws IOException
     */
    public String uploadFileMulti(String uploadFile, String key) throws IOException {
        OSSClient client = null;
        try {
            if(key.startsWith(separator)) {
                key = key.substring(1);
            }
            client = new OSSClient(Content.ossEndpoint, Content.ossAccessKeyId, Content.ossAccessKeySecret);
            UploadFileRequest uploadFileRequest = new UploadFileRequest(Content.ossBucketName, key);

            // 待上传的本地文件
            uploadFileRequest.setUploadFile(uploadFile);

            // 设置并发下载数，默认1
            uploadFileRequest.setTaskNum(5);

            // 设置分片大小，默认100KB
            uploadFileRequest.setPartSize(1024 * 1024 * 10);

            // 开启断点续传，默认关闭
            uploadFileRequest.setEnableCheckpoint(true);

            UploadFileResult uploadResult = client.uploadFile(uploadFileRequest);

            CompleteMultipartUploadResult multipartUploadResult =  uploadResult.getMultipartUploadResult();

            return multipartUploadResult.getETag();

        } catch (OSSException oe) {
            logger.error("OSSException",oe);
        } catch (ClientException ce) {
            logger.error("ClientException",ce);
        } catch (Throwable throwable) {
            logger.error("throwable", throwable);
        } finally {
            if (client != null) {
                client.shutdown();
            }
        }
        return null;
    }
}

```
