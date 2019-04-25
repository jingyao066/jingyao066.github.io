---
title: API-notice
tags: java
date: 2019-04-03 09:54:28
---

# 模板
```java
@PostMapping("/login")
public ResponseUtil login(@RequestBody Map<String,String> paramMap){
    LOGGER.info(
    "\n***************************************" + "\n" +
            "start login" + "\n" +
            "登录" + "\n" +
            "paramMap = " + paramMap + "\n" +
            "\n********************************************"
    );
    ResponseUtil response = ResponseUtil.success();
    CodeEnum code = CodeEnum.FAIL;
    try {
        Token token = TokenUtil.getToken(request.getHeader("token"));
        Assert.isTrue(token != null, "token解密失败");

        code = CodeEnum.ERROR_2001;
        Assert.isTrue(StringUtils.isNotEmpty(paramMap.get("mobile")), "手机号不可为空");
        response = usersService.login(paramMap,response,token);
    } catch (Exception e) {
        e.printStackTrace();
        response.setCode(code);
        response.setMessage(e.getMessage());
    }
    return response;
}
```

# Controller
- controller只负责收发参数、跳转页面等，不要在controller里写业务逻辑，业务逻辑全部写在接口实现类(serviceImpl)中。
- 入参使用`@RequestBody Map<String,String> map`，返回数据使用`@ResponseBody`
- 规范的使用restful
- 使用LOGGER.info()打印入参，方便线上调试。
- 使用ResponseUtil返回数据、code码和message。
    通过code码和message来向前端反馈信息，例如：1000代表成功，message为“请求成功”，2000代表失败，message为“请求失败”，其他自行定义。
- 使用断言验证参数：`Assert.isTrue()`
- 用户相关的接口都需要token，token需要前端通过header传递。

示例：
```java
@ResponseBody
@PostMapping("/login")
    public ResponseUtil login(@RequestBody Map<String,String> paramMap){
        LOGGER.info(
        "\n***************************************" + "\n" +
                "start login" + "\n" +
                "登录" + "\n" +
                "paramMap = " + paramMap + "\n" +
                "\n********************************************"
        );
        ResponseUtil response = ResponseUtil.success();
        CodeEnum code = CodeEnum.FAIL;
        try {
            Token token = TokenUtil.getToken(request.getHeader("token"));
            Assert.isTrue(token != null, "token解密失败");
            
            code = CodeEnum.ERROR_2001;
            Assert.isTrue(StringUtils.isNotEmpty(paramMap.get("mobile")), "手机号不可为空");

            response = usersService.login(paramMap,response,token);
        } catch (Exception e) {
            e.printStackTrace();
            response.setCode(code);
            response.setMessage(e.getMessage());
        }
        LOGGER.info(
        "\n***************************************" + "\n" +
                "end getUserInfo" + "\n" +
                "返回个人信息" + "\n" +
                "result = " + result.getData() + "\n" +
                "\n********************************************"
        );
        return response;
    }
```
还有另一种打印日志的样式：
```
LOGGER.info("\n***************\n start getUserinfo\n 获取个人信息 \n paramMap " + paramMap + "\n***************");
LOGGER.info("\n***************\n end getUserinfo\n 返回个人信息 \n result " + result.getData() + "\n***************");
```
两种样式二选一

运行异常时，可以使用logger打印：
`LOGGER.error("\n********************运行异常********************", e);`
这种方式和`e.printStackTrace();`有什么区别？

需要的LOGGER对象需要在类头部定义：
`private final static Logger LOGGER = LoggerFactory.getLogger(UserInfoController.class);`

需要的request、response、session需要在类头部定义：
```java
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

# ResponseUtil
```java
public class ResponseUtil<T> implements Serializable {
    private int code;
    private String message;
    private T data;

    public ResponseUtil() {

    }

    public ResponseUtil(CodeEnum code) {
        this.code = code.getCode();
        this.message = code.getMessage();
    }

    public static ResponseUtil success() {
        return new ResponseUtil(CodeEnum.SUCCESS);
    }

    public static ResponseUtil fail() {
        return new ResponseUtil(CodeEnum.FAIL);
    }

    public int getCode() {
        return code;
    }

    public void setCode(CodeEnum code) {
        this.code = code.getCode();
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
一般搭配CodeEnum状态码枚举类使用，错误码可自行定义：
```
public enum CodeEnum {

    //请求成功
    SUCCESS(1000,"请求成功"),

    //请求失败
    FAIL(2000,"请求失败"),

    //其他错误码
    ERROR_2001(2001),
    ERROR_2002(2002),
    ERROR_2003(2003),
    ERROR_2004(2004),
    ERROR_2005(2005),
    ERROR_2006(2006),
    ERROR_2007(2007),
    ERROR_2008(2008),
    ERROR_2009(2009),
    ERROR_2010(2010);

    private int code;
    private String message;
    private String desc;

    private CodeEnum(int code){
        this.code = code;
    }

    private CodeEnum(int code, String msg) {
        this.code = code;
        this.message = msg;
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
这个枚举类只定义了10个错误码，这里一码对应多种信息，只是从枚举类中获取状态码，具体信息从接口中定义。
状态码枚举类还有一种使用方法。就是在类中的每个枚举值定义到具体的信息，即一码对一信息。如：
`ERROR_2020(2020,"一些具体的错误信息"),`
这么写的缺点就是后期该类会变得非常长，不便于维护，而且本来可以用一种码表示多种信息，但是因为枚举值是别人写的，乱改可能会引起问题，也不该乱改，只能在后边加新的枚举值，这样枚举类会变的很长...

# 典型接口示例
## 分页接口
- controller
```java
@PostMapping("/getGoodsList")
public ResponseUtil getGoodsList(@RequestBody Map<String, String> paramMap) {
    LOGGER.info(
            "\n***************************************" + "\n" +
                    "start getGoodsList" + "\n" +
                    "商品列表" + "\n" +
                    "paramMap = " + map + "\n" +
                    "\n********************************************"
    );
    ResponseUtil response = ResponseUtil.success();
    CodeEnum code = CodeEnum.FAIL;
    try {
        result = goodsService.getGoodsList(paramMap);
    } catch (Exception e) {
        e.printStackTrace();
        response.setCode(code);
        response.setMessage(e.getMessage());
    }
    LOGGER.info(
            "\n***************************************" + "\n" +
                    "end getGoodsList" + "\n" +
                    "商品列表" + "\n" +
                    " resultMap " + response.getData() + "\n" +
                    "\n********************************************"
    );
    return response;
}
```
- service
`List<Goods> getGoodsList(Map<String, String> paramMap);`
或
`List<Map<String, Object>> getGoodsList(Map<String, String> paramMap);`

- serviceImpl
```
public List<Goods> getGoodsList(Map<String, String> paramMap) {
    PageHelper.startPage(
            paramMap.get("pageNum") == null ? 1 : Integer.parseInt(paramMap.get("pageNum")),
            paramMap.get("pageSize") == null ? 10 : Integer.parseInt(paramMap.get("pageSize")));
    List<Goods> list = goodsMapper.getGoodsList(paramMap);
    return list;
}
```

- mapper
`List<Goods> getGoodsList(Map<String,String> map);`
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
@PostMapping("/register")
public ResponseUtil register(@RequestBody Map<String,String> paramMap){
    LOGGER.info(
    "\n***************************************" + "\n" +
            "start register" + "\n" +
            "注册" + "\n" +
            "paramMap = " + paramMap + "\n" +
            "\n********************************************"
    );
    ResponseUtil response = ResponseUtil.success();
    CodeEnum codeEnum = CodeEnum.FAIL;
    try {
        String regex = "^(?![0-9]+$)(?![a-zA-Z]+$)[0-9A-Za-z]{6,10}$";
        codeEnum = CodeEnum.ERROR_2002;
        Assert.isTrue(StringUtils.isNotEmpty(paramMap.get("mobile")), "手机号码不能为空");
        codeEnum = CodeEnum.ERROR_2003;
        Assert.isTrue(paramMap.get("mobile").length() == 11, "手机号码长度不是11位");
        codeEnum = CodeEnum.ERROR_2004;
        Assert.isTrue(StringUtils.isNotEmpty(paramMap.get("password")), "密码不能为空");
        codeEnum = CodeEnum.ERROR_2005;
        Assert.isTrue(StringUtils.isNotEmpty(paramMap.get("confirmPassword")), "确认密码不能为空");
        codeEnum = CodeEnum.ERROR_2006;
        Assert.isTrue(paramMap.get("password").equals(paramMap.get("confirmPassword")), "两次密码不一致");
        codeEnum = CodeEnum.ERROR_2007;
        Assert.isTrue(paramMap.get("password").matches(regex), "请输入6-8位的数字与字母组合");

        paramMap.put("ip", HttpUtil.getIpAddr(request));
        response = usersService.register(paramMap,response);
        //删除redis中验证码
        redisTemplate.delete(RedisKeyEnum.MOBILE_SMSCODE.getValue() +paramMap.get("mobile"));
    } catch (Exception e) {
        e.printStackTrace();
        response.setCode(codeEnum);
        response.setMessage(e.getMessage());
    }

    return response;
}
```
- service
`ResponseUtil register(Map<String,String> paramMap, ResponseUtil response);`

- serviceIMpl
```
@Override
public ResponseUtil register(Map<String, String> paramMap, ResponseUtil response) {
    Users user = new Users();
    user.setMobile(paramMap.get("mobile"));
    List<Users> userLIst = usersMapper.getUserByCondition(user);
    //是否已注册
    if (userLIst.size() > 0) {
        response.setMessage("该手机号已注册，请直接登录");
        response.setCode(CodeEnum.ERROR_2001);
        return response;
    }

    user = insertUser(paramMap);
    Map<String,Object> resultMap = new HashedMap();
    resultMap.put("token",user.getToken());

    response.setMessage("注册成功");
    response.setData(resultMap);
    return response;
}
```
- 因为注册的具体流程还有其他接口需要，所以此处将该方法抽象出来。
```
public Users insertUser(Map<String, String> paramMap) {
    Users newUser = new Users();
    newUser.setMobile(paramMap.get("mobile"));
    newUser.setPassword(MD5Util.MD5Encode(paramMap.get("password")));
    newUser.setPlatform(StringUtils.isEmpty(paramMap.get("platfrom")) ? null : Byte.parseByte(paramMap.get("platfrom")));
    newUser.setNickname(paramMap.get("nickname") == null ? newUser.getMobile().replaceAll(newUser.getMobile().substring(3, 7), "****") : paramMap.get("nickname"));
    newUser.setDeviceCode(paramMap.get("deviceCode"));

    //执行新增用户
    usersMapper.insertSelective(newUser);
    newUser.setToken(TokenUtil.makeToken("token",newUser.getId(),newUser.getMobile(),paramMap.get("ip")));

    //其他业务逻辑

    return newUser;
}
```
mapper和xml都是自动生成的，这里就不贴了。

## 登录
### 验证码登录
- controller
```
@PostMapping("/smsCodeLogin")
public ResponseUtil smsCodeLogin(@RequestBody Map<String,String> paramMap){
    LOGGER.info(
    "\n***************************************" + "\n" +
            "start smsCodeLogin" + "\n" +
            "验证码登录" + "\n" +
            "paramMap = " + paramMap + "\n" +
            "\n********************************************"
    );
    ResponseUtil response = ResponseUtil.success();
    CodeEnum code = CodeEnum.FAIL;
    try {
        // 获取手机验证码
        code = CodeEnum.ERROR_2001;
        String 	smsCode = redisTemplate.opsForValue().get(RedisKeyEnum.MOBILE_SMSCODE.getValue() + paramMap.get("mobile"));
        Assert.notNull(smsCode, "未获取验证码或验证码已超时，请先获取验证码");
        code = CodeEnum.ERROR_2002;
        Assert.isTrue(StringUtils.isNotEmpty(paramMap.get("mobile")), "手机号码不能为空");
        code = CodeEnum.ERROR_2003;
        Assert.isTrue(StringUtils.isNotEmpty(paramMap.get("smscode")), "验证码不能为空");
        code = CodeEnum.ERROR_2004;
        Assert.isTrue(StringUtils.isNotEmpty(paramMap.get("platfrom")), "设备平台不可为空");
        code = CodeEnum.ERROR_2005;
        Assert.isTrue(smsCode.equals(paramMap.get("smscode")), "验证码错误");

        paramMap.put("ip",HttpUtil.getIpAddr(request));

        if(smsCode.equals(paramMap.get("smscode"))){
            response = usersService.login(paramMap,response);
            //登录成功，删除redis中的验证码
            if(response.getCode() == CodeEnum.SUCCESS.getCode()){
                redisTemplate.delete(RedisKeyEnum.MOBILE_SMSCODE.getValue() + paramMap.get("mobile"));
            }
        }else{
            response.setCode(code);
            response.setMessage("验证码错误");
        }
    } catch (Exception e) {
        e.printStackTrace();
        response.setCode(code);
        response.setMessage(e.getMessage());
    }
    return response;
}
```
- service
`ResponseUtil login(Map<String,String> paramMap, ResponseUtil response);`

- serviceImpl
```
@Override
public ResponseUtil login(Map<String, String> paramMap, ResponseUtil response) {
    Users user = new Users();
    user.setMobile(paramMap.get("mobile"));
    //密码登录用
    String password = paramMap.get("password");
    if(!StringUtils.isEmpty(password)){
        user.setPassword(MD5Util.MD5Encode(paramMap.get("password")));
    }
    List<Users> userList = usersMapper.getUserByCondition(user);
    if(userList.size() > 0){
        user = userList.get(0);
        if(user.getStatus() == 1){
            response.setCode(CodeEnum.ERROR_2006);
            response.setMessage("该账户已被禁用");
        }else{
            String token = TokenUtil.makeToken("token",user.getId(),user.getMobile(),paramMap.get("ip"));
            user.setToken(token);
            paramMap.put("id",user.getId().toString());
            paramMap.put("token",token);
            //更新设备码、token
            updateUserInfo(paramMap);
            response.setMessage("登录成功");
        }
    }else{
        response.setCode(CodeEnum.ERROR_2007);
        if(!StringUtils.isEmpty(password)){
            response.setMessage("手机号或密码错误");
        }else{
            response.setMessage("请先注册");
        }
        return response;
    }
    Map<String,Object> resultMap = BeanUtil.beanToMap(user);
    response.setData(resultMap);
    return response;
}
```
验证码登录和用户名密码登录业务逻辑十分相似，这里共用了一个service，有需要可以将这两个方法拆分开。

### 账号密码登录
- controller
```
@PostMapping("/login")
public ResponseUtil login(@RequestBody Map<String,String> paramMap){
    LOGGER.info(
    "\n***************************************" + "\n" +
            "start login" + "\n" +
            "手机号密码登录" + "\n" +
            "paramMap = " + paramMap + "\n" +
            "\n********************************************"
    );
    ResponseUtil response = ResponseUtil.success();
    CodeEnum code = CodeEnum.FAIL;
    try {
        code = CodeEnum.ERROR_2002;
        Assert.isTrue(StringUtils.isNotEmpty(paramMap.get("mobile")), "手机号不能为空");
        code = CodeEnum.ERROR_2003;
        Assert.isTrue(StringUtils.isNotEmpty(paramMap.get("password")), "密码不能为空");
        code = CodeEnum.ERROR_2004;
        Assert.isTrue(StringUtils.isNotEmpty(paramMap.get("platfrom")), "设备平台不可为空");
        paramMap.put("ip",HttpUtil.getIpAddr(request));
        response = usersService.login(paramMap,response);
    } catch (Exception e) {
        e.printStackTrace();
        response.setCode(code);
        response.setMessage(e.getMessage());
    }
    return response;
}
```
和上边的验证码登录共用一套service，有需要可以拆分开。

## 获取验证码
- controller
```
@PostMapping("/getSmsCode")
    public ResponseUtil getSmsCode(@RequestBody Map<String,String> paramMap){
        LOGGER.info(
        "\n***************************************" + "\n" +
                "start getSmsCode" + "\n" +
                "获取验证码" + "\n" +
                "paramMap = " + paramMap + "\n" +
                "\n********************************************"
        );
        ResponseUtil response = ResponseUtil.success();
        CodeEnum codeEnum = CodeEnum.FAIL;
        try {
            String mobile = paramMap.get("mobile");
            codeEnum = codeEnum.ERROR_2001;
            Assert.isTrue(StringUtils.isNotEmpty(mobile), "手机号码不能为空");
            codeEnum = codeEnum.ERROR_2002;
            Assert.isTrue(mobile.length() == 11, "手机号码长度不是11位");

            String msg = sendSmsCode(mobile);
            //1分钟只能获取一次验证码
            if(StringUtils.isNotEmpty(msg)){
                response.setCode(CodeEnum.ERROR_2003);
                response.setMessage(msg);
            }

        } catch (Exception e) {
            e.printStackTrace();
            response.setCode(codeEnum);
            response.setMessage(e.getMessage());
        }
        return response;
    }
```
- 发送验证码具体方法，注意redis
```
private String sendSmsCode(String mobile){
    Random random = new Random();
    String smsCode = String.valueOf(1000 + random.nextInt(8999));
    String key = RedisKeyEnum.MOBILE_SMSCODE.getValue() + mobile;
    //检查redis中是否有验证码
    if(redisTemplate.opsForValue().get(key) == null){
        String msg= "您的短信验证码是" + smsCode + "，打死也不能告诉别人，五分钟内有效。";
        LOGGER.info("验证码是："+msg);
        ChuangLanSmsUtil.sendSmsByPostSimple(mobile,msg);
        //1分钟只能请求一次
        redisTemplate.opsForValue().set(key,smsCode,60,TimeUnit.SECONDS);
    }else{
        return "1分钟内只能获取一次验证码";
    }
    return "";
}
```

## 判断验证码是否正确
- controller
```
@PostMapping("/checkSmsCode")
public ResponseUtil checkSmsCode(@RequestBody HashMap<String,String> paramMap){
    LOGGER.info(
    "\n***************************************" + "\n" +
            "start checkSmsCode" + "\n" +
            "判断验证码是否正确" + "\n" +
            "paramMap = " + paramMap + "\n" +
            "\n********************************************"
    );
    ResponseUtil response = ResponseUtil.success();
    CodeEnum code = CodeEnum.FAIL;
    try {
        String smsCode = redisTemplate.opsForValue().get(RedisKeyEnum.MOBILE_SMSCODE.getValue() + paramMap.get("mobile"));
        code = CodeEnum.ERROR_2001;
        Assert.notNull(smsCode, "未获取验证码或验证码已超时，请先获取验证码");
        code = CodeEnum.ERROR_2002;
        Assert.isTrue(StringUtils.isNotEmpty(paramMap.get("mobile")), "手机号不能为空");
        code = CodeEnum.ERROR_2003;
        Assert.isTrue(StringUtils.isNotEmpty(paramMap.get("smscode")), "验证码不能为空");

        Users user = new Users();
        user.setMobile(paramMap.get("mobile"));
        List<Users> userList =  usersService.getUserByCondition(user);
        if(userList.size() > 0){
            code = CodeEnum.ERROR_2004;
            response.setCode(code);
            response.setMessage("该手机号已注册，请直接登录");
        }else{
            response = checkSmsCode(smsCode,paramMap.get("smscode"),paramMap.get("mobile"));
        }
    } catch (Exception e) {
        e.printStackTrace();
        response.setCode(code);
        response.setMessage(e.getMessage());
    }
    return response;
}
```
- checkSmsCode方法，判断验证码是否正确
```
private ResponseUtil checkSmsCode(String redisSmsCode,String ParamSmsCode,String mobile){
    ResponseUtil response = ResponseUtil.success();
    if(redisSmsCode.equals(ParamSmsCode)){
        response.setCode(CodeEnum.SUCCESS);
        response.setMessage("验证码正确");
        //验证成功后，删除redis中的验证码
        redisTemplate.delete(RedisKeyEnum.MOBILE_SMSCODE.getValue() + mobile);
    }else{
        response.setCode(CodeEnum.ERROR_2010);
        response.setMessage("验证码错误");
    }
    return response;
}
```

## 检查密码是否正确
- controller
```
@PostMapping("/checkPassword")
public ResponseUtil checkPassword(@RequestBody Map<String,String> paramMap){
    LOGGER.info(
    "\n***************************************" + "\n" +
            "start checkPassword" + "\n" +
            "检查密码是否正确" + "\n" +
            "paramMap = " + paramMap + "\n" +
            "\n****************************************"
    );
    ResponseUtil response = ResponseUtil.success();
    CodeEnum code = CodeEnum.FAIL;
    try {
        Token token = TokenUtil.getToken(request.getHeader("token"));
        Assert.isTrue(token != null, "token解密失败");
        code = CodeEnum.ERROR_2001;
        Assert.isTrue(StringUtils.isNotEmpty(paramMap.get("password")), "密码不可为空");
        response = usersService.checkPassword(paramMap,response,token);
    } catch (Exception e) {
        e.printStackTrace();
        response.setCode(code);
        response.setMessage(e.getMessage());
    }
    return response;
}
```
- serviceImpl
```
@Override
public ResponseUtil checkPassword(Map<String, String> paramMap, ResponseUtil response, Token token) {
    Integer id =  token.getId();
    Users user = new Users();
    user.setId(id);
    user = usersMapper.getUserByCondition(user).get(0);
    String password=MD5Util.MD5Encode(paramMap.get("password"));
    if(!user.getPassword().equals(password)){
        response.setCode(CodeEnum.ERROR_2002);
        response.setMessage("密码错误");
    }else{
        response.setCode(CodeEnum.SUCCESS);
        response.setMessage("密码正确");
    }
    return response;
}
```

## 修改密码
- controller
```
@PostMapping("/updatePassword")
public ResponseUtil updatePassword(@RequestBody Map<String,String> paramMap){
    LOGGER.info(
    "\n***************************************" + "\n" +
            "start updatePassword" + "\n" +
            "修改密码" + "\n" +
            "paramMap = " + paramMap + "\n" +
            "\n********************************************"
    );
    ResponseUtil response = ResponseUtil.success();
    CodeEnum code = CodeEnum.FAIL;
    try {
        Token token = TokenUtil.getToken(request.getHeader("token"));
        String regex = "^(?![0-9]+$)(?![a-zA-Z]+$)[0-9A-Za-z]{6,10}$";
        Assert.isTrue(token != null, "token解密失败");
        code = CodeEnum.ERROR_2001;
        Assert.isTrue(paramMap.get("newPassword")!= null, "新密码不可为空");
        code = CodeEnum.ERROR_2002;
        Assert.isTrue(paramMap.get("newPassword").matches(regex), "请输入6-8位的数字与字母组合");
        code = CodeEnum.ERROR_2003;
        Assert.isTrue(paramMap.get("confirmPassword")!= null, "确认新密码不可为空");
        code = CodeEnum.ERROR_2004;
        Assert.isTrue(paramMap.get("newPassword").equals(paramMap.get("confirmPassword")), "两次密码不一致");

        response = usersService.updatePassword(paramMap,response,token);
    } catch (Exception e) {
        e.printStackTrace();
        response.setCode(code);
        response.setMessage(e.getMessage());
    }
    return response;
}
```
- serviceImpl
```
@Override
public ResponseUtil updatePassword(Map<String, String> paramMap, ResponseUtil response, Token token) {
    Users user = usersMapper.selectByPrimaryKey(token.getId());
    if(null != user){
        user.setPassword(MD5Util.MD5Encode(paramMap.get("confirmPassword")));
        int i = usersMapper.updateByPrimaryKeySelective(user);
        if(i == 1){
            response.setMessage("修改密码成功");
        }else{
            response.setCode(CodeEnum.ERROR_2005);
            response.setMessage("修改密码失败");
        }
    }else{
        response.setCode(CodeEnum.ERROR_2006);
        response.setMessage("该用户不存在");
    }
    return response;
}
```

## 修改手机号
```
@PostMapping("/modifyMobile")
public ResponseUtil modifyMobile(@RequestBody Map<String,String> paramMap){
    LOGGER.info(
    "\n***************************************" + "\n" +
            "start modifyMobile" + "\n" +
            "修改手机号" + "\n" +
            "paramMap = " + paramMap + "\n" +
            "\n********************************************"
    );
    ResponseUtil response = ResponseUtil.success();
    CodeEnum code = CodeEnum.FAIL;
    try {
        Token token = TokenUtil.getToken(request.getHeader("token"));
        Assert.isTrue(token != null, "token解密失败");
        // 获取手机验证码
        code = CodeEnum.ERROR_2001;
        String 	smsCode = redisTemplate.opsForValue().get(RedisKeyEnum.MOBILE_SMSCODE.getValue() + paramMap.get("newMobile"));
        Assert.notNull(smsCode, "未获取验证码或验证码已超时，请先获取验证码");
        code = CodeEnum.ERROR_2002;
        Assert.isTrue(StringUtils.isNotEmpty(paramMap.get("smsCode")), "验证码不能为空");
        code = CodeEnum.ERROR_2003;
        Assert.isTrue(StringUtils.isNotEmpty(paramMap.get("newMobile")), "手机号不可为空");
        code = CodeEnum.ERROR_2004;
        Assert.isTrue(smsCode.equals(paramMap.get("smsCode")), "验证码错误");
        response = usersService.modifyMobile(paramMap,response,token);
    } catch (Exception e) {
        e.printStackTrace();
        response.setCode(code);
        response.setMessage(e.getMessage());
    }
    return response;
}
```
- serviceImpl
```
@Override
public ResponseUtil modifyMobile(Map<String, String> paramMap, ResponseUtil response, Token token) {
    Users oldUser = usersMapper.selectByPrimaryKey(token.getId());
    if(null != oldUser){
        Users user = new Users();
        user.setMobile(paramMap.get("newMobile"));
        List<Users> userList = usersMapper.getUserByCondition(user);
        if(userList.isEmpty()){
            oldUser.setMobile(paramMap.get("newMobile"));
            usersMapper.updateByPrimaryKeySelective(oldUser);
            response.setMessage("手机号修改成功");
        }else{
            response.setCode(CodeEnum.ERROR_2006);
            response.setMessage("该手机号已被占用");
        }
    }else{
        response.setCode(CodeEnum.ERROR_2005);
        response.setMessage("用户不存在");
    }
    return response;
}
```
