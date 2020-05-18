title: 百度云推送分析
layout: post
comments: true
keywords: baidupush
categories: APP推送
date: 2016-01-09 20:17:09
tag: 技术分析
---

## 百度云推送介绍
### 云推送原理
云推送（Push）是百度开放云向开发者提供的消息推送服务,通过利用百度云端与客户端之间建立稳定、可靠的长连接来为开发者提供向客户端应用推送实时消息的一种开发者服务

android的流程
服务端 － 百度云 － app端

ios的流程
服务端 － 百度云 － 苹果APNS服务 － app端

<!-- more -->

### app端
> app端负责接收百度云推送的消息接受

只有成功集成了百度云推送客户端的SDK的应用才能接收到由"云推送"服务下发的消息
只支持 `Android`及`iOS`两个平台

### 服务器端
> 服务器端负责提供推送数据，将数据发送到百度云推送平台。

提供2种方式
1. 利用百度云提供的管理控制台来管理应用的推送，能实现所有推送功能和大部分统计功能
2. 利用百度云提供的服务接口集成到自己的后台中
3. 无论哪种方式，数据都是传递到百度云，经百度云再转发到设备

## 百度云推送的功能
### 服务端功能
#### 推送内容
* 推送系统通知栏通知，标题和内容

```
可设置点击通知的行为，1：打开Url; 2：自定义行为；3：默认打开应用;

可以设置通知的基本样式包括(响铃：0x04;振动：0x02;可清除：0x01;),这是一个flag整形，每一位代表一种样式
```

* 推送透传消息，以静默的方式直接发送消息到客户端，然后客户端在进行后续动作
* ~~推送富媒体消息~~（目前android测试的情况是，直接用富文本编辑器编辑的一个hmtl文本，然后将这个文本压缩成rar之后发送到app端，由于虚拟机没有sd卡，所以能收到通知但是无法看到富文本）

#### 推送方式
* 添加标签（createTag）
* 删除标签（deleteTag）
* 向标签组中批量添加设备（addDevicesToTag）
* 从标签组中批量删除设备（deleteDevicesFromTag）
* 基于标签推送（pushMsgToTag）

#### 统计
* (查看标签组中关联的设备)queryDeviceNumInTag
* (统计所有推送消息信息,返回30天内的)queryStatisticMsg
* (统计某个消息的30天内的)queryStatisticTopic
* (统计安装了app的设备数,返回30天的所有统计项)queryStatisticDevice

经测试，调用统计接口发送的消息，通过服务端api调用方式消息之后立即查询统计都无效，需要隔一天之后才能查询。
通过百度控制台发送的，控制台也只有一个推送消息信息的发送列表，其他三个也没有。而且30天这个时间段还不能自己定义，百度提供的统计功能还是比较薄弱的，实际使用需要自己开发管理统计。

### 客户端
* 接受通知 Android/iOS
* 接收透传消息 Android/~~IOS~~

透传消息相当于直接的长链接请求，ios因为需要先通过ios的APNS服务然后再到设备，所以不支持透传。~~不过百度推送官网提示即将支持ios~~

## APP端集成方式
### SDK下载
官网sdk下载 [http://push.baidu.com/sdk/push_client_sdk_for_android](http://push.baidu.com/sdk/push_client_sdk_for_android)

### 在百度应用中心创建应用
创建应用不需要传递代码等东西，只是为了获取百度的关联关系
创建应用之后能得到API KEY和SECRET KEY，apikey是标志，客户端和服务端都会配置这个key，然后百度云端才能找到对应的app。
创建应用的地址如下[http://push.baidu.com/console/app](http://push.baidu.com/console/app)

### Android
官方参考文档 [http://push.baidu.com/doc/android/api](http://push.baidu.com/doc/android/api)

主要流程
绑定到百度云通知服务，和接收云通知。

1 通过`PushManager`启动推送,关联到百度云

```
// 在当前工程的主Activity的onCreate函数中，添加以下代码：
PushManager.startWork(getApplicationContext(),PushConstants.LOGIN_TYPE_API_KEY,"api_key")
```

2 继承`PushMessageReceiver`来创建接收请求类
其中一些主要的方法

```java
// 调用PushManager.startWork后，sdk将对push server发起绑定请求，这个过程是异步的。绑定请求的结果通过onBind返回。 如果您需要用单播推送，需要把这里获取的channel id和user id上传到应用server中，再调用server接口用channel id和user id给单个手机或者用户推送。

public void onBind(Context context, int errorCode, String appid, String userId, String channelId, String requestId);

//接收透传消息的函数-- onMessage
public void onMessage(Context context, String message, String customContentString);

//通知到达时，当通知被用户点击时，会回调onNotificationClicked函数
public void onNotificationClicked(Context context, String title, String description, String customContentString)

```
### IOS
*因为ios需要开发者证书（个人版一年688rmb），未进行ios的测试。*

### 服务端api
#### api参考地址
[http://push.baidu.com/doc/java/api](http://push.baidu.com/doc/java/api)

都是一些调用推送例子，一目了然。

* 定时推送   
定时推送的sendtime参数，是根据时间戳来进行的，只能指定时间之后定时发送一次，如果做复杂点的，指定时间重复推送，需要后台自己实现定时方法来调用。   

```java
// 设置定时推送时间，必需超过当前时间一分钟，单位秒.实例70秒后推送
 addSendTime(System.currentTimeMillis() / 1000 + 70).
```

#### 单个推送举例
```java
package com.baidu.yun.push.sample;

import com.baidu.yun.core.log.YunLogEvent;
import com.baidu.yun.core.log.YunLogHandler;
import com.baidu.yun.push.auth.PushKeyPair;
import com.baidu.yun.push.client.BaiduPushClient;
import com.baidu.yun.push.constants.BaiduPushConstants;
import com.baidu.yun.push.exception.PushClientException;
import com.baidu.yun.push.exception.PushServerException;
import com.baidu.yun.push.model.PushMsgToSingleDeviceRequest;
import com.baidu.yun.push.model.PushMsgToSingleDeviceResponse;

public class PushMsgToSingleDevice {
    public static void main(String[] args) 
            throws PushClientException,PushServerException {
        /*1. 创建PushKeyPair
         *用于app的合法身份认证
         *apikey和secretKey可在应用详情中获取
         */
        String apiKey = “xxxxxxxxxxxxxxxxxxxxxx”;
        String secretKey = “xxxxxxxxxxxxxxxxxxxxxx”;
        PushKeyPair pair = new PushKeyPair(apiKey,secretKey);

        // 2. 创建BaiduPushClient，访问SDK接口
        BaiduPushClient pushClient = new BaiduPushClient(pair,
                BaiduPushConstants.CHANNEL_REST_URL);

        // 3. 注册YunLogHandler，获取本次请求的交互信息
        pushClient.setChannelLogHandler (new YunLogHandler () {
            @Override
            public void onHandle (YunLogEvent event) {
                System.out.println(event.getMessage());
            }
        });

        try {
        // 4. 设置请求参数，创建请求实例
            PushMsgToSingleDeviceRequest request = new PushMsgToSingleDeviceRequest().
                addChannelId("xxxxxxxxxxxxxxxxxx").
                addMsgExpires(new Integer(3600)).   //设置消息的有效时间,单位秒,默认3600*5.
                addMessageType(1).              //设置消息类型,0表示透传消息,1表示通知,默认为0.
                add("{\"title\":\"TEST\",\"description\":\"Hello Baidu push!\"}").
                addDeviceType(3);      //设置设备类型，deviceType => 1 for web, 2 for pc, 
                                       //3 for android, 4 for ios, 5 for wp.
        // 5. 执行Http请求
            PushMsgToSingleDeviceResponse response = pushClient.
                pushMsgToSingleDevice(request);
        // 6. Http请求返回值解析
            System.out.println("msgId: " + response.getMsgId()
                    + ",sendTime: " + response.getSendTime());
        } catch (PushClientException e) {
            //ERROROPTTYPE 用于设置异常的处理方式 -- 抛出异常和捕获异常,
            //'true' 表示抛出, 'false' 表示捕获。
            if (BaiduPushConstants.ERROROPTTYPE) { 
                throw e;
            } else {
                e.printStackTrace();
            }
        } catch (PushServerException e) {
            if (BaiduPushConstants.ERROROPTTYPE) {
                throw e;
            } else {
                System.out.println(String.format(
                        "requestId: %d, errorCode: %d, errorMsg: %s",
                        e.getRequestId(), e.getErrorCode(), e.getErrorMsg()));
            }
        }
    }
}                  
```
#### 网络通讯

百度云提供的接口是以REST API协议发布的，依托http+json形式。

上述第二步的 BaiduPushClient，配置了服务端与百度云通信的访问域名
有2个可选`api.push.baidu.com`；`api.tuisong.baidu.com`

真实拼凑的访问百度云推送接口地址代码如下。

```java
private String obtainIntegrityUrl(String classType, String method) {
   String resurl = host;
   if (host.startsWith("http://") || host.startsWith("https://")) {
   } else {
      resurl = "http://" + host;
   }
   if (resurl.endsWith("/")) {
      resurl = resurl + "rest/3.0/";
   } else {
      resurl = resurl + "/rest/3.0/";
   }
   resurl = resurl + classType + "/" + method;
   return resurl;
}
```

由代码可知，需要访问`api.push.baidu.com`或`api.tuisong.baidu.com`域名的80端口，走http协议。默认使用http模式。如果服务器网络架构问题需要做代理就对这个地址和端口做对应处理（由于家里环境问题没有成功模拟出来，但是它走接口的形势就是http请求的，公司有类似项目的情况一样的都能一样的处理）。
## 常见问题
####通知和透传消息的区别？
通知会直接在通知栏弹出展示，消息则由应用控制。
####应用关闭或结束进程后，还能收到推送吗？
应用退至后台或结束进程，百度云推送的Service会继续在后台运行并接收推送；部分情况下使用安全软件或内存管理工具强制清理后台，Service会被清除，但会快速重启；在小米和魅族手机上，用户清理后台应用后必须要等到再次打开app，开发者调用StartWork之后，Service才会在后台启动并继续接收推送。


## 行业对比和评测
以上是本人分析的百度云推送接入方式，和细节，android虚拟机测试时,消息也是瞬间到达的。

评测[http://www.devstore.cn/evaluation/testInfo/119-180.html](http://www.devstore.cn/evaluation/testInfo/119-180.html)

知乎对比 [https://www.zhihu.com/question/20628786](https://www.zhihu.com/question/20628786)







