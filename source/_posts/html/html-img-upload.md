title: 图片上传显示预览
tags: html
categories: html
description: 兼容性还不错，试了几个浏览器.
date: 2015-06-29 10:06:46
---

### 例子
<!-- more -->
```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
    <script type="text/javascript" src="http://libs.baidu.com/jquery/1.10.2/jquery.min.js"></script>
    <script type="text/javascript">
    $(function(){
        $(".file").change(function(){
            var fileImg = $(".fileImg");
            var explorer = navigator.userAgent;
            var imgSrc = $(this)[0].value;
            if (explorer.indexOf('MSIE') >= 0) {
                if (!/\.(jpg|jpeg|png|JPG|PNG|JPEG)$/.test(imgSrc)) {
                    imgSrc = "";
                    fileImg.attr("src","/img/default.png");
                    return false;
                }else{
                    fileImg.attr("src",imgSrc);
                }
            }else{
                if (!/\.(jpg|jpeg|png|JPG|PNG|JPEG)$/.test(imgSrc)) {
                    imgSrc = "";
                    fileImg.attr("src","/img/default.png");
                    return false;
                }else{
                    var file = $(this)[0].files[0];
                    var url = URL.createObjectURL(file);
                    fileImg.attr("src",url);
                }
            }
        })
    })
    </script>
</head>
<body>
    <form enctype="multipart/form-date" method="post">
        <input type="file" class="file">
        <input type="submit" class="sub">
    </form>
    <img src="" class="fileImg">
</body>
</html>
```