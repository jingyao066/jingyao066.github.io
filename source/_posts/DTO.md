---
title: DTO
tags: java
date: 2018-12-09 00:44:08
---

# 前言
在写接口的时候，如果返回实体类，会返回一些前端并不需要的参数，例如：
- 学生和班级是一对一，我们会在学生类中，加入班级的实体类，构成一对一关系；
- 部门和员工是一对多，我们会在部门类中，加入List<Employee>，list泛型实体类，构成一对多关系。

这样我们其他的接口在用到学生或部门类的时候，就会返回我们不需要的空数据，为null，影响接口效率，整洁程度，还有可能泄露数据库表结构。

# 简单的DTO使用
在使用dto之后，我们新建dto类继承部门类，然后再写上List<Employee>，这样避免破坏部门实体类的完整性，如：
```
public class BusinessDto extends Business {
    //与商户账户一对一
    private BusinessAccount businessAccount;
    //与商户图片一对多
    private List<BusinessImg> imgList;
    ...省略get()、set()方法
}
```
这样接口需要什么样的数据，我们就可以组装什么样的dto，十分清晰。

# DTO深入使用
上边的简单使用虽然没有破坏原本的类，但是还是将一些不需要的属性返回给了接口。
因为我们使用BusinessDto继承了Business类，所以将Business的所有属性都继承了过来。
此处有两种解决方法：
1. `@JsonIgnore`
将该参数加在接口不需要的属性上，以json返回数据时，就会忽略该属性。使用该属性，需要在pom文件中加入如下坐标：
```
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <version>2.9.0</version>
</dependency>
```
2. 直接新建实体类(名字还是modelDto的形式，以区分该实体类和model类相关)，不继承，只填写需要的属性，这样虽然麻烦点，但是不会返回不需要的属性。

# 其他
另外，因为需要引入一对一或一对多，也会将引入的类不需要的属性返回，我们可以采用上面的两种方式。
还可以使用静态内部类，这样比较简洁，省的新建许多类。
内部类只写我们需要的属性，使用方式如下：
```
public class modelDto implements Serializable {
    /** 主键*/
    private Integer id;

    /** 名字*/
    private String name;

    /** 联系电话*/
    private String tel;

    /** 收货地址*/
    private String address;

    /** 详情一对多关系*/
    List<InnerDetailDto> detailList;

    /** 一对一关系 */
    private InnerOne innerOne;

    //微信群内部类
    static class InnerOne implements  Serializable{
        //主键
        private Integer innerId;
        //群名
        private String innerName;
    }

    //订单详情一对多内部类
    static class InnerDetailDto implements Serializable{
        //主键
        private Integer detailId;

        //商品价格
        private BigDecimal goodsPrice;

        //商品名字
        private String goodsName;

        //购买数量
        private Integer goodsNum;
    }

}
```
注意：
- 由于内部类中的属性是私有的，所以无法在外部访问。这样，如果我们使用ModelAndView向jsp或freemarker页面返回数据，页面使用诸如el表达式访问数据，会无法获取内部类的属性。所以这种dto，我们只在写接口时使用。

