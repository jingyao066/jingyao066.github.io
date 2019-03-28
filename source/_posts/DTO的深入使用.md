---
title: DTO的深入使用
tags: java
date: 2018-12-09 00:44:08
---

# 前言
在写接口的时候，如果返回实体类，会返回一些前端并不需要的参数，例如：
学生和班级是一对一，我们会在学生类中，加入班级的实体类，构成一对一关系；
部门和员工是一对多，我们会在部门类中，加入List<Employee>，list泛型实体类，构成一对多关系。
这样我们其他的接口在用到学生或部门类的时候，就会返回我们不需要的空数据，为null，影响接口效率，整洁程度。
还有可能泄露数据库表结构。
在使用dto之后，我们新建dto类继承部门类，然后再写上List<Employee>，这样避免破坏部门实体类的完整性，如:
```
public class BusinessDto extends Business {
    /** 与商户与负责人关系一对一 */
    private BusinessUserRelation businessUserRelation;
    /** 与商户账户一对一 */
    private BusinessAccount businessAccount;
    /** 与商户图片一对多 */
    private List<BusinessImg> imgList;

    public List<BusinessImg> getImgList() {
        return imgList;
    }
    public void setImgList(List<BusinessImg> imgList) {
        this.imgList = imgList;
    }
    public BusinessUserRelation getBusinessUserRelation() {
        return businessUserRelation;
    }
    public void setBusinessUserRelation(BusinessUserRelation businessUserRelation) {
        this.businessUserRelation = businessUserRelation;
    }
    public BusinessAccount getBusinessAccount() {
        return businessAccount;
    }
    public void setBusinessAccount(BusinessAccount businessAccount) {
        this.businessAccount = businessAccount;
    }
}
```
这样接口需要什么样的数据，我们就可以组装什么样的dto，灰常的清晰。

顺便记录一下分页接口的写法
# Controller
```
**
     * @author: wjy
     * @description: 运营合伙人-爆品订单列表
     */
    @ResponseBody
    @PostMapping("/tubitu/goodsOrder/explodeOrderList")
    public ResponseResultUtil explodeOrderList(@RequestBody Map<String,String> paramMap){
        LOGGER.info(
                "\n***************************************" + "\n" +
                        "start explodeOrderList" + "\n" +
                        "运营合伙人-爆品订单列表" + "\n" +
                        "paramMap = " + paramMap + "\n" +
                        "\n********************************************"
        );
        ResponseResultUtil result = ResponseResultUtil.responseSuccess();
        try {
            Token token = TokenUtil.getToken(request.getHeader("token"));
            Assert.isTrue(token != null, "token解密失败");
            Assert.isTrue( paramMap.get("pageNum") != null, "页码不可为空");
            Assert.isTrue( paramMap.get("pageSize") != null, "步长不可为空");

            result = goodsOrderService.explodeOrderList(paramMap,result,token.getId());
        } catch (Exception e) {
            e.printStackTrace();
            result.setCode(ErrorEnum.ERROR_200000.getCode());
            result.setMessage(ErrorEnum.ERROR_200000.getMessage());
        }
        return result;
    }
```
```
需要注意的地方：
1.接口一律采用postMapping
2.接收参数，使用@RequestBody，返回参数使用@ResponseBody，json收发参数
3.进方法logger参数列表，方便线上时调试。
4.通过code码和message来向前端反馈信息，
例如：10000代表成功，message为“请求成功”，
20000代表失败，message为“请求失败”，
其他自行定义。
大部分方法都需要token，增加了接口的安全性，即必须登陆后，才可以请求数据，
5.不要再controller里写逻辑，controller只负责收发参数，跳转页面等，业务逻辑全部卸载接口的实现类中。
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
```
<resultMap id="ExplodeOrderMap" type="com.tubitu.model.apiDto.ExplodeOrderDto">
    <id column="id" property="id" jdbcType="INTEGER" />
    <result column="order_no" property="orderNo" jdbcType="VARCHAR" />
    <result column="consignee" property="consignee" jdbcType="VARCHAR" />
    <result column="tel" property="tel" jdbcType="VARCHAR" />
    <result column="address" property="address" jdbcType="VARCHAR" />
    <result column="status" property="status" jdbcType="INTEGER" />
    <result column="create_time" property="createTime" jdbcType="TIMESTAMP" />
  <!--一对一-->
    <association property="wxGroup" javaType="com.tubitu.model.apiDto.ExplodeOrderDto$WxGroup">
      <id column="groupId" property="groupId" jdbcType="INTEGER" />
      <result column="group_name" property="groupName" />
      <!--这里写的没错，是mybatis插件的bug-->
     <!--一对一对一-->
      <association property="store" javaType="com.tubitu.model.apiDto.ExplodeOrderDto$InnerStore">
        <id column="storeId" property="storeId" jdbcType="INTEGER" />
        <result column="store_name" property="storeName"/>
        <result column="contacts" property="contacts"/>
        <result column="mobile" property="mobile"/>
      </association>
    </association>
   <!--一对多-->
    <collection property="detailList" ofType="com.tubitu.model.apiDto.ExplodeOrderDto$DetailInnerDto">
      <id column="detailId" property="detailId" jdbcType="INTEGER" />
      <result column="goods_name" property="goodsName"/>
      <result column="goods_img" property="goodsImg"/>
      <result column="goods_price" property="goodsPrice"/>
      <result column="goods_num" property="goodsNum"/>
    </collection>
  </resultMap>
```
xml中的核心时resultMap的使用。
```
public class ExplodeOrderDto implements Serializable {

    /** 主键*/
    private Integer id;

    /** 收货人*/
    private String consignee;

    /** 收货人联系电话*/
    private String tel;

    /** 收货地址*/
    private String address;

    /** 订单状态*/
    private Integer status;

    /** 订单创建时间 */
    private Date createTime;

    /** 订单编号 */
    private String orderNo;

    /** 订单详情一对多(内部类在最下边) */
    List<DetailInnerDto> detailList;

    private WxGroup wxGroup;

    public WxGroup getWxGroup() {
        return wxGroup;
    }

    public void setWxGroup(WxGroup wxGroup) {
        this.wxGroup = wxGroup;
    }

    public String getOrderNo() {
        return orderNo;
    }

    public void setOrderNo(String orderNo) {
        this.orderNo = orderNo;
    }

    public Date getCreateTime() {
        return createTime;
    }

    public void setCreateTime(Date createTime) {
        this.createTime = createTime;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getConsignee() {
        return consignee;
    }

    public void setConsignee(String consignee) {
        this.consignee = consignee;
    }

    public String getTel() {
        return tel;
    }

    public void setTel(String tel) {
        this.tel = tel;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public Integer getStatus() {
        return status;
    }

    public void setStatus(Integer status) {
        this.status = status;
    }

    public List<DetailInnerDto> getDetailList() {
        return detailList;
    }

    public void setDetailList(List<DetailInnerDto> detailList) {
        this.detailList = detailList;
    }

    //微信群一对一内部类
    static class WxGroup implements  Serializable{
        //主键
        private Integer groupId;
        //群名
        private String groupName;

        private InnerStore store;

        public InnerStore getStore() {
            return store;
        }

        public void setStore(InnerStore store) {
            this.store = store;
        }

        public Integer getGroupId() {
            return groupId;
        }

        public void setGroupId(Integer groupId) {
            this.groupId = groupId;
        }

        public String getGroupName() {
            return groupName;
        }

        public void setGroupName(String groupName) {
            this.groupName = groupName;
        }

    }

    //微信群一对一商超(内部类)
    static class InnerStore implements Serializable{
        private Integer storeId;
        //商店名
        private String storeName;
        //联系人
        private String contacts;
        //联系电话
        private String mobile;

        public Integer getId() {
            return storeId;
        }

        public void setId(Integer storeId) {
            this.storeId = storeId;
        }

        public String getStoreName() {
            return storeName;
        }

        public void setStoreName(String storeName) {
            this.storeName = storeName;
        }

        public String getContacts() {
            return contacts;
        }

        public void setContacts(String contacts) {
            this.contacts = contacts;
        }

        public String getMobile() {
            return mobile;
        }

        public void setMobile(String mobile) {
            this.mobile = mobile;
        }
    }

    //订单详情一对多内部类
    static class DetailInnerDto implements Serializable{

        //主键
        private Integer detailId;

        //商品价格
        private BigDecimal goodsPrice;

        //商品名字
        private String goodsName;

        //商品图片
        private String goodsImg;

        //购买数量
        private Integer goodsNum;

        public Integer getDetailId() {
            return detailId;
        }

        public void setDetailId(Integer detailId) {
            this.detailId = detailId;
        }

        public BigDecimal getGoodsPrice() {
            return goodsPrice;
        }

        public void setGoodsPrice(BigDecimal goodsPrice) {
            this.goodsPrice = goodsPrice;
        }

        public String getGoodsName() {
            return goodsName;
        }

        public void setGoodsName(String goodsName) {
            this.goodsName = goodsName;
        }

        public String getGoodsImg() {
            return goodsImg;
        }

        public void setGoodsImg(String goodsImg) {
            this.goodsImg = goodsImg;
        }

        public Integer getGoodsNum() {
            return goodsNum;
        }

        public void setGoodsNum(Integer goodsNum) {
            this.goodsNum = goodsNum;
        }

    }

}
```
以及上边的dto。
1.一对多映射，必须使用实体类返回，使用map不行(据我所知)
2.如果只返回一个实体类中的数据，或者是一些零散的数据，完全可以返回map
3.dto中，只写前端需要的字段。一对一，或一对多时，可以使用内部类的形式，规定需要的字段。

最后附上ResponseResultUtil<T>，即controller返回的内容，封装。
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
