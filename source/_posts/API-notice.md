---
title: API-notice
tags: 开发规范
date: 2019-04-03 09:54:28
---

# Controller
1. controller只负责收发参数、跳转页面等，不要在controller里写业务逻辑，业务逻辑全部写在接口实现类(serviceImpl)中。
2. 入参使用`@RequestBody Map<String,String> map`，返回数据使用`@ResponseBody`
3. 使用LOGGER.info()打印入参，方便线上调试。
4. 使用ResponseResultUtil返回数据、code码和message。
    通过code码和message来向前端反馈信息，例如：10000代表成功，message为“请求成功”，20000代表失败，message为“请求失败”，其他自行定义。
5. 使用断言验证参数：`Assert.isTrue()`
6. 大部分方法都需要token，增加了接口的安全性，即必须登陆后，才可以请求数据，token需要前端通过header传递。

示例：
```
    @ResponseBody
    @PostMapping(value = "getUserInfo")
    public ResponseResultUtil getUserInfo(@RequestBody Map<String,String> paramMap){
        LOGGER.info("\n***************\n start getUserinfo\n 获取个人信息 \n paramMap " + paramMap + "\n***************");
        ResponseResultUtil result = ResponseResultUtil.responseSuccess();
        try {
            Token token = TokenUtil.getToken(request.getHeader("token"));
            Assert.isTrue(token != null, "token解密失败");

            UserInfo searchUserInfo = new UserInfo();
            searchUserInfo.setUserId(token.getId());
            searchUserInfo = userInfoService.getUserInfo(searchUserInfo);
            result.setMessage("请求成功");
            result.setData(userInfo);
        }catch (Exception e){
            e.printStackTrace();
            result.setCode(code);
            result.setMessage(e.getMessage());
        }
        LOGGER.info("\n***************\n end getUserinfo\n 返回个人信息 \n result " + result.getData() + "\n***************");
        return result;
    }
```
还有另一种打印日志的样式：
```
LOGGER.info(
"\n***************************************" + "\n" +
        "start getUserInfo" + "\n" +
        "获取个人信息" + "\n" +
        "paramMap = " + paramMap + "\n" +
        "\n********************************************"
);
```
```
LOGGER.info(
"\n***************************************" + "\n" +
        "end getUserInfo" + "\n" +
        "返回个人信息" + "\n" +
        "result = " + result.getData() + "\n" +
        "\n********************************************"
);
```
这种样式在填写数据时，比较好分辨，每行都代表一些信息。缺点是占用的篇幅太多。

运行异常时，可以使用logger打印：
`LOGGER.error("\n********************运行异常********************", e);`
这种方式和`e.printStackTrace();`有什么区别？

需要的LOGGER对象需要在类头部定义：
`private final static Logger LOGGER = LoggerFactory.getLogger(UserInfoController.class);`

需要的request、response、session需要在类头部定义：
```
protected HttpServletRequest request;
protected HttpServletResponse response;
protected HttpSession session;

//初始化
@ModelAttribute
    public void init(HttpServletRequest request, HttpServletResponse response) {
        this.request = request;
        this.response = response;
        this.session = request.getSession();
    }
```


# service
```
ResponseResultUtil explodeOrderList(Map<String, String> paramMap,ResponseResultUtil responseResultUtil,Integer userId);
```

# serviceImpl
```
@Override
    public ResponseResultUtil explodeOrderList(Map<String, String> paramMap,ResponseResultUtil result,Integer userId) {
        Integer pageNum = StringUtils.isEmpty(paramMap.get("pageNum")) ? 1 : Integer.parseInt(paramMap.get("pageNum"));
        Integer pageSize = StringUtils.isEmpty(paramMap.get("pageSize")) ? 1 : Integer.parseInt(paramMap.get("pageSize"));
        PageHelper.startPage(pageNum,pageSize);
        paramMap.put("userId",userId.toString());
        Map<String,Object> resultMap = new HashMap<>();
        Map<String,Object> map = new HashMap<>();
        map.put("userId",userId.toString());
        List<ExplodeOrderDto> explodeList = goodsOrderMapper.explodeOrderList(paramMap);
        //微信群列表
        List<StoreWxGroupDto> wxList = storeWxGroupMapper.selectStoreWxGroupUserSelective(map);
        resultMap.put("explodeList",explodeList);
        resultMap.put("wxList",wxList);
        result.setData(resultMap);
        return result;
    }
```

# mapper
```
List<ExplodeOrderDto> explodeOrderList(Map<String,String> map);
```

# xml
```
<!--运营合伙人-爆品订单列表-->
  <select id="explodeOrderList" resultMap="ExplodeOrderMap" parameterType="map">
    select
    <include refid="explodeOrderSql"/>
    from goods_order a
    left join store_wx_group b on a.store_wx_group_id = b.id
    left join goods_order_detail c on c.order_id = a.id
    <where>
      b.user_id = #{userId}
      <if test="id != null">
        and a.id = #{id}
      </if>
      <if test="status != null">
        and a.status = #{status}
      </if>
      <if test="startTime != null">
        and a.create_time >= #{startTime}
      </if>
      <if test="endTime != null">
        and a.create_time <![CDATA[<=]]> #{endTime}
      </if>
      <if test="groupId != null">
        and b.id = #{groupId}
      </if>
    </where>
  </select>
```

```
<sql id="explodeOrderSql">
    a.id,
    a.order_no,
    a.bp_id,
    a.total_price,
    a.order_price,
    a.fgt_price,
    a.consignee,
    a.tel,
    a.area,
    a.address,
    a.create_time,
    a.receive_time,
    a.status,
    a.user_id,
    a.pay_method,
    a.postage_type,
    a.currency_type,
    a.tracking_no,
    a.express_name,
    a.out_trade_no,
    a.parent_order_no,
    a.store_wx_group_id,
    b.id as groupId,
    b.group_name,
    c.id as detailId,
    c.goods_name,
    c.goods_img,
    c.goods_price,
    c.goods_num
  </sql>
```

# ResponseResultUtil
```
public class ResponseResultUtil<T> implements Serializable {
    private int code;
    private String message;
    private T data;

    public ResponseResultUtil(T data) {
        this.code = ErrorEnum.ERROR_100000.getCode();
        this.message = ErrorEnum.ERROR_100000.getMessage();
        this.data = data;
    }

    public ResponseResultUtil(ErrorEnum error) {
        this.code = error.getCode();
        this.message = error.getMessage();
    }

    public ResponseResultUtil(ErrorEnum error, T data) {
        this.code = error.getCode();
        this.message = error.getMessage();
        this.data = data;
    }

    public static ResponseResultUtil responseSuccess() {
        return new ResponseResultUtil(ErrorEnum.ERROR_100000);
    }

    public static ResponseResultUtil responseFail() {
        return new ResponseResultUtil(ErrorEnum.ERROR_200000);
    }

    public int getCode() {
        return code;
    }

    public void setCode(int code) {
        this.code = code;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }
}
```
一般搭配ErrorEnum错误枚举类使用，错误码可自行定义：
```
/**
* 标准错误码枚举
* 格式 ERROR_ABBCCC
* A、B、C代表数字
* A为1 成功   A为2 失败
* BB 代表错误板块  暂定 00为通用  01为管理后台  02为用户板块   03为家厨板块  04为配送板块
* CCC 代表具体唯一错误码
* message作为前台显示
* desc作为后台日志打印跟踪 如果有参数 用MessageFormat.format(desc,arg1,arg2……)的方式注入
*/
public enum ErrorEnum {

    //请求成功
    ERROR_100000(100000, "请求成功"),

    //请求失败
    ERROR_200000(200000, "请求失败"),

    //通用错误码
    ERROR_200001(200001, "全局异常"),
    ERROR_200002(200002, "全局错误"),
    //商城错误码
    ERROR_200020(200020, "库存不足"),
    //管理后台板块 错误码
    ERROR_201001(201001, "您的权限不足，请与管理员联系", "后台管理用户{0}执行{1}操作失败"),

    ERROR_200100(200100, "调用api未传token。请先调用登陆接口获取token，然后在http header加入参数token，再调用api"),
    ERROR_200200(200200, "用户未登陆或者登陆已超时"),
    ERROR_200300(200300, "token不正确"),
    ERROR_200400(200400, "已在另一设备登录，请重新登录");


    private int code;
    private String message;
    private String desc;

    private ErrorEnum(int code, String message) {
        this.code = code;
        this.message = message;
        this.desc = message;
    }

    private ErrorEnum(int code, String message, String desc) {
        this.code = code;
        this.message = message;
        this.desc = desc;
    }

    public int getCode() {
        return code;
    }

    public String getMessage() {
        return message;
    }

    public String getDesc() {
        if (desc == null) {
            desc = message;
        }
        return code + ":" + desc;
    }

    public void setDesc(String desc) {
        this.desc = desc;
    }
}
```
