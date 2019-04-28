---
title: mybatis
tags: mybatis
date: 2019-03-27 15:07:56
---

# 基础
## 面向接口编程
1. `Mapper`文件的`namespace`要和接口的完整路径对应
`<mapper namespace="com.wjy.mapper.TestMapper">`

2. 接口中的方法名字 要和 Mapper文件中SQL语句的ID对应
dao层接口
`xxxBean selectByPrimaryKey(Integer id);`
```xml
<select id="selectByPrimaryKey" resultMap="BaseResultMap" parameterType="java.lang.Integer" >
    select
    <include refid="Base_Column_List" />
    from unit
    where id = #{id,jdbcType=INTEGER}
</select>
```

3. 接口中方法的返回值要和Mapper文件中的resultType对应
4. 接口中方法的参数 要和Mapper文件中的parameterType对应

## 简单的ResultMap
```xml
<!-- 
    type：返回类型  
    id：唯一标识 
-->
<resultMap type = "EmpBean" id = "selectAllMap">
    <!-- 
        id标签是主键
        column：sql查询结果的字段，需要和查询结果的表头字段一致，即如果用了as，需要和as后边的别名一致
        property：对应的java实体类属性
        javaType：property对应的数据类型
        jdbcType：column对应的数据类型
    -->
    <id column="id" property="id" />
    <!-- 普通字段 -->
    <result column="name" property="name" />
</resultMap>

<!-- 此处的resultMap需要和上边resultMap标签的id相对应 -->
<select id = "selectAll" resultMap="selectAllMap">
    select * from emp
</select>
```

## resultMap一对一
在`ResultMap`中使用`association`标签

EmpBean 实体类中存在一对一关系：
`private DeptBean deptBean`

定义sql语句：
```xml
<select id = "selectEmpResultMap" resultMap="selectEmpMap">
    select
        e.*,
        d.dept_name,
        d.id as dept_id
    from emp e
    left join dept d 
    on d.id = e.dept_id
</select>

<!-- 
    column为数据库字段，property为实体类的属性
    Bean模型属性与数据库字段不相同的，需要写一下result标签，其他的可以用自动映射automapping，也可以全部写上
 -->
<resultMap type = "EmpBean" id = "selectAllMap">
    <id column="id" property="id" />
    <result column="name" property="name" />
    <!-- 一对一关系 -->
    <association property = "deptBean" javaType = "DeptBean">
        <id column = "dept_id" property = "id" />
        <result column = "dept_name" property = "deptName" />
    </association>
</resultMap>
```

## resultMap一对多
在`resultMap`中使用`collection`标签
`DeptBean`实体类中存在一对多关系，站在部门的角度，部门和员工是一对多关系(一个部门对应多个员工)
`private List<Emp> empList`

编写sql
```xml
<select>
    select
        d.*,
        e.*,
        e.id as e_id
    from dept d
    left join emp e on e.dept_id = d.id
</select>
```
编写resultMap映射结果集
```xml
<resultMap id="selectAllMap" type="DeptBean">
    <id column="id" property="id" />
    <result column="dept_name" property="deptName" />
    <collection>
        <!-- 
            一对多使用collection
            property：实体类中list属性的名字
            ofType：list中的对象类型List<EmpBean>
            若使用自动映射automapping，需要为使用as别名的字段单独使用result标签
         -->
        <id column="id" property="id" />
        <result column = "emp_name" property = "empName" />
    </collection>
</resultMap>
```
注：一对多关系中，`一`方的`id`和`多`方的`dept_id`作为`left join`的连接关系，其中：
1. `一方的id`必须被查询和成功的映射。
2. `多方的dept_id`必须被成功的映射，不是必须查询
若不符合上方条件，会造成结果集的关系匹配不正确。

# MyBatis传入多个参数
## 单个参数
`public List<XXBean> getXXBeanList(String xxCode);`
```xml
<select id="getXXXBeanList" parameterType="java.lang.String" resultType="XXBean">
　　select t.* from tableName t where t.id= #{id}
</select>
```
其中方法名和ID一致，#{}中的参数名与方法中的参数名一直， 我这里采用的是XXXBean是采用的短名字,
select 后的字段列表要和bean中的属性名一致， 如果不一致的可以用as来补充。

## 多参数
`public List<XXXBean> getXXXBeanList(String xxId, String xxCode);`
```xml
<select id="getXXXBeanList" resultType="XXBean">
　　select t.* from tableName where id = #{0} and name = #{1}
</select>
```
由于是多参数那么就不能使用parameterType， 改用#｛index｝是第几个就用第几个的索引，索引从0开始

## Map封装多参数
`public List<XXXBean> getXXXBeanList(HashMap map);`
```xml
<select id="getXXXBeanList" parameterType="hashmap" resultType="XXBean">
　　select 字段... from XXX where id=#{xxId} code = #{xxCode}
</select>
```
其中hashmap是mybatis自己配置好的直接使用就行。map中key的名字是那个就在#{}使用那个，map如何封装就不用了我说了吧。

## List封装in
`public List<XXXBean> getXXXBeanList(List<String> list);`
```xml
<select id="getXXXBeanList" resultType="XXBean">
　　select 字段... from XXX where id in
　　<foreach item="item" index="index" collection="list" open="(" separator="," close=")">
　　　　#{item}
　　</foreach>
</select>
```
foreach 最后的效果是select 字段... from XXX where id in ('1','2','3','4')

## 多参数传递之注解方式示
例子：
 `public AddrInfo getAddrInfo(@Param("corpId")int corpId, @Param("addrId")int addrId);`
 xml配置这样写：
```xml
<select id="getAddrInfo"  resultMap="com.xxx.xxx.AddrInfo">
       SELECT * FROM addr__info 
　　　　where addr_id=#{addrId} and corp_id=#{corpId}
</select>
```
 以前在`<select>`语句中要带parameterType的，现在可以不要这样写。

## 复杂参数类型
`selectList()`只能传递一个参数，但实际所需参数既要包含String类型，又要包含List类型时的处理方法
将参数放入Map，再取出Map中的List遍历。如下：
```java
List<String> list_3 = new ArrayList<String>();
Map<String, Object> map2 = new HashMap<String, Object>();
list.add("1");
list.add("2");
map2.put("list", list); //网址id
map2.put("siteTag", "0");//网址类型
```
```java
public List<SysWeb> getSysInfo(Map<String, Object> map2) {
　　return getSqlSession().selectList("sysweb.getSysInfo", map2);
}
```
```xml
<select id="getSysInfo" parameterType="java.util.Map" resultType="SysWeb">
    select t.sysSiteId, t.siteName, t1.mzNum as siteTagNum, t1.mzName as siteTag, t.url, t.iconPath
    from TD_WEB_SYSSITE t
    left join TD_MZ_MZDY t1 on t1.mzNum = t.siteTag and t1.mzType = 10
    WHERE t.siteTag = #{siteTag } 
    and t.sysSiteId not in 
    <foreach collection="list" item="item" index="index" open="(" close=")" separator=",">
       #{item}
    </foreach>
</select>
```

# 实体类中有内部类
因为涉及到一对一和一对多，而且是内部类，需要注意写法：
```xml
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

# mybatis-Generator
推荐使用maven-plugin的方式，直接把配置文件放到项目中。
将下面的配置文件放到合适的位置。分布式项目可以放到common模块下，`src->main->resources`，路径下新建`MGB`包，包内放下面的xml配置文件。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <context id="test" targetRuntime="MyBatis3">
        <property name="javaFileEncoding" value="UTF-8"/>
        <!--generate entity时，生成hashcode和equals方法-->
        <plugin type="org.mybatis.generator.plugins.EqualsHashCodePlugin"></plugin>
        <!-- JavaBean 实现 序列化 接口 -->
        <plugin type="org.mybatis.generator.plugins.SerializablePlugin"></plugin>
        <!-- 生成toString -->
        <plugin type="org.mybatis.generator.plugins.ToStringPlugin"></plugin>
        <commentGenerator>
            <!-- 这个元素用来去除指定生成的注释中是否包含生成的日期 false:表示保护 -->
            <!-- 如果生成日期，会造成即使修改一个字段，整个实体类所有属性都会发生变化，不利于版本控制，所以设置为true -->
            <property name="suppressDate" value="true" />
            <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
            <property name="suppressAllComments" value="true" />
        </commentGenerator>
        <!--数据库链接URL，用户名、密码 -->
        <!--数据库链接URL，用户名、密码 -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/test?useUnicode=true&amp;characterEncoding=utf8&amp;allowMultiQueries=true&amp;autoReconnect=true&amp;tinyInt1isBit=false"
                        userId="root" password="root">

        <javaTypeResolver>
            <!-- This property is used to specify whether MyBatis Generator should
                force the use of java.math.BigDecimal for DECIMAL and NUMERIC fields, -->
            <!-- 默认false，把JDBC DECIMAL 和 NUMERIC 类型解析为 Integer，为 true时把JDBC DECIMAL 和
                NUMERIC 类型解析为java.math.BigDecimal -->
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>

        <!--去掉Mybatis Generator生成的一堆 example
		<table schema="general" tableName="tb_table_name" domainObjectName="EntityName"
			   enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false"
			   enableSelectByExample="false" selectByExampleQueryId="false" >
			<property name="useActualColumnNames" value="true"/>
		</table>
		-->
        <!-- 生成模型的包名和位置，注意这里使用相对路径，而且要使用左斜杠/ -->
        <javaModelGenerator targetPackage="com.zjx.model"
                            targetProject="src/main/java">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>
        <!-- 生成映射文件的包名和位置 -->
        <sqlMapGenerator targetPackage="mapper"
                         targetProject="src/main/resources/">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>
        <!-- 生成DAO的包名和位置 -->
        <javaClientGenerator type="XMLMAPPER"
                             targetPackage="com.zjx.mapper"   
                            targetProject="src/main/java">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>

        <!-- 要生成哪些表 -->
        <table tableName="unit" domainObjectName="unit"
               enableCountByExample="false" enableUpdateByExample="false"
               enableDeleteByExample="false" enableSelectByExample="false"
               selectByExampleQueryId="false"></table>

        <!-- 有些表的字段需要指定java类型
		 <table schema="" tableName="">
			<columnOverride column="" javaType="" />
		</table> -->

    </context>
</generatorConfiguration>
```
然后在模块的pom文件中加入如下plugin：
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-maven-plugin</artifactId>
            <version>1.3.2</version>
            <dependencies>
                <dependency>
                    <groupId>mysql</groupId>
                    <artifactId>mysql-connector-java</artifactId>
                    <version>5.1.38</version>
                </dependency>
            </dependencies>
            <configuration>
                <!--配置文件的路径-->
                <configurationFile>src/main/resources/MBG/generatorConfig.xml</configurationFile>
                <overwrite>true</overwrite>
            </configuration>
        </plugin>
    </plugins>
</build>
```
然后刷新maven

# 映射枚举类

# 简单的SQL注入
登录界面用户名输入：`or 1=1#`，密码随便输入，密码随便输，有可能进入系统。
通常我们的登录语句是这样的
`select * from users where username = '' and password = ''`
而#在mysql中是注释符，#号后面的内容将呗Mysql视为注释内容，上边的sql会变成这样：
`select * from users where username = '' or 1=1`


# 一些问题
- #{}和${}的区别是什么
	`#{}`是预编译处理，`${}`是字符串替换。
	mybatis在处理`#{}`时，会将sql中的`#{}`替换为?号，然后调用PreparedStatement的set方法来赋值。
	mybatis在处理`${}`时，就是把`${}`替换成变量的值。
	使用`#{}`可以有效的防止SQL注入，提高系统安全性。原因在于：预编译机制。预编译完成之后，SQL的结构已经固定，即便用户输入非法参数，也不会对SQL的结构产生影响，从而避免了潜在的安全风险。
	简单来说，#会给传入的参数加上引号，$会被直接替换成传入的参数。

- 最佳实践中，通常一个Xml映射文件，都会写一个Dao接口与之对应，请问，这个Dao接口的工作原理是什么？Dao接口里的方法，参数不同时，方法能重载吗？
Dao接口，就是人们常说的Mapper接口，接口的全名，就是映射文件中的namespace的值，接口的方法名，就是映射文件中MappedStatement的id值，接口方法内的参数，就是传递给sql的参数。
Mapper接口是没有实现类的，当调用接口方法时，接口全限名+方法名拼接字符串作为key值，可唯一定位一个MappedStatement，
举例：com.mybatis3.mappers.StudentDao.findStudentById，可以唯一找到namespace为com.mybatis3.mappers.StudentDao下面id = findStudentById的MappedStatement。
在Mybatis中，每一个`<select>、<insert>、<update>、<delete>`标签，都会被解析为一个MappedStatement对象。
Dao接口里的方法，是不能重载的，因为是全限名+方法名的保存和寻找策略。
Dao接口的工作原理是JDK动态代理，Mybatis运行时会使用JDK动态代理为Dao接口生成代理proxy对象，代理对象proxy会拦截接口方法，转而执行MappedStatement所代表的sql，然后将sql执行结果返回。


- Mybatis是如何将sql执行结果封装为目标对象并返回的？都有哪些映射形式？
第一种是使用<resultMap>标签，逐一定义列名和对象属性名之间的映射关系。
第二种是使用sql列的别名功能，将列别名书写为对象属性名，比如T_NAME AS NAME，对象属性名一般是name，小写，但是列名不区分大小写，Mybatis会忽略列名大小写，智能找到与之对应对象属性名，
你甚至可以写成T_NAME AS NaMe，Mybatis一样可以正常工作。有了列名与属性名的映射关系后，Mybatis通过反射创建对象，同时使用反射给对象的属性逐一赋值并返回，那些找不到映射关系的属性，是无法完成赋值的。

- Mybatis是否支持延迟加载？如果支持，它的实现原理是什么？
Mybatis仅支持association关联对象和collection关联集合对象的延迟加载，association指的就是一对一，collection指的就是一对多查询。
在Mybatis配置文件中，可以配置是否启用延迟加载lazyLoadingEnabled=true|false。
它的原理是，使用cglib创建目标对象的代理对象，当调用目标方法时，进入拦截器方法，比如调用a.getB().getName()，拦截器invoke()方法发现a.getB()是null值，
那么就会单独发送事先保存好的查询关联B对象的sql，把B查询上来，然后调用a.setB(b)，于是a的对象b属性就有值了，接着完成a.getB().getName()方法的调用。这就是延迟加载的基本原理。
当然了，不光是Mybatis，几乎所有的包括Hibernate，支持延迟加载的原理都是一样的。

- Mybatis的Xml映射文件中，不同的Xml映射文件，id是否可以重复？
不同的Xml映射文件，如果配置了namespace，那么id可以重复；如果没有配置namespace，那么id不能重复；毕竟namespace不是必须的，只是最佳实践而已。
原因就是namespace+id是作为Map<String, MappedStatement>的key使用的，如果没有namespace，就剩下id，那么，id重复会导致数据互相覆盖。
有了namespace，自然id就可以重复，namespace不同，namespace+id自然也就不同。

- Mybatis都有哪些Executor执行器？它们之间的区别是什么？
Mybatis有三种基本的Executor执行器，SimpleExecutor、ReuseExecutor、BatchExecutor。
SimpleExecutor：每执行一次update或select，就开启一个Statement对象，用完立刻关闭Statement对象。
ReuseExecutor：执行update或select，以sql作为key查找Statement对象，存在就使用，不存在就创建，
用完后，不关闭Statement对象，而是放置于Map<String, Statement>内，供下一次使用。简言之，就是重复使用Statement对象。
BatchExecutor：执行update（没有select，JDBC批处理不支持select），将所有sql都添加到批处理中（addBatch()），
等待统一执行（executeBatch()），它缓存了多个Statement对象，每个Statement对象都是addBatch()完毕后，等待逐一执行executeBatch()批处理。与JDBC批处理相同。
作用范围：Executor的这些特点，都严格限制在SqlSession生命周期范围内。
