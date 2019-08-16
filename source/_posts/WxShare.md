---
title: WxShare
tags: 
    wx
date: 2019-04-25 10:04:49
---

# 公众号类型功能介绍
[微信官方介绍](http://kf.qq.com/faq/170815aUZjeQ170815mU7bI7.html)

公众号，是订阅号和服务号的统称。订阅号是公众号、服务号也是公众号。

订阅号：偏于为用户传达资讯，每天只可以发送一条群发，用户不会收到即时消息提醒，因为所有的订阅号都被收在了订阅号文件夹中。订阅号适用于个人和组织。
想看一个订阅号的消息，有两种方法：
1. 打开订阅号文件夹查看订阅号有没有更新消息；
2. 打开微信手机通讯录，点开公众号选项，是你关注的所有公众号(包括订阅号和服务号)。

服务号：偏于服务交互(如：招商银行信用卡、微信读书、京东白条)，每月可群发四条消息，收费，不适用于个人，微信支付需要**微信认证服务号**才可以。

选择：
想简单的发送消息，达到宣传效果，建议选择的订阅号。
想用公众号获得更多功能，如开通微信支付，建议选择服务号。

服务号、订阅号功能对比：
![](WxShare/1.jpg)

# 微信开放平台、公众平台区别
## 公众平台
[官网](https://mp.weixin.qq.com/)
[技术文档](https://mp.weixin.qq.com/wiki)

微信公众平台是运营者通过公众号为微信用户提供资讯和服务的平台，登录公众平台账号后，可以看到它有一个不错的交互界面。可以提供给公司的运营人员使用，用来发布消息和提供服务。

## 开放平台
[官网](https://open.weixin.qq.com/)
是为开发者（程序员）提供的一个平台，在这里你可以将你的公众平台下的公众号（订阅号、服务号）绑定到你的开放平台账号下，从而可以基于订阅号、服务号做更多的开发。公众号中的订阅号接口权限是有限的，比如它无法获得网页授权的权限，也就无法通过网页授权获取用户的基本信息（比如openID、unionID等）。

# 概述
[参考地址](https://blog.csdn.net/change_on/article/details/75264843)

流程图：
![](WxShare/2.png)

步骤：
公众号
已经备案好的域名

1. 配置JS回调域名
2. 获取appId和secret
3. 从官方代码copy签名函数
4. 获取access_token、ticket
5. 获取url，并进行签名
6. 集成进java web
7. 前端config函数配置

## 获取appId和secret
可以在基本配置中找到AppID和AppSecret。
登录微信公众平台->公众号设置->功能设置->JS接口安全域名。填写你要分享的网页所在的域名。

## 从官方代码copy签名函数
打开[微信分享官方文档](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421141115)，在页面最下面找到并下载[示例代码](http://demo.open.weixin.qq.com/jssdk/sample.zip)
我们将java中的`sign.java`直接放到项目中备用。

## 获取access_token、ticket
```java
public class WxShareUtil {

    public static WxShare getWinXinEntity(String url) {
        WxShare wx = new WxShare();
        String access_token = getAccessToken();
        String ticket = getTicket(access_token);
        Map<String, String> ret = Sign.sign(ticket, url);
        //System.out.println(ret.toString());
        wx.setTicket(ret.get("jsapi_ticket"));
        wx.setSignature(ret.get("signature"));
        wx.setNoncestr(ret.get("nonceStr"));
        wx.setTimestamp(ret.get("timestamp"));
        return wx;
    }

    //获取token
    private static String getAccessToken() {
        String access_token = "";
        String grant_type = "client_credential";//获取access_token填写client_credential
        String AppId="XXXXXXXXXXXXXXXXXXX";//第三方用户唯一凭证
        String secret="XXXXXXXXXXXXXXXXXXXXXXXXXXX";//第三方用户唯一凭证密钥，即appsecret
        //这个url链接地址和参数皆不能变
        String url = "https://api.weixin.qq.com/cgi-bin/token?grant_type="+grant_type+"&appid="+AppId+"&secret="+secret;  //访问链接

        try {
            URL urlGet = new URL(url);
            HttpURLConnection http = (HttpURLConnection) urlGet.openConnection();
            http.setRequestMethod("GET"); // 必须是get方式请求
            http.setRequestProperty("Content-Type","application/x-www-form-urlencoded");
            http.setDoOutput(true);
            http.setDoInput(true);
            /*System.setProperty("sun.net.client.defaultConnectTimeout", "30000");// 连接超时30秒
            System.setProperty("sun.net.client.defaultReadTimeout", "30000"); // 读取超时30秒 */
            http.connect();
            InputStream is = http.getInputStream();
            int size = is.available();
            byte[] jsonBytes = new byte[size];
            is.read(jsonBytes);
            String message = new String(jsonBytes, "UTF-8");
            JSONObject demoJson = JSONObject.fromObject(message);
            access_token = demoJson.getString("access_token");
            is.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return access_token;
    }

    //获取ticket
    private static String getTicket(String access_token) {
        String ticket = null;
        String url = "https://api.weixin.qq.com/cgi-bin/ticket/getticket?access_token="+ access_token +"&type=jsapi";//这个url链接和参数不能变
        try {
            URL urlGet = new URL(url);
            HttpURLConnection http = (HttpURLConnection) urlGet.openConnection();
            http.setRequestMethod("GET"); // 必须是get方式请求
            http.setRequestProperty("Content-Type","application/x-www-form-urlencoded");
            http.setDoOutput(true);
            http.setDoInput(true);
            System.setProperty("sun.net.client.defaultConnectTimeout", "30000");// 连接超时30秒
            System.setProperty("sun.net.client.defaultReadTimeout", "30000"); // 读取超时30秒
            http.connect();
            InputStream is = http.getInputStream();
            int size = is.available();
            byte[] jsonBytes = new byte[size];
            is.read(jsonBytes);
            String message = new String(jsonBytes, "UTF-8");
            JSONObject demoJson = JSONObject.fromObject(message);
            ticket = demoJson.getString("ticket");
            is.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return ticket;
    }

}
```

## 获取url，并进行签名
url的获取要特别注意，因为微信在分享时会在原来的url上加上一些&from=singlemessage、&from=…的，所以url必须要动态获取，网上有些用ajax，
在网页通过“location.href.split(‘#’)“ 获取url，在用ajax传给后台，这种方法可行，但是不推荐，一方面用ajax返回，就要访问分享的逻辑，
这样后台分享的逻辑增加复杂度，带来不便，是代码不易于维护，可读性低！另一方面分享是返回页面，而ajax是返回json，又增加了复杂度。
所以，一个java程序员是不会通过ajax从前台获取url的，这里我们用HttpRequest的方法即可，不管微信加多少后缀，都可以获取到完整的当前url。

获取url的代码如下，只给出核心代码（代码位于处理分享的controller中）：
```
public String share(HttpServletRequest request) {
    ...

    String strUrl = "http://www.xxxxx.com"       //换成安全域名
                    + request.getContextPath()   //项目名称  
                    + request.getServletPath()   //请求页面或其他地址  
                    + "?" + (request.getQueryString()); //参数  

    ...
}
```

我们再新建一个存放微信信息的实体类：WxShare.java
```java
public class WinXinEntity {

    private String access_token;
    private String ticket ;
    private String noncestr;
    private String timestamp;
    private String str;
    private String signature;

   ...setter and getter...
}
```

sign类里面有一个签名的方法public static Map<String, String> sign(String jsapi_ticket, String url)

传入ticket和url即可。也就是WeinXinUtil的getWinXinEntity方法，并将返回的map的信息读取存入WinXinEntity 中。
在调试时，把sign返回的map打印出来，主要看看生成的signature。然后，把jsapi_ticket、noncestr、timestamp、url 复制到微信提供的[微信 JS 接口签名校验工具](https://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=jsapisign)，
比较代码签名生成的signature与校验工具生成的签名signature是否一致，如果一致，说明前面的步骤都是正确的，如果不一致，仔细检查！

## 
我们把微信分享分解成3个工具类，现在在处理分享的controller，只要两句话就可以调用微信分享，一句获取url，一句获取WinXinEntity，下面是核心代码：
```java
//微信分享
String strUrl = "http://www.xxxxx.com"
		+ request.getContextPath()   //项目名称  
		+ request.getServletPath()   //请求页面或其他地址  
		+ "?" + (request.getQueryString()); //参数  
WinXinEntity wx = WeinXinUtil.getWinXinEntity(strUrl);
//将wx的信息到给页面
request.setAttribute("wx", wx);
```

## 前端config函数配置
下面的代码放在网页js代码的最前面！
”var url = location.href.split(‘#’)[0];“ 页面的url也可以从后台传过来，也可以通过location.href.split(‘#’)[0]获取。
为了一点微不足道的速度，这里才用网页获取方式。（网页的url跟前面的后台签名时得url是一样的，只是绕过了ajax）

```js
<script src="http://res.wx.qq.com/open/js/jweixin-1.2.0.js"></script>
<script>
var url = location.href.split('#')[0];
    wx.config({
    debug: false,
    appId: 'xxxxxxxxxxxxxx',
    timestamp: "${wx.timestamp}",
    nonceStr: "${wx.noncestr}",
    signature: "${wx.signature}",
    jsApiList: [
      // 所有要调用的 API 都要加到这个列表中
       'checkJsApi',
       'onMenuShareTimeline',
       'onMenuShareAppMessage',
    ]
  });
  wx.ready(function () {
    // 在这里调用 API
     wx.onMenuShareTimeline({
    title: 'xxxxxxxxxx', // 分享标题
    link: url, // 分享链接，该链接域名或路径必须与当前页面对应的公众号JS安全域名一致
    imgUrl: 'xxxxxxxxxxxxxx', // 分享图标
    success: function () {
        // 用户确认分享后执行的回调函数
    },
    cancel: function () {
        // 用户取消分享后执行的回调函数
    }
});
wx.onMenuShareAppMessage({
   title: 'xxxxxxxxxxx', // 分享标题
    desc: 'xxxxxxxxxxx', // 分享描述
    link: url, // 分享链接，该链接域名或路径必须与当前页面对应的公众号JS安全域名一致
    imgUrl: 'xxxxxxxxxx', // 分享图标
    type: '', // 分享类型,music、video或link，不填默认为link
    dataUrl: '', // 如果type是music或video，则要提供数据链接，默认为空
    success: function () {
        // 用户确认分享后执行的回调函数
    },
    cancel: function () {
        // 用户取消分享后执行的回调函数
    }
});
});
</script>
```
