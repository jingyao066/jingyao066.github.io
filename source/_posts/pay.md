---
title: pay
tags: pay
date: 2018-12-24 13:50:44
---

支付宝支付官网：
https://docs.open.alipay.com/catalog
微信支付官网：
https://pay.weixin.qq.com/wiki/doc/api/index.html

# 设计思路
策略模式+工厂模式，工厂方法用于创建不同场景下的策略。类图如下：
![](pay/1.jpg)
策略接口`IPayStrategy`定义了需要子类实现的方法 ，每一种支付方式为一种策略。
场景类通过枚举类获取策略类型，通过PayFactory创建场景类需要的支付策略。
工厂方法可以创建默认商户的支付策略，也可以在工厂方法里再写一个使用指定商户id或第三方支付的沙箱环境的方法 ，payBaseModel为一些业务参数，例如标题，描述，回调接口以及支付完的跳转页面等。

# maven依赖
```xml
<dependency>
    <groupId>com.alipay.sdk</groupId>
    <artifactId>alipay-sdk-java</artifactId>
    <version>3.0.52.ALL</version>
</dependency>

<dependency>
    <groupId>com.wxpay.sdk</groupId>
    <artifactId>wxpay-sdk</artifactId>
    <version>3.0.9</version>
</dependency>

<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5</version>
</dependency>

<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.3.1</version>
</dependency>

 <dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.3.1</version>
</dependency>
```
由于微信最新的支付sdk在maven仓库里已经没有，合并为了weixin-java-tools里的一个模块，详见
https://github.com/wechat-group/WxJava
本文所使用的微信支付sdk是官网下载后自己编译的。

# 关键类
## 策略接口
```
public interface IPayStrategy {
    public Map<String,String> excutePayOrder(PayBaseModel model) throws Exception;
    public int excuteQueryStatus(String orderNo) throws Exception;
}
```

## 业务参数类
```java
public class PayBaseModel implements Serializable {
    private static final long serialVersionUID = 1L;
    
    //商品标题
    private String title;
    //商品描述
    private String desc;
    //回调地址
    private String notifyUrl;
    //交易金额
    private String tradeMoney;
    //商户订单号
    private String outTradeNo;
    //自定义参数
    private String attach;
    //网页支付完成后的跳转地址
    private String returnUrl;
    //用户ip
    private String realIp;
    //请求头
    private String referer;
    ...
    ...
    //get set方法自己实现 也可以不用实现，因为构造函数里传入了
    public PayBaseModel(String title, String notifyUrl, String tradeMoney, String outTradeNo, String returnUrl) {
        this.title = title;
        this.notifyUrl = notifyUrl;
        this.tradeMoney = tradeMoney;
        this.outTradeNo = outTradeNo;
        this.returnUrl = returnUrl;
    }
 
    public PayBaseModel(String title, String notifyUrl, String tradeMoney, String outTradeNo) {
        this.title = title;
        this.notifyUrl = notifyUrl;
        this.tradeMoney = tradeMoney;
        this.outTradeNo = outTradeNo;
    }
}
```

## 四种支付策略
### AliAppPayStrategy
```java
import com.alipay.api.AlipayApiException;
import com.alipay.api.AlipayClient;
import com.alipay.api.AlipayResponse;
import com.alipay.api.DefaultAlipayClient;
import com.alipay.api.domain.AlipayTradeAppPayModel;
import com.alipay.api.request.AlipayTradeAppPayRequest;
import com.tubitu.util.pay.IPayStrategy;
import com.tubitu.util.pay.PayConstant;
import com.tubitu.util.pay.model.PayBaseModel;
import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
 
import java.math.BigDecimal;
import java.text.DecimalFormat;
import java.util.HashMap;
import java.util.Map;

//支付宝APP支付
public class AliAppPayStrategy implements IPayStrategy {
 
    private String url;
    private String appId;
    private String privateKey;
    private String publicKey;
    private int orderTimeOUt;
    private final static Logger LOGGER = LoggerFactory.getLogger(AliAppPayStrategy.class);

    public AliAppPayStrategy(String url, String appId, String privateKey, String publicKey, int orderTimeOUt) {
        this.url = url;
        this.appId = appId;
        this.privateKey = privateKey;
        this.publicKey = publicKey;
        this.orderTimeOUt = orderTimeOUt;
    }
 
    @Override
    public Map<String, String> excutePayOrder(PayBaseModel payBaseModel) throws AlipayApiException {
        LOGGER.info("**********************获取支付宝APP支付模板" + payBaseModel.toString()
                + "**************************************************");
        Map<String, String> resultMap = new HashMap<>();
        AlipayTradeAppPayRequest payRequest = new AlipayTradeAppPayRequest();
        AlipayTradeAppPayModel model = new AlipayTradeAppPayModel();
        //组装必填参数：订单标题 商家交易订单号 交易金额
        model.setSubject(payBaseModel.getTitle());
        model.setOutTradeNo(payBaseModel.getOutTradeNo());
        if (!StringUtils.isEmpty(payBaseModel.getDesc())) {
            //交易详情
            model.setBody(payBaseModel.getDesc());
        }
        model.setTotalAmount(new DecimalFormat("0.00").format(new BigDecimal(payBaseModel.getTradeMoney())));
        if (!StringUtils.isEmpty(payBaseModel.getAttach())) {
            //自定义参数
            model.setPassbackParams(payBaseModel.getAttach());
        }
        //非必填参数 订单过期时间
        model.setTimeoutExpress(this.orderTimeOUt + "m");
        //app支付交易码-固定值
        model.setProductCode("QUICK_MSECURITY_PAY");
        //回调通知
        payRequest.setNotifyUrl(payBaseModel.getNotifyUrl());
        //业务参数模板
        payRequest.setBizModel(model);
        AlipayClient alipayClient = new DefaultAlipayClient(this.url, this.appId, this.privateKey,
                PayConstant.ALI_DEFAULT_FORMAT, PayConstant.DEFAULT_CHARSET, this.publicKey, PayConstant.ALI_DEFAULT_SIGNTYPE);
        AlipayResponse response = alipayClient.sdkExecute(payRequest);
        resultMap.put("payContent", response.getBody());
        resultMap.put("out_trade_no", payBaseModel.getOutTradeNo());
        LOGGER.info("**********************阿里支付下单接口返回数据:" + resultMap + "**************************************************");
        return resultMap;
    }
}
```

### AliWebPayStrategy
```
import com.alibaba.fastjson.JSONObject;
import com.alipay.api.AlipayApiException;
import com.alipay.api.AlipayClient;
import com.alipay.api.DefaultAlipayClient;
import com.alipay.api.request.AlipayTradeWapPayRequest;
import com.tubitu.util.pay.IPayStrategy;
import com.tubitu.util.pay.PayConstant;
import com.tubitu.util.pay.model.PayBaseModel;
 
import java.math.BigDecimal;
import java.text.DecimalFormat;
import java.util.HashMap;
import java.util.Map;

//支付宝网页支付
public class AliWebPayStrategy implements IPayStrategy {
 
    private String url;
    private String appId;
    private String privateKey;
    private String publicKey;
    private int orderTimeOut;

    public AliWebPayStrategy(String url, String appId, String privateKey, String publicKey, int orderTimeOUt) {
        this.url = url;
        this.appId = appId;
        this.privateKey = privateKey;
        this.publicKey = publicKey;
        this.orderTimeOut = orderTimeOUt;
    }
 
    @Override
    public Map<String, String> excutePayOrder(PayBaseModel payBaseModel) throws AlipayApiException {
        Map<String,String> resultMap = new HashMap<>();
        //创建API对应的request
        AlipayTradeWapPayRequest alipayRequest = new AlipayTradeWapPayRequest();
        //支付完成阿里回跳转到该页面
        alipayRequest.setReturnUrl(payBaseModel.getReturnUrl());
        //判断传入的地址是否为http或者https开头  支付结果通知接口
        alipayRequest.setNotifyUrl(payBaseModel.getNotifyUrl());
        JSONObject json = new JSONObject();
        json.put("out_trade_no", payBaseModel.getOutTradeNo());
        json.put("total_amount", new DecimalFormat("0.00").format(new BigDecimal(payBaseModel.getTradeMoney())));
        json.put("subject", payBaseModel.getTitle());
        json.put("passback_params", payBaseModel.getAttach());
        json.put("product_code", "QUICK_WAP_PAY");
        json.put("timeout_express",this.orderTimeOut+"");
        //填充业务参数
        alipayRequest.setBizContent(json.toJSONString());
        String form = "";
        AlipayClient alipayClient = new DefaultAlipayClient(this.url, this.appId,this.privateKey, 
                PayConstant.ALI_DEFAULT_FORMAT, PayConstant.DEFAULT_CHARSET, this.publicKey, PayConstant.ALI_DEFAULT_SIGNTYPE);
        //调用SDK生成表单
        form = alipayClient.pageExecute(alipayRequest).getBody();
        resultMap.put("form",form);
        return resultMap;
    }
}
```

### WxAppPayStrategy
```java
import com.github.wxpay.sdk.WXPayConstants;
import com.github.wxpay.sdk.WXPayUtil;
import com.tubitu.util.date.DateUtil;
import com.tubitu.util.http.ContentTypeEnum;
import com.tubitu.util.pay.IPayStrategy;
import com.tubitu.util.pay.PayUtil;
import com.tubitu.util.pay.model.PayBaseModel;
import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
 
import java.math.BigDecimal;
import java.util.Calendar;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

//微信APP支付
public class WxAppPayStrategy implements IPayStrategy {
    
    private String url;
    private String appId;
    private String mchId;
    private String appKey;
    private String partnerId;
    private int orderTimeOut;
 
    public WxAppPayStrategy(String url, String appId, String mchId, String appKey, String partnerId, int orderTimeOut) {
        this.url = url;
        this.appId = appId;
        this.mchId = mchId;
        this.appKey = appKey;
        this.partnerId = partnerId;
        this.orderTimeOut = orderTimeOut;
    }
 
    private final static Logger LOGGER = LoggerFactory.getLogger(WxAppPayStrategy.class);
 
    @Override
    public Map<String, String> excutePayOrder(PayBaseModel payBaseModel) {
        LOGGER.info("**********************获取微信APP支付模板" + payBaseModel.toString() + 
                "**************************************************");
        Map<String, String> resultMap = null;
       
            //返回结果
            resultMap = new HashMap<>();
        /*
        1 拼接参数向微信请求，获取prepare_id
         */
            Map<String, String> paramMap = new HashMap<>();
            paramMap.put("appid", this.appId);
            paramMap.put("body", payBaseModel.getTitle());
            paramMap.put("mch_id", this.mchId);
            //后面返回给app时的nonceStr需要与这里的一致
            String nonceStr = WXPayUtil.generateNonceStr();
            paramMap.put("nonce_str", nonceStr);
            paramMap.put("sign_type", WXPayConstants.MD5);
            if (!StringUtils.isEmpty(payBaseModel.getAttach())) {
                paramMap.put("attach", payBaseModel.getAttach());
            }
            paramMap.put("out_trade_no", payBaseModel.getOutTradeNo());
            //微信支付必须乘以100
            paramMap.put("total_fee", new BigDecimal(payBaseModel.getTradeMoney()).multiply(new BigDecimal("100")).intValue() + "");
            //获取用户真实ip
            paramMap.put("spbill_create_ip", payBaseModel.getRealIp());
            Date currDate = new Date();
            //订单生成时间
            paramMap.put("time_start", DateUtil.formatDate(currDate, "yyyyMMddHHmmss"));
            //订单失效时间
            paramMap.put("time_expire", DateUtil.formatDate(DateUtil.addDate(currDate, Calendar.MINUTE, this.orderTimeOut), 
                    "yyyyMMddHHmmss"));
            //异步回调
            paramMap.put("notify_url", payBaseModel.getNotifyUrl());
            //支付类型
            //app支付
            paramMap.put("trade_type", "APP");
        try {
            paramMap.put("sign", WXPayUtil.generateSignature(paramMap, this.appKey, WXPayConstants.SignType.MD5));
            LOGGER.info("**********************微信支付参数集合:" + paramMap + "**************************************************");
            String xmlString = WXPayUtil.mapToXml(paramMap);
            String entityString = PayUtil.post(this.url,xmlString,ContentTypeEnum.TextXml.getContentType());
            LOGGER.info("**********************微信支付统一下单接口返回数据:" + entityString + "**************************************************");
            Map<String, String> wxXmlMap = WXPayUtil.xmlToMap(entityString);
                /*
                2  此处组装返回给前端的参数集合
                */
            //组装返回给前端的数据
            //app支付   服务端返回App所需数据
            resultMap.put("appid", this.appId);
            resultMap.put("noncestr", nonceStr);
            resultMap.put("partnerid", this.partnerId);
            resultMap.put("prepayid", wxXmlMap.get("prepay_id"));
            //官方暂时规定为固定值
            resultMap.put("package", "Sign=WXPay");
            resultMap.put("timestamp", System.currentTimeMillis() / 1000 + "");
            //再次生成签名
            resultMap.put("sign", WXPayUtil.generateSignature(resultMap, this.appKey, WXPayConstants.SignType.MD5));
            resultMap.put("out_trade_no", payBaseModel.getOutTradeNo());
        } catch (Exception e) {
            LOGGER.error("\n********************请求微信统一下单接口异常********************",e);
        }
        return resultMap;
    }
}
```

### WxWebPayStrategy
由于微信小程序支付的请求逻辑和公众号内支付基本一致，唯一不同的就是appid和appSecret不同(appSecret用于公众号内和小程序内或者用户的openId，即使同一商家，公众号和小程序的appID和appSecret也不一致，而由此获得的同一个用户的openID也不一致），因此可以和公众号内使用相同的逻辑。
```java
import com.github.wxpay.sdk.WXPayConstants;
import com.github.wxpay.sdk.WXPayUtil;
import com.tubitu.util.date.DateUtil;
import com.tubitu.util.http.ContentTypeEnum;
import com.tubitu.util.pay.IPayStrategy;
import com.tubitu.util.pay.PayConstant;
import com.tubitu.util.pay.PayUtil;
import com.tubitu.util.pay.model.PayBaseModel;
import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
 
import java.math.BigDecimal;
import java.net.URLEncoder;
import java.util.Calendar;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

//微信公众号/网页支付
public class WxWebPayStrategy implements IPayStrategy {
 
    private String url;
    private String appId;
    private String mchId;
    private String appKey;
    private String openId;
    private int orderTimeOut;
 
    public WxWebPayStrategy(String url, String appId, String mchId, String appKey, String openId, int orderTimeOut) {
        this.url = url;
        this.appId = appId;
        this.mchId = mchId;
        this.appKey = appKey;
        this.openId = openId;
        this.orderTimeOut = orderTimeOut;
    }
 
    private final static Logger LOGGER = LoggerFactory.getLogger(WxWebPayStrategy.class);
 
 
    @Override
    public Map<String, String> excutePayOrder(PayBaseModel payBaseModel) {
        LOGGER.debug("\n********************微信网页支付参数\n" + payBaseModel.toString() + "\n********************");
        Map<String, String> resultMap = new HashMap<>();
        Map<String, String> preParamMap = new HashMap<>();
        String nonceStr = WXPayUtil.generateNonceStr();
        preParamMap.put("appid", this.appId);
        preParamMap.put("mch_id", this.mchId);
        preParamMap.put("nonce_str", nonceStr);
        preParamMap.put("body", payBaseModel.getTitle());
        preParamMap.put("out_trade_no", payBaseModel.getOutTradeNo());
        Date currDate = new Date();
        //订单生成时间
        preParamMap.put("time_start", DateUtil.formatDate(currDate, "yyyyMMddHHmmss"));
        //订单失效时间
        preParamMap.put("time_expire", DateUtil.formatDate(DateUtil.addDate(currDate, Calendar.MINUTE, this.orderTimeOut),
                "yyyyMMddHHmmss"));
        preParamMap.put("total_fee", new BigDecimal(payBaseModel.getTradeMoney()).multiply(new BigDecimal("100")).intValue() + "");
        preParamMap.put("spbill_create_ip", payBaseModel.getRealIp());
        LOGGER.debug("************************spbill_create_ip" + preParamMap.get("spbill_create_ip") + "************************************");
        preParamMap.put("notify_url", payBaseModel.getNotifyUrl());
        if (StringUtils.isNotEmpty(payBaseModel.getAttach())) {
            preParamMap.put("attach", payBaseModel.getAttach());
        }
        if (StringUtils.isNotEmpty(this.openId)) {
            //公众号支付 或者 小程序
            preParamMap.put("openid", this.openId);
            preParamMap.put("trade_type", "JSAPI");
        } else {
            //h5 页面支付
            preParamMap.put("trade_type", "MWEB");
            //需要注意的是H5 页面支付微信需要下面的场景信息，在最终页面提交支付请求的时候拿此处的referer里的数据与提交状态比对
            preParamMap.put("scene_info", "{\"h5_info\": {\"type\":\"Wap\",\"wap_url\": " + payBaseModel.getReferer() + "," + "\"wap_name\":" + " " + "\"兔比兔\"}}");
        }
        try {
            preParamMap.put("sign", WXPayUtil.generateSignature(preParamMap, PayConstant.WX_APP_KEY));
            String entityString = PayUtil.post(this.url, WXPayUtil.mapToXml(preParamMap), ContentTypeEnum.TextXml
                    .getContentType());
            Map<String, String> wxXmlMap = WXPayUtil.xmlToMap(entityString);
                    /*
                    2  此处组装返回给前端的参数集合
                    */
            LOGGER.debug("**********************微信支付统一下单接口返回数据:" + wxXmlMap +
                    "**************************************************");
            if (StringUtils.isEmpty(this.openId)) {
                String mwebUrl = wxXmlMap.get("mweb_url") + "&redirect_url=" + URLEncoder.encode(payBaseModel.getReturnUrl(),
                        PayConstant.DEFAULT_CHARSET);
                LOGGER.debug("*****************  mweb_url = " + mwebUrl + " ************************************************");
                wxXmlMap.put("mweb_url", mwebUrl);
                resultMap.putAll(wxXmlMap);
                resultMap.put("code", 100000 + "");
            } else {
                resultMap.put("appId", this.appId);
                resultMap.put("timeStamp", System.currentTimeMillis() / 1000 + "");
                resultMap.put("nonceStr", nonceStr);
                resultMap.put("package", "prepay_id=" + wxXmlMap.get("prepay_id"));
                resultMap.put("signType", WXPayConstants.MD5);
                resultMap.put("paySign", WXPayUtil.generateSignature(resultMap, this.appKey));
                resultMap.put("code", 100000 + "");
            }
        } catch (Exception e) {
            LOGGER.error("\n********************请求微信统一下单接口异常********************",e);
        }
        return resultMap;
    }
}
```

## Http请求
自己封装的用HttpClient发送Http请求的工具类：
```java
public static String post(String url, String stringEntity, String ContentType){
    String result = "";
    CloseableHttpClient httpClient = HttpClients.createDefault();
    CloseableHttpResponse response = null;
    try {
        HttpPost httpPost = new HttpPost(url);
        RequestConfig requestConfig = RequestConfig.custom().setSocketTimeout(8000).setConnectTimeout(6000).build();
        httpPost.setConfig(requestConfig);
        StringEntity entity = new StringEntity(stringEntity, PayConstant.DEFAULT_CHARSET);
        httpPost.addHeader("Content-Type", ContentType);
        httpPost.addHeader("User-Agent", "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1;SV1)");
        httpPost.setEntity(entity);
        response = httpClient.execute(httpPost);
        //读取响应体转化为字符串  建议：确定目标地址发出有限长度的响应时，使用该方法
        HttpEntity responseEntiry = response.getEntity();
        //不然使用流的方式好些
        /* if (entity != null) {
            InputStream inputStream = entity.getContent();

            BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(inputStream));
            String line = "";
            while ((line = bufferedReader.readLine()) != null) {
                System.out.println(line);

            }
        }*/
        result = EntityUtils.toString(responseEntiry, PayConstant.DEFAULT_CHARSET);
    } catch (IOException e) {
        LOGGER.error("\n******************http post error**********************",e);
    } finally {
        try {
            httpClient.close();
            response.close();
        } catch (IOException e) {
            LOGGER.error("\n********************close httpclient/response error********************",e);
        }
    }
    return result;
}
```

## 策略枚举
枚举暴露给客户端，以生成不同策略。
```java
public enum PayAlgorithm {
    //支付宝客户端
    ALI_APP,
    //支付宝页面端
    ALI_WEB,
    //微信客户端
    WX_APP,
    //微信页面端
    WX_WEB,
    //微信小程序
    WX_XCX;
}
```

## 策略工厂
工厂创建策略时，需要传入策略所需的一些配置参数，本文中商户只有一个，因此只有一个创建默认商户的支付策略。
```java
//支付工厂类，可以创建默认支付客户端，也可以创建自定义账户客户端
public class PayFactory {
    
    //创建默认支付客户端
    public static IPayStrategy createDefaultTubituStrategy(PayAlgorithm algorithm, String openId){
        IPayStrategy iPayStrategy = null;
        switch (algorithm){
            case ALI_APP:
                iPayStrategy = new AliAppPayStrategy(PayConstant.ALI_ORDER_URL,PayConstant.ALI_APP_ID,PayConstant
                        .ALI_PRIVATE_KEY,PayConstant.ALI_PUBLIC_KEY,30);
                break;
            case ALI_WEB:
                iPayStrategy = new AliWebPayStrategy(PayConstant.ALI_ORDER_URL,PayConstant.ALI_APP_ID,PayConstant
                        .ALI_PRIVATE_KEY,PayConstant.ALI_PUBLIC_KEY,30);
                break;
            case WX_APP:
                iPayStrategy = new WxAppPayStrategy(PayConstant.WX_UNIFIEDORDER_URL,PayConstant.WX_APP_ID,
                        PayConstant.WX_MCH_ID,PayConstant.WX_APP_KEY,PayConstant.WX_PARTNER_ID,30);
                break;
            case WX_WEB:
                iPayStrategy = new WxWebPayStrategy(PayConstant.WX_UNIFIEDORDER_URL,PayConstant.WX_H5_APPID,
                        PayConstant.WX_H5_MCHID,PayConstant.WX_APP_KEY,openId,30);
                break;
            case WX_XCX:
                iPayStrategy = new WxWebPayStrategy(PayConstant.WX_UNIFIEDORDER_URL,PayConstant.WX_XCX_APPID,
                        PayConstant.WX_H5_MCHID,PayConstant.WX_APP_KEY,openId,30);
                break;
        }
        return iPayStrategy;
    }
    
}
```

## 场景调用
首先通过工厂创建指定策略，然后执行接口里的方法，Java能够动态绑定对应实现类。
以下代码适用于在controller里调用策略返回支付参数。
```
IPayStrategy strategy = PayFactory.createDefaultTubituStrategy(PayAlgorithm.WX_WEB,"");
PayBaseModel payBaseModel = new PayBaseModel(商品描述,回调接口,交易金额,商户订单编号,支付完跳转页面);
Map<String,String> resultMap = strategy.excutePayOrder(payBaseModel);
```
需要注意的是，小程序支付和公众号内支付需要首先获取openId，所以中间需要一系列重定向和参数。
![](pay/2.jpg)

由于设置`redirect_url`后,回跳指定页面的操作可能发生在：
1. 微信支付中间页调起微信收银台后超过5秒
2. 用户点击“取消支付“或支付完成后点“完成”按钮。因此无法保证页面回跳时，支付流程已结束，所以商户设置的redirect_url地址不能自动执行查单操作，应让用户去点击按钮触发查单操作。

# 前端代码
## 公众号内支付
### 第一种方式
```js
function onBridgeReady(){
   WeixinJSBridge.invoke(
      'getBrandWCPayRequest', {
         "appId":"wx2421b1c4370ec43b",     //公众号名称，由商户传入     
         "timeStamp":"1395712654",         //时间戳，自1970年以来的秒数     
         "nonceStr":"e61463f8efa94090b1f366cccfbbb444", //随机串     
         "package":"prepay_id=u802345jgfjsdfgsdg888",     
         "signType":"MD5",         //微信签名方式：     
         "paySign":"70EA570631E4BB79628FBCA90534C63FF7FADD89" //微信签名 
      },
      function(res){
      if(res.err_msg == "get_brand_wcpay_request:ok" ){
      // 使用以上方式判断前端返回,微信团队郑重提示：
            //res.err_msg将在用户支付成功后返回ok，但并不保证它绝对可靠。
            //此处可看作支付可能成功，跳转到成功页面
            window.location.replace("index.html");
      } 
   }); 
}
if (typeof WeixinJSBridge == "undefined"){
   if( document.addEventListener ){
       document.addEventListener('WeixinJSBridgeReady', onBridgeReady, false);
   }else if (document.attachEvent){
       document.attachEvent('WeixinJSBridgeReady', onBridgeReady); 
       document.attachEvent('onWeixinJSBridgeReady', onBridgeReady);
   }
}else{
   onBridgeReady();
}
```

### 第二种方式
```javascript
wx.config({
    debug: true, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
    appId: '', // 必填，企业号的唯一标识，此处填写企业号corpid
    timestamp: , // 必填，生成签名的时间戳
    nonceStr: '', // 必填，生成签名的随机串
    signature: '',// 必填，签名，见附录1
    jsApiList: [] // 必填，需要使用的JS接口列表，所有JS接口列表见附录2
});
wx.ready(function(){
    // config信息验证后会执行ready方法，所有接口调用都必须在config接口获得结果之后，config是一个客户端的异步操作，所以如果需要在页面加载时就调用相关接口，则须把相关接口放在ready函数中调用来确保正确执行。对于用户触发时才调用的接口，则可以直接调用，不需要放在ready函数中。
});
wx.chooseWXPay({
    appId: 'wx23841cce7185b550',
    timestamp: '1461300911', // 支付签名时间戳，注意微信jssdk中的所有使用timestamp字段均为小写。但最新版的支付后台生成签名使用的timeStamp字段名需大写其中的S字符
    nonceStr: '5719aeafb587f', // 支付签名随机串，不长于 32 位
    package: 'prepay_id=wx20160422125512b7d2205c9c0913643939', // 统一支付接口返回的prepay_id参数值，提交格式如：prepay_id=***）
    signType: 'MD5', // 签名方式，默认为'SHA1'，使用新版支付需传入'MD5'
    paySign: '5DAB1DDABE1AD34E8FF3386AE971B727', // 支付签名
    success: function(res) {
    // 支付成功后的回调函数
    if (res.errMsg == "chooseWXPay:ok") {
    //支付成功
    alert('支付成功');
    else {
    alert(res.errMsg);
    }
        },  
    cancel: function(res) {
    //支付取消
    alert('支付取消');
    }
        }); 
```

这里可以看到公众号内支付实际上有两种方式，第一种是jsapi的模式，微信浏览器内部有jsapi直接可以调用，第二种为js-sdk的方式，但是两种方式都需要引入微信的js。
http：//res.wx.qq.com/js/jwexin-1.2.0.js

第二种方式可以看到多了一个signature的签名参数，大概提一下如何获取：
1. 获取AccessToken
2. 获取jsapi_ticket
3. 生成signature

```java

//获取微信token
public static String getAccessToken(){
    //获取token
    String tokenParam = "grant_type=client_credential&appid=" + WxPayConfig.APPID + "&secret="
            + WxPayConfig.APPSECRET;
    //这一步的发送请求工具类请自己实现
    String tokenJsonStr = HttpUtil.sendGET("https://api.weixin.qq.com/cgi-bin/token", tokenParam);
    Map<String,String> tokenMap = JSONObject.fromObject(tokenJsonStr);
    System.out.println(tokenMap);
    // 获取access_token
    String access_token = (String) tokenMap.get("access_token");
    return access_token;
}
//获取jsapi_ticket
public static String getJsApiTicket(String accessToken){
    String ticketParam = "access_token=" + accessToken + "&type=jsapi";
    String ticketJsonStr = HttpUtil.sendGET("https://api.weixin.qq.com/cgi-bin/ticket/getticket", ticketParam);
    Map<String,String> ticketMap = JSONObject.fromObject(ticketJsonStr);
    // 获取jsapi_ticket
    String ticket = (String) ticketMap.get("ticket");
    return ticket;
}

//生成signature
String url = PayConstant.PAY_DOMAIN;//微信授权的网址 这个要看微信公众号设置
String signValue = "jsapi_ticket=" + ticket + "&noncestr=" + noncestr + "×tamp=" + timestamp
            + "&url=" + url;
String signature = Sha1Util.getSha1((signValue))

public static String getSha1(String str) {
    if (str == null || str.length() == 0) {
	    return null;
    }
	char hexDigits[] = {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9','a', 'b', 'c', 'd', 'e', 'f'};
	try {
		MessageDigest mdTemp = MessageDigest.getInstance("SHA1");
		mdTemp.update(str.getBytes("GBK"));
		byte[] md = mdTemp.digest();
		int j = md.length;
		char buf[] = new char[j * 2];
		int k = 0;
		for (int i = 0; i < j; i++) {
			byte byte0 = md[i];
			buf[k++] = hexDigits[byte0 >>> 4 & 0xf];
			buf[k++] = hexDigits[byte0 & 0xf];
		}
		return new String(buf);
	} catch (Exception e) {
		return null;
	}
}
```

# 回调函数
```java
//处理微信回调请求
public static Map<String, String> resolveWXPayResponse(HttpServletRequest request, String appkey) throws Exception {
    LOGGER.info("***************************************处理微信回调request域********************************************");
    ByteArrayOutputStream outSteam = new ByteArrayOutputStream();
    InputStream inStream = request.getInputStream();
    byte[] buffer = new byte[1024];
    int len = 0;
    while ((len = inStream.read(buffer)) != -1) {
        outSteam.write(buffer, 0, len);
    }
    outSteam.close();
    inStream.close();
    String result = new String(outSteam.toByteArray(), PayConstant.DEFAULT_CHARSET);
    LOGGER.info("***************************************微信返回结果" + result + "********************************************");
    Map<String, String> resultMap = new HashMap<>();
    if (!StringUtils.isEmpty(result)) {
        Map<String, String> map = WXPayUtil.xmlToMap(result);
        //验签
        if (WXPayUtil.isSignatureValid(map, appkey, WXPayConstants.SignType.MD5)) {
            String attach = map.get("attach");
            LOGGER.info("***************************************微信自定义参数" + attach + "********************************************");
            LOGGER.info("***************************************商户订单号    ：" + map.get("out_trade_no") + "********************************************");
            resultMap = map;
        } else {
            resultMap.put("return_code", "FAIL");
            resultMap.put("return_msg", "验签失败");
        }
    }
    return resultMap;
}

//处理支付宝回调请求
public static Map<String, String> resolveAliPayResponse(HttpServletRequest request) throws UnsupportedEncodingException, AlipayApiException {
    Map<String, String> result = new HashMap();
    LOGGER.info("***************************************处理支付宝回调request域********************************************");
    //1.从支付宝回调的request域中取值
    Map<String, String[]> requestParams = request.getParameterMap();
    for (Iterator<String> iter = requestParams.keySet().iterator(); iter.hasNext(); ) {
        String name = iter.next();
        String[] values = requestParams.get(name);
        String valueStr = "";
        for (int i = 0; i < values.length; i++) {
            valueStr = (i == values.length - 1) ? valueStr + values[i] : valueStr + values[i] + ",";
        }
        result.put(name, valueStr);
    }
    //2.签名验证(对支付宝返回的数据验证，确定是支付宝返回的)
    boolean signVerified;
    //2.1调用SDK验证签名
    signVerified = AlipaySignature.rsaCheckV1(result, PayConstant.ALI_PUBLIC_KEY, PayConstant.DEFAULT_CHARSET, PayConstant.ALI_DEFAULT_SIGNTYPE);
    if (signVerified) {
        return result;
    }
    return null;
}
```
由于设置`redirect_url`后，回跳指定页面的操作可能发生在：
1. 微信支付中间页调起微信收银台后超过5秒
2. 用户点击“取消支付“或支付完成后点“完成”按钮。因此无法保证页面回跳时，支付流程已结束，所以商户设置的redirect_url地址不能自动执行查单操作，应让用户去点击按钮触发查单操作

# 微信公众号支付
微信网页支付时序图：
![](pay/3.png)

需要准备：
- [微信公众平台已认证的服务号](https://mp.weixin.qq.com/)，并且需要开通微信支付该能（必须是企业才有资格申请，找产品或运营去申请）
- [微信商户平台账号](https://pay.weixin.qq.com)
- 一个正式的应用服务器，并且注册域名，微信支付涉及的私密数据比较多，不允许使用natapp，花生壳之类的内网穿透工具实现，需要有正式的服务器环境，并且要注册域名，不能使用IP。比如：`http://www.wjy.com`
- 相关配置
	+ 配置支付授权目录，登录商户平台->产品中心—>开发配置->支付配置
	在项目根路径下，以及web目录下的页面都有支付权限，如果不在该路径的页面，则无法调用支付功能。
	若页面地址为：`http://mywexx.xxxx.com/web/pay.html`
	则需要配置为：`http://mywexx.xxxx.com/web/`
	+ 设置API密钥，登录商户平台——>账户中心——>API安全——>API密钥
	+ 配置JS接口安全域名与网页授权域名，登录公众平台——>公众号设置——>功能设置
	配置网页授权域名：主要用于获取用户的openId，需要识别这是哪个人，这个域名必须和`获取用户授权`时的接口一致。
	然后按照提示，将文件下载并上传到`网页授权域名`目录下，并且在浏览器，测试访问，`http://www.baidu.com/你的文件名.txt`看到一串字符串，证明成功。(可以通过nginx访问该文件)
	若对openID不了解的同学可先参考微信公众号开发文档：https://mp.weixin.qq.com/wiki
	配置JS接口安全域名：要让我们的页面中弹出输入密码的窗口，需要使用微信提供的JS-SDK工具，如果不配置JS接口安全域名，你的页面无法使用JS-SDK。
	+ 设置IP白名单：登录公众平台->基本配置->IP白名单，然后根据提示添加，这里同时添加两个，
	一个是本机的外网ip(通过百度搜索`IP`即可查看)，另外一个是服务器的ip地址。
	必须设置IP白名单，因为在IP白名单内的IP来源，获取access_token接口才可调用成功。

公众号开通微信支付，是在微信商户平台完成的，绑定完成后，可以在公众号看到绑定的商户号。[这是绑定教程](https://pay.weixin.qq.com/static/pay_setting/appid_protocol.shtml)
可以先找商户平台所有者(超级管理员)，把自己的微信绑定为管理员：顶部点击账号中心->左侧找到员工账号管理->右侧找到管理员，点击新增账号->然后可能需要安装安全控件。
安装控件之后，可能需要重启浏览器，才能检测到安全控件，所以需要重新登录，然后发送验证，就成功在本机上安装安全控件了。安装好控件之后，就可以将自己的微信号绑定为管理员了。
注意提交的时候，需要输入`操作密码`，该密码不是商户平台的登录密码，是操作密码。
根据[绑定教程](https://pay.weixin.qq.com/static/pay_setting/appid_protocol.shtml)，输入公众号的appID后，可以在绑定的公众号->商户号管理，看到待关联商户号，可以确认以及后续步骤，即可成功关联。

商户的API密钥：商户平台账户中心->API安全->往下拉找到`API密钥`->如果没有设置过，可以重置，不过这个需要自己设置，我随便找了个32密码生成填进去了。

登录微信公众平台，拉到页面最下边，在左侧找到开发一栏，点击基本配置，需要点击确认申请开通，然后就可以看到AppID和其他配置信息。
AppSecret需要向该公众号的管理员申请开通，30分钟内通过即可，然后输入该公众号密码，即可查看到AppSecret，按照页面提示存储好AppSecret，并进行后续操作。

此处是账号模板,请参考：
```
微信公众平台：账户：con*******om 登录密码 ******
公众APPID：wx15*********a8
APPSECEPT : c210***************892d7
微信商户平台：账户：149**********6742 登录密码：******
商户ID：14******42
API密钥：5d5************b35b
```
至此，需要的四大参数都找到了。可以先[测试一下](https://mp.weixin.qq.com/debug/)我们的配置是否有问题，如果返回`Request successful`，以及`access_token`和`expires_in`，说明配置没问题。

我们直接把微信官方提供的一堆util，放到项目中，懒得打jar包了。

## 获取用户授权
在确保微信公众账号拥有授权作用域（scope参数）的权限的前提下（服务号获得高级接口后，默认拥有scope参数中的snsapi_base和snsapi_userinfo），引导关注者打开如下页面：
```
https://open.weixin.qq.com/connect/oauth2/authorize
?appid=APPID
&redirect_uri=REDIRECT_URI
&response_type=code
&scope=SCOPE
&state=STATE#wechat_redirect
```
参数解析：
appid：公众号的唯一标识
redirect_uri：授权后重定向的回调链接地址，这个地址和前端要，请使用urlEncode对链接进行处理，注意这个回调地址不能带端口，只能是80，也就是说只能是域名
response_type：返回类型，请填写code，写死
scope：应用授权作用域，snsapi_base （不弹出授权页面，直接跳转，只能获取用户openid），snsapi_userinfo （弹出授权页面，可通过openid拿到昵称、性别、所在地。并且，即使在未关注的情况下，只要用户授权，也能获取其信息）
state：重定向后会带上state参数，开发者可以填写a-zA-Z0-9的参数值，最多128字节
`#wechat_redirect`：无论直接打开还是做页面302重定向时候，必须带此参数

若提示“该链接无法访问”，请检查参数是否填写错误，是否拥有scope参数对应的授权作用域权限。
尤其注意，由于授权操作安全等级较高，所以在发起授权请求时，微信会对授权链接做正则强匹配校验，如果链接的参数顺序不对，授权页面将无法正常访问。

这一步其实就是引导用户打开授权页面，如果用户同意授权，页面将跳转至`redirect_uri/?code=CODE&state=STATE`。
code说明：code作为换取access_token的票据，每次用户授权带上的code将不一样，code只能使用一次，5分钟未被使用自动过期。
由于公众号的secret和获取到的access_token安全级别都非常高，必须只保存在服务器，不允许传给客户端。后续刷新access_token、通过access_token获取用户信息等步骤，也必须从服务器发起。
既然必须从服务器发起，所以后台接口如下：
```java
/**
 * @author: wjy
 * @description: 微信公众号支付入口：指引用户授权，获取code
 */
@GetMapping("wxAuthorization")
public ModelAndView weiXinPublicPay() {
	LOGGER.debug("*************获取code***********************************************");
	ModelAndView mv = new ModelAndView();
	String rediretUrl = PayUtil.getProjectUrl(this.request) + "zjx/api/pay/perPay";
	try {
		String url = "https://open.weixin.qq.com/connect/oauth2/authorize?"
				+ "appid=" + PayConstant.WX_H5_APPID
				+ "&redirect_uri=" + URLEncoder.encode(rediretUrl, PayConstant.DEFAULT_CHARSET)
				+ "&response_type=code&scope=snsapi_base&state=123#wechat_redirect";
				//这里必须是重定向
		mv.setViewName("redirect:" + url);
	} catch (UnsupportedEncodingException e) {
		e.printStackTrace();
	}
	LOGGER.debug("**********重定向到微信" + mv.getViewName() + "******************************************");
	return mv;
}
```
注意上边的`redirect_uri`，可以是前台地址也可以是后台接口，反正微信会把code拼接到`redirect_uri`的地址栏后边。
若是前台地址，让前端在地址栏获取`code`参数，然后再次调用`perPay`预支付接口，并传递`code`参数，后台通过`@requestParam`获取前端传递的code参数。
若是后台地址，我们直接在接口上用`@requestParam`获取code参数，然后通过code换取`access_token`和`openid`。
这两地方法的区别是，`redirect_uri`填写前台地址，会多跳一次页面，然后前台再把code传回来，`redirect_uri`填写后台地址，就不用前端传递code参数，
过程是：
- 授权接口重定向到微信
- 微信重定向你指定的`redirect_uri`，并在地址栏带上code
- 前端通过code请求预支付接口
code作为换取access_token的票据，每次用户授权带上的code将不一样，code只能使用一次，5分钟未被使用自动过期，所以不能缓存。

预支付接口只需要用户的openid，一个用户对于一个公众号的openid是不会变的，所以没必要非要在请求预支付的时候才通过code换取openid，可以在进入公众号就获取授权，然后获取openid，
然后让前端缓存在本地，或是通过接口存到数据库中，这样在请求预支付接口时，直接传递存储好的openid就好了。

调用预支付接口代码如下：
```java
/**
     * @Description 公众号支付(JSAPI)
     * @return Map
     */
    @ResponseBody
    @GetMapping("perPay")
    public Map perPay(@RequestParam("openid") String openid) {
        LOGGER.info(">>>>>>>>>>>>>>>>>>>openid是 " + openid +">>>>>>>>>>>>>>>>>>>>>>>>>>");
        Map<String, String> resultMap = new HashMap<>();
        try {

            //生成订单编号
            int number = (int)((Math.random()*9)*1000);//随机数
            DateFormat dateFormat = new SimpleDateFormat("yyyyMMddHHmmss");//时间
            String orderNo = dateFormat.format(new Date()) + number;
            //拼接统一下单地址参数
            Map<String, String> paraMap = new HashMap<>();
            paraMap.put("appid", PayConstant.WX_H5_APPID);
            paraMap.put("mch_id", PayConstant.WX_H5_MCHID);
            paraMap.put("body", "最美课本支付测试");
            paraMap.put("nonce_str",WXPayUtil.generateNonceStr());
            paraMap.put("openid", openid);
            paraMap.put("out_trade_no", orderNo);
            paraMap.put("spbill_create_ip",HttpUtil.getIpAddr(this.request));
            paraMap.put("total_fee",new BigDecimal("0.01").multiply(new BigDecimal("100")).intValue() + "");
            paraMap.put("trade_type", "JSAPI");
            paraMap.put("notify_url", PayUtil.getProjectUrl(this.request) + "zjx/api/wxNotify");
            paraMap.put("sign", WXPayUtil.generateSignature(paraMap, PayConstant.WX_H5_APP_KEY));//和下面sign方法一致，不是说参数一致。
            //将所有参数(map)转xml格式
            String xml = WXPayUtil.mapToXml(paraMap);
            LOGGER.info("*******************请求微信参数"+xml);
            //发送post请求"统一下单接口"返回预支付id：prepay_id
            String wxReturnStr = PayUtil.post(PayConstant.WX_UNIFIEDORDER_URL, xml, ContentTypeEnum.TextXml.getContentType());

            Map<String, String> wxXmlMap = WXPayUtil.xmlToMap(wxReturnStr);
            LOGGER.info("**********************微信统一下单接口返回数据:" + wxXmlMap + "*****************************************");
            //组装返回给前端的数据
            resultMap.put("appId", PayConstant.WX_H5_APPID);
            resultMap.put("timeStamp", WXPayUtil.getCurrentTimestamp()+"");
            resultMap.put("nonceStr", WXPayUtil.generateNonceStr());
            resultMap.put("signType", "MD5");
            resultMap.put("package", "prepay_id=" + wxXmlMap.get("prepay_id"));
			//注意这里的签名方法需要和上边paramMap中sign的方法一致，app_key也必须一致
            resultMap.put("paySign", WXPayUtil.generateSignature(resultMap, PayConstant.WX_H5_APP_KEY));
            LOGGER.info("**********************返回给前端的数据:" + resultMap + "*****************************************");
        } catch (Exception e) {
            e.printStackTrace();
        }
        return resultMap;
    }
```

支付成功的回调：
```java
@RequestMapping("wxBuySpaceNotify")
public void wxBuySpaceNotify() {
	try {
		Map<String, String> map = PayUtil.resolveWXPayResponse(this.request, PayConstant.WX_H5_APP_KEY);
		LOGGER.info("****************微信支付回调参数:" + JSONObject.toJSONString(map) + "***************************");
		if (map.size() >= 2) {
			//执行回调逻辑
			map = payService.buySpaceCallback(map);
			//内部逻辑是否执行成功
			int resultCode = Integer.valueOf(map.get("dealSuccess"));
			map.clear();
			if (resultCode == 1) {
				//处理成功
				map.put("return_code", "SUCCESS");
				LOGGER.info("\n********************微信支付回调成功********************");
			} else {
				map.put("return_code", "FAIL");
				map.put("return_msg", "业务处理失败");
			}
			response.getWriter().write(WXPayUtil.mapToXml(map));
			response.getWriter().flush();
			response.getWriter().close();
		}
	} catch (Exception e) {
		//告诉微信业务失败
		try {
			Map<String, String> map = new HashMap<>();
			map.put("return_code", "FAIL");
			map.put("return_msg", "业务处理失败");
			response.getWriter().write(WXPayUtil.mapToXml(map));
			response.getWriter().flush();
			response.getWriter().close();
		} catch (Exception e1) {
			LOGGER.info("\n********************微信支付回调响应异常********************", e);
		}
		LOGGER.info("\n********************微信支付回调异常********************", e);
	}
}
```

关键就这两个方法，一个预支付，微信返回perpay_id，你返回给前端，前端通过perpay_id调起支付组件，若完成支付，微信会回调你设置的notify_url地址，这就是你支付成功的回调地址。
你要在回调方法里写一些业务相关的逻辑，比如：更改订单状态为已支付，以及赠送会员等后续操作。

[公众号支付参考](https://www.jianshu.com/p/9c322b1a5274)
[参考2](https://blog.csdn.net/javaYouCome/article/details/79473743)