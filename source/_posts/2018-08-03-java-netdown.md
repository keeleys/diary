---
title: java远程下载
date: 2018-08-03 13:53:11
tags: java
---

```java
java.net.URL url = new java.net.URL(path);
logger.info("从url下载流:{}",url);
InputStream inputStream = url.openConnection().getInputStream();
HttpHeaders headers = new HttpHeaders();
headers.setContentLength(attachment.getSize());
headers.set(HttpHeaders.CONTENT_TYPE, attachment.getContentType());
headers.set(HttpHeaders.CONTENT_DISPOSITION, "attachment;filename=fileName");
headers.setCacheControl("public");
InputStreamResource resource = new InputStreamResource(inputStream);
ResponseEntity<InputStreamResource> response = new ResponseEntity<>(resource, headers, HttpStatus.OK);
return response;
```