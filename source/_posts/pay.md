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