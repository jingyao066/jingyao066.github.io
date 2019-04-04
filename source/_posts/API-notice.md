---
title: API-notice
tags: 开发规范
date: 2019-04-03 09:54:28
---

# Controller
- controller只负责收发参数、跳转页面等，不要在controller里写业务逻辑，业务逻辑全部写在接口实现类(serviceImpl)中。
- 入参使用`@RequestBody Map<String,String> map`，返回数据使用`@ResponseBody`
- 规范的使用restful
- 使用LOGGER.info()打印入参，方便线上调试。
- 使用ResponseResultUtil返回数据、code码和message。
    通过code码和message来向前端反馈信息，例如：10000代表成功，message为“请求成功”，20000代表失败，message为“请求失败”，其他自行定义。
- 使用断言验证参数：`Assert.isTrue()`
- 大部分方法都需要token，增加了接口的安全性，即必须登陆后，才可以请求数据，token需要前端通过header传递。

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

# 典型接口示例
## 分页接口
- controller
```
@PostMapping("/getGoodsList")
public ResponseResultUtil getGoodsList(@RequestBody Map<String, Object> paramMap) {
    LOGGER.info(
            "\n***************************************" + "\n" +
                    "start getGoodsList" + "\n" +
                    "商品列表" + "\n" +
                    "paramMap = " + map + "\n" +
                    "\n********************************************"
    );
    ResponseResultUtil result = ResponseResultUtil.responseSuccess();
    try {
        Token token = TokenUtil.getToken(request.getHeader("token"));
        Assert.isTrue(token != null, "token不能为空!");
        map.put("userId", String.valueOf(token.getId()));
        result = goodsService.getGoodsList(map);
    } catch (Exception e) {
        e.printStackTrace();
        result = ResponseResultUtil.responseFail();
        result.setMessage(e.getMessage());
    }
    LOGGER.info(
            "\n***************************************" + "\n" +
                    "end getGoodsList" + "\n" +
                    "商品列表" + "\n" +
                    " resultMap " + result.getData() + "\n" +
                    "\n********************************************"
    );
    return result;
}
```
- service
`List<Goods> getGoodsList(Map<String, Object> map);`
或
`List<Map<String, Object>> getGoodsList(Map<String, Object> map);`

- serviceImpl
```
public List<Goods> getGoodsList(Map<String, Object> map) {
    PageHelper.startPage(
            map.get("pageNum") == null ? 1 : Integer.parseInt(map.get("pageNum")),
            map.get("pageSize") == null ? 10 : Integer.parseInt(map.get("pageSize")));
    List<Goods> list = goodsMapper.getGoodsList(map);
    return list;
}
```

- mapper
`List<Goods> getGoodsList(Map<String,Object> map);`
- mapper.xml
```
<select id="getGoodsList" resultMap="BaseResultMap" parameterType="map">
  select
  goods_name,
  cover_img,
  convert(price * #{exchangeRate},decimal(10,2)) price,
  sales_volume,
  id
  FROM
    goods
  <where>
    <if test="isNew != null">
      and is_new = 1
    </if>
    and status = 1
  </where>
  and del_flag =  0
  and audit_status = 1
</select>
```
## 注册接口
- controller
```
@ResponseBody
@PostMapping(value = "/register")
public ResponseResultUtil register(@RequestBody Map<String,String> paramMap){
    LOGGER.info(
            "\n***************************************" + "\n" +
                    "start register" + "\n" +
                    "注册" + "\n" +
                    "paramMap = " + paramMap + "\n" +
                    "\n********************************************"
    );
    ResponseResultUtil result = ResponseResultUtil.responseSuccess();
    int code = ErrorEnum.ERROR_200000.getCode();
    try {
        String regex = "^(?![0-9]+$)(?![a-zA-Z]+$)[0-9A-Za-z]{6,10}$";
        code = 200001;
        Assert.isTrue(StringUtils.isNotEmpty(paramMap.get("mobile")), "手机号码不能为空");

        code = 200002;
        Assert.isTrue(paramMap.get("mobile").length() == 11, "手机号码长度不是11位");

        code = 200003;
        Assert.isTrue(StringUtils.isNotEmpty(paramMap.get("password")), "密码不能为空");

        code = 200004;
        Assert.isTrue(StringUtils.isNotEmpty(paramMap.get("twoPassword")), "确认密码不能为空");
        code = 200005;
        Assert.isTrue(paramMap.get("password").equals(paramMap.get("twoPassword")), "两次密码不一致");
        code = 200007;
        Assert.isTrue(paramMap.get("password").matches(regex), "请输入6-8位的数字与字母组合");

        paramMap.put("ip",PayUtil.getIpAddr(request));
        result = userInfoService.register(paramMap,result);

        //删除redis中验证码
        redisTemplate.delete(RedisKey.MOBILE_SMSCODE+paramMap.get("mobile"));
    }catch (Exception e){
        e.printStackTrace();
        result.setCode(code);
        result.setMessage(e.getMessage());
    }
    return result;
}
```
- service
`ResponseResultUtil register(Map<String,String> paramMap, ResponseResultUtil result);`
- serviceIMpl
```
@Override
    public ResponseResultUtil register(Map<String,String> paramMap, ResponseResultUtil result) {

        UserInfo userInfo = new UserInfo();
        userInfo.setMobile(paramMap.get("mobile"));

        //检查是否已经注册过了
        List<UserInfo> userInfoList = selectByCondition(userInfo);
        if(userInfoList.size() > 0){
            result.setMessage("该手机号已注册，请直接登录");
            result.setCode(200006);
            return result;
        }

        userInfo = insertUserInfo(paramMap);

        Map dataMap = new HashMap();
        dataMap.put("token",userInfo.getToken());

        result.setMessage("注册成功");
        result.setData(dataMap);
        return result;
    }
```
- 因为注册的具体流程还有其他接口需要，所以此处将该方法抽象出来。
```
@Override
    public UserInfo insertUserInfo(Map<String,String> paramMap){
        String invitationCode = paramMap.get("invitationCode");
        UserInfo userInfo = new UserInfo();
        userInfo.setMobile(paramMap.get("mobile"));
        userInfo.setPassword(MD5Util.MD5Encode(StringUtils.isEmpty(invitationCode) ? paramMap.get("password") : invitationCode));
        userInfo.setSeriousStatus(1);
        userInfo.setDeviceCode(StringUtils.isEmpty(paramMap.get("deviceCode")) ? "" : paramMap.get("deviceCode"));
        userInfo.setPlatfrom(StringUtils.isEmpty(paramMap.get("platfrom")) ? null : Integer.parseInt(paramMap.get("platfrom")));
        userInfo.setPersonalizedSignature("编辑个性签名");
        userInfo.setNickname(paramMap.get("nickname")==null?userInfo.getMobile().replaceAll(userInfo.getMobile().substring(3, 7), "****"):paramMap.get("nickname"));
        userInfo.setHeadImage(userDefaultImage);//注册时添加黙认头像
        userInfo.setRole(paramMap.get("role")==null?1:Integer.parseInt(paramMap.get("role")));
        userInfo.setGender(1);
        //邀请码
        userInfo.setInvitationCode(StringUtils.isEmpty(invitationCode) ?  AESEncrypt.randomStr(6) : invitationCode);
        userInfo.setInvitationCodeEr(paramMap.get("invitationCodeEr"));
        userInfo.setUserName(userInfo.getMobile());
        userInfo.setCreateTime(new Date());
        userInfo.setUpdateTime(new Date());
        userInfo.setStatus(1);
        //新增用户
        userInfoMapper.insertSelective(userInfo);
        userInfo.setToken(TokenUtil.makeToken("token",userInfo.getUserId(),userInfo.getMobile(),paramMap.get("ip")));
        
        //其他业务逻辑

        return userInfo;
    }
```
mapper和xml都是自动生成的，这里就不贴了。

## 登录
### 验证码登录
- controller
```
@PostMapping(value = "jSmscodeLogin")
@ResponseBody
public ResponseResultUtil jSmscodeLogin(@RequestBody Map<String,String> paramMap){
    LOGGER.info(
            "\n***************************************" + "\n" +
                    "start jSmscodeLogin" + "\n" +
                    "验证码登录" + "\n" +
                    "paramMap = " + paramMap + "\n" +
                    "\n********************************************"
    );
    ResponseResultUtil result = ResponseResultUtil.responseSuccess();
    int code = ErrorEnum.ERROR_200000.getCode();
    try {
        // 获取手机验证码
        code = 200001;
        String 	smsCode = redisTemplate.opsForValue().get(RedisKey.MOBILE_SMSCODE + paramMap.get("mobile"));
        Assert.notNull(smsCode, "未获取验证码或验证码已超时，请先获取验证码");

        code = 200002;
        Assert.isTrue(StringUtils.isNotEmpty(paramMap.get("mobile")), "手机号码不能为空");

        code = 200003;
        Assert.isTrue(StringUtils.isNotEmpty(paramMap.get("smscode")), "验证码不能为空");

        code = 200005;
        Assert.isTrue(StringUtils.isNotEmpty(paramMap.get("platfrom")), "设备平台不可为空");

        code = 200006;
        Assert.isTrue(smsCode.equals(paramMap.get("smscode")), "验证码错误");

        paramMap.put("ip",PayUtil.getIpAddr(request));
        if(smsCode.equals(paramMap.get("smscode"))){
            //注册或登录
            result = userInfoService.registerOrLogin(paramMap,result);
            if(result.getCode() == ErrorEnum.ERROR_100000.getCode()){
                redisTemplate.delete(RedisKey.MOBILE_SMSCODE+paramMap.get("mobile"));
            }
        }else{
            result.setCode(code);
            result.setMessage("验证码错误");
        }
    }catch (Exception e){
        e.printStackTrace();
        result.setCode(code);
        result.setMessage(e.getMessage());
    }
    return result;
}
```
- service
`ResponseResultUtil registerOrLogin(Map<String,String> paramMap,ResponseResultUtil responseResultUtil);`

- serviceImpl
```
@Override
public ResponseResultUtil registerOrLogin(Map<String, String> paramMap,ResponseResultUtil responseResultUtil) {
    //登录类型
    Integer loginType = StringUtils.isEmpty(paramMap.get("loginType")) ? 1 : Integer.parseInt(paramMap.get("loginType"));
    UserInfo searchUserInfo = new UserInfo();
    searchUserInfo.setMobile(paramMap.get("mobile"));
    List<UserInfo> userInfoList = userInfoMapper.selectByCondition(searchUserInfo);
    UserInfo userInfo = null;
    if(userInfoList.size() > 0){
        userInfo = userInfoList.get(0);
        if(userInfo.getStatus() == 0){
            responseResultUtil.setCode(200000);
            responseResultUtil.setMessage("该账户违规,已被禁用");
        }else{
            if(loginType == 1 || userInfo.getRole().equals(loginType)){
                String token = TokenUtil.makeToken("token",userInfo.getUserId(),userInfo.getMobile(),paramMap.get("ip"));
                userInfo.setToken(token);
                //设置认证状态
                userInfo =  setUserSeriousStatus(userInfo);
                //更新设备码
                paramMap.put("userId",userInfo.getUserId().toString());
                paramMap.put("token",token);
                updateUserDeviceCode(paramMap);
                responseResultUtil.setCode(100000);
                responseResultUtil.setMessage("登录成功");
            }else{
                responseResultUtil.setCode(200000);
                responseResultUtil.setMessage("用户角色错误");
            }
        }
    }else{
        //注册
        String invitationCode = AESEncrypt.randomStr(6);
        paramMap.put("invitationCode",invitationCode);
        paramMap.put("role",loginType.toString());
        userInfo = insertUserInfo(paramMap);
        //刚注册为未认证
        userInfo.setSeriousStatus(-1);
        //注册成功后通知用户
        String msg= "您已完成注册. 初始密码为: " + userInfo.getInvitationCode() + " ;请您尽快进入App,进行密码修改.";//短信内容
        ChuangLanSmsUtil.sendSmsByPostSimple(paramMap.get("mobile"),msg);
        responseResultUtil.setCode(100000);
        responseResultUtil.setMessage("注册成功");
    }

    Map userInfoMap = BeanUtil.beanToMap(userInfo);
    userInfoMap.put("gender",userInfoMap.get("gender").equals(1) ? "男":"女");
    responseResultUtil.setData(userInfoMap);

    return responseResultUtil;
}
```
如果没有注册，就注册，否则登录。

### 账号密码登录
- controller
```
@PostMapping(value = "login")
@ResponseBody
public ResponseResultUtil login(@RequestBody Map<String,String> paramMap){
    LOGGER.info(
            "\n***************************************" + "\n" +
                    "start login" + "\n" +
                    "账号密码登录" + "\n" +
                    "paramMap = " + paramMap + "\n" +
                    "\n********************************************"
    );
    ResponseResultUtil result = ResponseResultUtil.responseSuccess();
    int code = ErrorEnum.ERROR_200000.getCode();
    try {
        code = 200001;
        Assert.isTrue(StringUtils.isNotEmpty(paramMap.get("mobile")), "登录账号不能为空");

        code = 200003;
        Assert.isTrue(StringUtils.isNotEmpty(paramMap.get("password")), "登陆密码不可为空");

        code = 200005;
        Assert.isTrue(StringUtils.isNotEmpty(paramMap.get("platfrom")), "设备平台不可为空");

        paramMap.put("ip",PayUtil.getIpAddr(request));
        result = userInfoService.registerOrLogin(paramMap,result);

    }catch (Exception e){
        e.printStackTrace();
        result.setCode(code);
        result.setMessage(e.getMessage());
    }
    return result;
}
```
- service
`ResponseResultUtil login(Map<String,String> paramMap,ResponseResultUtil result);`

- serviceImpl
```
@Override
public Map<String,Object> login(Map<String, String> paramMap,ResponseResultUtil result) {
    UserInfo searchUserInfo = new UserInfo();
    searchUserInfo.setMobile(paramMap.get("mobile"));
    searchUserInfo.setPassword(MD5Util.MD5Encode(paramMap.get("password")));
    List<UserInfo> userInfoList = selectByCondition(searchUserInfo);

    UserInfo userInfo = null;
    if(userInfoList.size() > 0){
        userInfo = userInfoList.get(0);
        if(userInfo.getStatus() == 0){
            result.setCode(200000);
            result.setMessage("该账户违规,已被禁用");
        }else{
            if(loginType == 1 || userInfo.getRole().equals(loginType)){
                String token = TokenUtil.makeToken("token",userInfo.getUserId(),userInfo.getMobile(),paramMap.get("ip"));
                userInfo.setToken(token);
                //设置认证状态
                userInfo =  setUserSeriousStatus(userInfo);
                //更新设备码和Token
                paramMap.put("userId",userInfo.getUserId().toString());
                paramMap.put("token",token);
                updateUserDeviceCode(paramMap);

                result.setCode(100000);
                result.setMessage("登录成功");
            }else{
                result.setCode(200000);
                result.setMessage("用户角色错误");
            }
        }
    }else{
        result.setCode(200000);
        result.setMessage("手机号或密码错误!");
    }

    Map userInfoMap = BeanUtil.beanToMap(userInfo);
    userInfoMap.put("gender",userInfoMap.get("gender").equals(1) ? "男":"女");
    responseResultUtil.setData(userInfoMap);

    return result;
}
```

## 获取验证码
- controller
```
@ResponseBody
@PostMapping(value = "getSmsCode")
public ResponseResultUtil getSmsCode(@RequestBody Map<String,String> paramMap){
    LOGGER.info(
            "\n***************************************" + "\n" +
                    "start getSmsCode" + "\n" +
                    "获取短信验证码" + "\n" +
                    "paramMap = " + paramMap + "\n" +
                    "\n********************************************"
    );
    ResponseResultUtil result = ResponseResultUtil.responseSuccess();
    int code = ErrorEnum.ERROR_200000.getCode();
    try {
        String mobile = paramMap.get("mobile");
        code = 202001;
        Assert.isTrue(StringUtils.isNotEmpty(mobile), "手机号码不能为空");
        code = 202002;
        Assert.isTrue(mobile.length() == 11, "手机号码长度不是11位");

        String str =  sendSmsCode(mobile);
        if(StringUtils.isNotEmpty(str)){
            result.setCode(200000);
            result.setMessage(str);
        }

    }catch (Exception e){
        e.printStackTrace();
        result.setCode(code);
        result.setMessage(e.getMessage());
    }
    return result;
}
```
- 发送验证码具体方法，注意redis
```
private String sendSmsCode(String mobile){
    Random random = new Random();
    String smsCode = String.valueOf(1000 + random.nextInt(8999));
    String key = RedisKey.MOBILE_SMSCODE + mobile;

    if(redisTemplate.opsForValue().get(RedisKey.MOBILE_SMSCODE+mobile) == null){
        //短信内容
        String msg= "您的短信验证码是"+smsCode+",请勿告他人,五分钟内有效。";
        LOGGER.debug(msg);
        ChuangLanSmsUtil.sendSmsByPostSimple(mobile,msg);
        //验证码有效期300秒(5分钟)
        redisTemplate.opsForValue().set(key, smsCode, 300,TimeUnit.SECONDS);
        //一个手机号请求接口需要间隔1分钟(过期时间1分钟)
        redisTemplate.opsForValue().set(RedisKey.MOBILE_SMSCODE+mobile,smsCode,60,TimeUnit.SECONDS);
    }else{
        return "1分钟内只能获取一次验证码";
    }
    return "";
}
```

## 判断验证码是否正确
- controller
```
@ResponseBody
@PostMapping(value = "/jSmscodeRegister")
public ResponseResultUtil jSmscodeRegister(@RequestBody Map<String,String> paramMap){
    LOGGER.info("\n***************************************start jSmscodeRegist() 判断验证码是否正确 ***************************************\n");
    ResponseResultUtil result = ResponseResultUtil.responseSuccess();
    int code = ErrorEnum.ERROR_200000.getCode();
    try {
        String smsCode = redisTemplate.opsForValue().get(RedisKey.MOBILE_SMSCODE + paramMap.get("mobile"));
        code = 200001;
        Assert.notNull(smsCode, "未获取验证码或验证码已超时，请先获取验证码");
        code = 200003;
        Assert.isTrue(StringUtils.isNotEmpty(paramMap.get("mobile")), "手机号不能为空");
        code = 200002;
        Assert.isTrue(StringUtils.isNotEmpty(paramMap.get("smscode")), "验证码不能为空");


        UserInfo userInfo = new UserInfo();
        userInfo.setMobile(paramMap.get("mobile"));
        List<UserInfo> userInfoList =  userInfoService.selectByCondition(userInfo);
        if(userInfoList.size() > 0){
            code = 202004;
            result.setMessage("该手机号已注册，请直接登录");
        }else{
            result = checkSmsCode(smsCode,paramMap.get("smscode"),paramMap.get("mobile"),result);
        }
        result.setCode(code);
    }catch (Exception e){
        e.printStackTrace();
        result.setCode(code);
        result.setMessage(e.getMessage());
    }
    return result;
}
```
- checkSmsCode方法，判断验证码是否正确
```
private ResponseResultUtil checkSmsCode(Object smsCode,String smsCodeParam,String mobile,ResponseResultUtil result){
    if(smsCode.equals(smsCodeParam)){
        result.setCode(100000);
        result.setMessage("验证码正确");
//            redisTemplate.delete("smsCode" + mobile);
        redisTemplate.delete(RedisKey.MOBILE_SMSCODE + mobile);
    }else{
        result.setCode(200004);
        result.setMessage("验证码错误");
    }
    return result;
}
```

## 判断密码是否正确
- controller
```
@ResponseBody
@PostMapping(value = "judgePassword")
public ResponseResultUtil judgePassword(@RequestBody Map<String,String> paramMap){
    LOGGER.info(
            "\n***************************************" + "\n" +
                    "start judgePassword" + "\n" +
                    "判断密码是否正确" + "\n" +
                    "paramMap = " + paramMap + "\n" +
                    "\n********************************************"
    );
    ResponseResultUtil result = ResponseResultUtil.responseSuccess();
    int code = ErrorEnum.ERROR_200000.getCode();
    try {

        Token token = TokenUtil.getToken(request.getHeader("token"));
        Assert.isTrue(token != null, "token解密失败");
        code = 200001;
        Assert.isTrue(StringUtils.isNotEmpty(paramMap.get("password")), "密码不可为空");

        Integer userId =  token.getId();
        UserInfo userInfo = new UserInfo();
        userInfo.setUserId(userId);
        userInfo =  userInfoService.selectByCondition(userInfo).get(0);
        String password=MD5Util.MD5Encode(paramMap.get("password"));
        if(!userInfo.getPassword().equals(password)){
            result.setCode(200002);
            result.setMessage("密码错误");
        }else{
            result.setCode(100000);
            result.setMessage("密码正确");
        }
    }catch (Exception e){
        e.printStackTrace();
        result.setCode(code);
        result.setMessage(e.getMessage());
    }
    return result;
}
```

## 修改密码
- controller
```
@PostMapping(value = "updatePassword")
@ResponseBody
public ResponseResultUtil updatePassword(@RequestBody Map<String,String> paramMap){
    LOGGER.info(
            "\n***************************************" + "\n" +
                    "start updatePassword" + "\n" +
                    "修改密码" + "\n" +
                    "paramMap = " + paramMap + "\n" +
                    "\n********************************************"
    );
    ResponseResultUtil result = ResponseResultUtil.responseSuccess();
    int code = ErrorEnum.ERROR_200000.getCode();
    try {
        Token token = TokenUtil.getToken(request.getHeader("token"));
        String regex = "^(?![0-9]+$)(?![a-zA-Z]+$)[0-9A-Za-z]{6,10}$";

        Assert.isTrue(token != null, "token解密失败");

//            code = 200001;
//            Assert.isTrue(paramMap.get("password") != null, "旧密码不可为空");

        code = 200002;
        Assert.isTrue(paramMap.get("newPassword")!= null, "新密码不可为空");

        code = 200007;
        Assert.isTrue(paramMap.get("newPassword").matches(regex), "请输入6-8位的数字与字母组合");

        code = 200003;
        Assert.isTrue(paramMap.get("twoPassword")!= null, "确认新密码不可为空");

        code = 200004;
        Assert.isTrue(paramMap.get("newPassword").equals(paramMap.get("twoPassword")), "两次密码不一致");

        paramMap.put("userId",token.getId().toString());
        result =  userInfoService.updatePassword(paramMap,result);

    }catch (Exception e){
        e.printStackTrace();
        result.setCode(code);
        result.setMessage(e.getMessage());
    }
    return result;
}
```

## 忘记密码
- controller
```
@PostMapping(value = "forGotPassword")
@ResponseBody
public ResponseResultUtil forGotPassword(@RequestBody Map<String,String> paramMap){
    LOGGER.info(
            "\n***************************************" + "\n" +
                    "start forGotPassword" + "\n" +
                    "忘记密码" + "\n" +
                    "paramMap = " + paramMap + "\n" +
                    "\n********************************************"
    );
    ResponseResultUtil result = ResponseResultUtil.responseSuccess();
    int code = ErrorEnum.ERROR_200000.getCode();
    try {
        code = 202001;
        Assert.isTrue(paramMap.get("mobile")!= null, "手机号码不能为空");

        code = 200002;
        Assert.isTrue(paramMap.get("newPassword")!= null, "新密码不可为空");

        String regex = "^(?![0-9]+$)(?![a-zA-Z]+$)[0-9A-Za-z]{6,10}$";
        code = 200007;
        Assert.isTrue(paramMap.get("newPassword").matches(regex), "请输入6-8位的数字与字母组合");

        code = 200003;
        Assert.isTrue(paramMap.get("twoPassword")!= null, "确认新密码不可为空");

        code = 200004;
        Assert.isTrue(paramMap.get("newPassword").equals(paramMap.get("twoPassword")), "两次密码不一致");

        UserInfo searchUserInfo = new UserInfo();
        searchUserInfo.setMobile(paramMap.get("mobile"));
        List<UserInfo> userInfoList = userInfoService.selectByCondition(searchUserInfo);

        if(userInfoList.size() > 0){
            UserInfo userInfo = userInfoList.get(0);
            userInfo.setPassword(MD5Util.MD5Encode(paramMap.get("twoPassword")));
            if(userInfoService.updateUserInfo(userInfo)){
                result.setCode(100000);
                result.setMessage("修改成功");
            }else{
                result.setCode(200000);
                result.setMessage("修改失败，请重试");
            }
        }else{
            result.setCode(200001);
            result.setMessage("没有该账号");
        }

    }catch (Exception e){
        e.printStackTrace();
        result.setCode(code);
        result.setMessage(e.getMessage());
    }
    return result;
}
```
service、mapper、xml很简单，就不贴了。
