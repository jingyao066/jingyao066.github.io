---
title: mybatis
tags: mybatis
date: 2019-03-27 15:07:56
---

# 基础
## 面向接口编程
1. `Mapper`文件的`namespace`要和`dao层接口`的完整路径对应
`<mapper namespace="com.wjy.mapper.TestMapper">`

2. 接口中的方法名字要和Mapper文件中SQL语句的ID对应
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
4. 接口中方法的参数要和Mapper文件中的parameterType对应

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
<resultMap id="selectAllMap" type="DeptBean" autoMapping="true">
    <id column="id" property="id" /> -- 就算使用了autoMapping，还是需要映射该字段，该字段必须单独映射，否则报错
    <result column="dept_name" property="deptName" />
    <collection property="empList" ofType="com.zjx.model.Emp" autoMapping="true">
        <!-- 
            一对多使用collection
            property：实体类中list属性的名字(必须)
            ofType：list中的对象类型List<EmpBean>(必须)
            若使用自动映射automapping，需要为使用as别名的字段单独使用result标签
         -->
		 -- 若返回结果需要id，该字段必须映射，否则会把主表的id映射到每一条子数据上，造成子数据每条id都一样的错误
		 -- 而且查询的id必须起别名，不能和主表的id一样(主表的主键叫id，子表的主键也叫id，会造成映射错误)
        <id column="e_id" property="id" /> -- 此处的cloumn必须改为和id的别名一致，否则会造成映射错误
        <result column = "emp_name" property = "empName" />
    </collection>
</resultMap>
```
注：一对多关系中，`一`方(主表)的`id`和`多`方(子表)的`id`，其中：
1. `一方的id(主表)`必须被查询和成功的映射(自动映射autoMapping不可以，必须单独映射)。
2. `多方的id(子表)`必须被查询和成功的映射

若不符合上方条件，会造成结果集的关系匹配不正确。

resultMap的用法以及关联结果集映射：
https://blog.csdn.net/qq_42780864/article/details/81429114

## 批量插入
批量插入大致有两种形式
1. 复杂参数类型，传入参数为map
`void bulkInsertImg(@Param("imgMap") Map<String,Object> map);`

这里我们可以只传递一个map参数，也可以传入多个参数：
`void insertBatch(@Param("skuList") List<GoodsSku> skuList,@Param("goodsId") Integer goodsId);`

```xml
<!--批量插入商品图片-->
<insert id="bulkInsertImg">
INSERT INTO
goods_img
(
  goods_id,
  goods_img
)
VALUES
<foreach collection="imgMap.imgArr" separator="," item="i"  index="idx">
  (
  #{imgMap.goodsId},
  #{i}
  )
</foreach>
</insert>
```

需要注意foreach循环数组和list是有区别的：
```xml
<!-- 批量插入商品规格 -->
<insert id="bulkInsert">
INSERT INTO
  goods_spec(
	goods_id,
	spec_name,
	spec_val)
VALUES
<foreach collection="specMap.specName" separator="," item="i"  index="idx">
  (
	#{specMap.goodsId},
	#{i},
	${specMap.specVal[idx]}
  )
</foreach>
</insert>
```


# MyBatis传入参数
## 单个参数
### 名字一致
`public List<XXBean> getXXBeanList(String xxCode);`
```xml
<select id="getXXXBeanList" parameterType="java.lang.String" resultType="XXBean">
　　select t.* from tableName t where t.id= #{id}
</select>
```
其中方法名和ID一致，#{}中的参数名与方法中的参数名一直， 我这里采用的是XXXBean是采用的短名字,
select 后的字段列表要和bean中的属性名一致， 如果不一致的可以用as来补充。

### #{_parameter}
只传入一个参数，可以使用
`#{_parameter}`
当作占位符。

### {arg0} 与 {0}
mybatis3.4.2或者之后的版本，使用`#{0}`就会产生绑定异常：
`org.apache.ibatis.binding.BindingException: Parameter '0' not found. Available parameters are [arg1, arg0, param1, param2]`

从异常可以看出在没有使用@参数注解的情况下，传递参数需要使用：
`#{arg0},#{argN}`或`#{param1},#{paramN}`

3.4.1以及之前版本的绑定异常信息：
`org.apache.ibatis.binding.BindingException: Parameter 'arg0' not found. Available parameters are [0, 1, param1, param2]`
可以发现3.4.1版本中传递参数可以使用#{0} - #{N}

原因：
MyBatis的通过XMLConfigBuilder类来加载配置文件中的各项参数

3.4.2版本之前设置属性中useActualParamName参数的默认值为flase 
XMLConfigBuilder.class的settingsElement方法中的源代码：
`configuration.setUseActualParamName(booleanValueOf(props.getProperty("useActualParamName"), true));`

3.4.2版本之后设置属性中useActualParamName参数的默认值为true 
XMLConfigBuilder.class的settingsElement方法中的源代码
`configuration.setUseActualParamName(booleanValueOf(props.getProperty("useActualParamName"), false));`

虽然`#{arg0}`和`#{param1}`可以混用，如
`where id = #{arg0} and user_id = #{param2}`
但是最好别这么用，容易让人懵逼。

arg参数从0开始，param从1开始，注意。

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

问题：
mysql有字段是text类型，使用generator生成xml后，多生成一个sql片段：
```xml
<sql id="Blob_Column_List" >
    content
</sql>
```
同时resultMap也会有变化：
```xml
<resultMap id="ResultMapWithBLOBs" type="com.zjx.model.Question" extends="BaseResultMap" >
    <result column="content" property="content" jdbcType="LONGVARCHAR" />
</resultMap>
```
`jdbcType="LONGVARCHAR"`
update语句也会多出一个`updateByPrimaryKeyWithBLOBs`方法。

可以在generator中通过加入下面的代码避免上面的情况：
```
<!-- 有些表的字段需要指定java类型
<table schema="" tableName="">
	<columnOverride column="content" javaType="java.lang.String" jdbcType="VARCHAR"/>
</table> -->
```

但是，不建议这样修改，数据库字段设置成text，就是想多承载数据，生成的mapper数据类型是varchar 那么承装的数据量不就没有longvarchar高了。
如果这样的话还不如直接写个String 就不用text属性了，是不是这个理？

## 完整的配置说明
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
"http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<!-- 配置生成器 -->
<generatorConfiguration>
<!-- 可以用于加载配置项或者配置文件，在整个配置文件中就可以使用${propertyKey}的方式来引用配置项
    resource：配置资源加载地址，使用resource，MBG从classpath开始找，比如com/myproject/generatorConfig.properties        
    url：配置资源加载地质，使用URL的方式，比如file:///C:/myfolder/generatorConfig.properties.
    注意，两个属性只能选址一个;

    另外，如果使用了mybatis-generator-maven-plugin，那么在pom.xml中定义的properties都可以直接在generatorConfig.xml中使用
<properties resource="" url="" />
 -->

 <!-- 在MBG工作的时候，需要额外加载的依赖包
     location属性指明加载jar/zip包的全路径
<classPathEntry location="/Program Files/IBM/SQLLIB/java/db2java.zip" />
  -->

<!-- 
    context:生成一组对象的环境 
    id:必选，上下文id，用于在生成错误时提示
    defaultModelType:指定生成对象的样式
        1，conditional：类似hierarchical；
        2，flat：所有内容（主键，blob）等全部生成在一个对象中；
        3，hierarchical：主键生成一个XXKey对象(key class)，Blob等单独生成一个对象，其他简单属性在一个对象中(record class)
    targetRuntime:
        1，MyBatis3：默认的值，生成基于MyBatis3.x以上版本的内容，包括XXXBySample；
        2，MyBatis3Simple：类似MyBatis3，只是不生成XXXBySample；
    introspectedColumnImpl：类全限定名，用于扩展MBG
-->
<context id="mysql" defaultModelType="hierarchical" targetRuntime="MyBatis3Simple" >

    <!-- 自动识别数据库关键字，默认false，如果设置为true，根据SqlReservedWords中定义的关键字列表；
        一般保留默认值，遇到数据库关键字（Java关键字），使用columnOverride覆盖
     -->
    <property name="autoDelimitKeywords" value="false"/>
    <!-- 生成的Java文件的编码 -->
    <property name="javaFileEncoding" value="UTF-8"/>
    <!-- 格式化java代码 -->
    <property name="javaFormatter" value="org.mybatis.generator.api.dom.DefaultJavaFormatter"/>
    <!-- 格式化XML代码 -->
    <property name="xmlFormatter" value="org.mybatis.generator.api.dom.DefaultXmlFormatter"/>

    <!-- beginningDelimiter和endingDelimiter：指明数据库的用于标记数据库对象名的符号，比如ORACLE就是双引号，MYSQL默认是`反引号； -->
    <property name="beginningDelimiter" value="`"/>
    <property name="endingDelimiter" value="`"/>

    <!-- 必须要有的，使用这个配置链接数据库
        @TODO:是否可以扩展
     -->
    <jdbcConnection driverClass="com.mysql.jdbc.Driver" connectionURL="jdbc:mysql:///pss" userId="root" password="admin">
        <!-- 这里面可以设置property属性，每一个property属性都设置到配置的Driver上 -->
    </jdbcConnection>

    <!-- java类型处理器 
        用于处理DB中的类型到Java中的类型，默认使用JavaTypeResolverDefaultImpl；
        注意一点，默认会先尝试使用Integer，Long，Short等来对应DECIMAL和 NUMERIC数据类型； 
    -->
    <javaTypeResolver type="org.mybatis.generator.internal.types.JavaTypeResolverDefaultImpl">
        <!-- 
            true：使用BigDecimal对应DECIMAL和 NUMERIC数据类型
            false：默认,
                scale>0;length>18：使用BigDecimal;
                scale=0;length[10,18]：使用Long；
                scale=0;length[5,9]：使用Integer；
                scale=0;length<5：使用Short；
         -->
        <property name="forceBigDecimals" value="false"/>
    </javaTypeResolver>


    <!-- java模型创建器，是必须要的元素
        负责：1，key类（见context的defaultModelType）；2，java类；3，查询类
        targetPackage：生成的类要放的包，真实的包受enableSubPackages属性控制；
        targetProject：目标项目，指定一个存在的目录下，生成的内容会放到指定目录中，如果目录不存在，MBG不会自动建目录
     -->
    <javaModelGenerator targetPackage="com._520it.mybatis.domain" targetProject="src/main/java">
        <!--  for MyBatis3/MyBatis3Simple
            自动为每一个生成的类创建一个构造方法，构造方法包含了所有的field；而不是使用setter；
         -->
        <property name="constructorBased" value="false"/>

        <!-- 在targetPackage的基础上，根据数据库的schema再生成一层package，最终生成的类放在这个package下，默认为false -->
        <property name="enableSubPackages" value="true"/>

        <!-- for MyBatis3 / MyBatis3Simple
            是否创建一个不可变的类，如果为true，
            那么MBG会创建一个没有setter方法的类，取而代之的是类似constructorBased的类
         -->
        <property name="immutable" value="false"/>

        <!-- 设置一个根对象，
            如果设置了这个根对象，那么生成的keyClass或者recordClass会继承这个类；在Table的rootClass属性中可以覆盖该选项
            注意：如果在key class或者record class中有root class相同的属性，MBG就不会重新生成这些属性了，包括：
                1，属性名相同，类型相同，有相同的getter/setter方法；
         -->
        <property name="rootClass" value="com._520it.mybatis.domain.BaseDomain"/>

        <!-- 设置是否在getter方法中，对String类型字段调用trim()方法 -->
        <property name="trimStrings" value="true"/>
    </javaModelGenerator>


    <!-- 生成SQL map的XML文件生成器，
        注意，在Mybatis3之后，我们可以使用mapper.xml文件+Mapper接口（或者不用mapper接口），
            或者只使用Mapper接口+Annotation，所以，如果 javaClientGenerator配置中配置了需要生成XML的话，这个元素就必须配置
        targetPackage/targetProject:同javaModelGenerator
     -->
    <sqlMapGenerator targetPackage="com._520it.mybatis.mapper" targetProject="src/main/resources">
        <!-- 在targetPackage的基础上，根据数据库的schema再生成一层package，最终生成的类放在这个package下，默认为false -->
        <property name="enableSubPackages" value="true"/>
    </sqlMapGenerator>


    <!-- 对于mybatis来说，即生成Mapper接口，注意，如果没有配置该元素，那么默认不会生成Mapper接口 
        targetPackage/targetProject:同javaModelGenerator
        type：选择怎么生成mapper接口（在MyBatis3/MyBatis3Simple下）：
            1，ANNOTATEDMAPPER：会生成使用Mapper接口+Annotation的方式创建（SQL生成在annotation中），不会生成对应的XML；
            2，MIXEDMAPPER：使用混合配置，会生成Mapper接口，并适当添加合适的Annotation，但是XML会生成在XML中；
            3，XMLMAPPER：会生成Mapper接口，接口完全依赖XML；
        注意，如果context是MyBatis3Simple：只支持ANNOTATEDMAPPER和XMLMAPPER
    -->
    <javaClientGenerator targetPackage="com._520it.mybatis.mapper" type="ANNOTATEDMAPPER" targetProject="src/main/java">
        <!-- 在targetPackage的基础上，根据数据库的schema再生成一层package，最终生成的类放在这个package下，默认为false -->
        <property name="enableSubPackages" value="true"/>

        <!-- 可以为所有生成的接口添加一个父接口，但是MBG只负责生成，不负责检查
        <property name="rootInterface" value=""/>
         -->
    </javaClientGenerator>

    <!-- 选择一个table来生成相关文件，可以有一个或多个table，必须要有table元素
        选择的table会生成一下文件：
        1，SQL map文件
        2，生成一个主键类；
        3，除了BLOB和主键的其他字段的类；
        4，包含BLOB的类；
        5，一个用户生成动态查询的条件类（selectByExample, deleteByExample），可选；
        6，Mapper接口（可选）

        tableName（必要）：要生成对象的表名；
        注意：大小写敏感问题。正常情况下，MBG会自动的去识别数据库标识符的大小写敏感度，在一般情况下，MBG会
            根据设置的schema，catalog或tablename去查询数据表，按照下面的流程：
            1，如果schema，catalog或tablename中有空格，那么设置的是什么格式，就精确的使用指定的大小写格式去查询；
            2，否则，如果数据库的标识符使用大写的，那么MBG自动把表名变成大写再查找；
            3，否则，如果数据库的标识符使用小写的，那么MBG自动把表名变成小写再查找；
            4，否则，使用指定的大小写格式查询；
        另外的，如果在创建表的时候，使用的""把数据库对象规定大小写，就算数据库标识符是使用的大写，在这种情况下也会使用给定的大小写来创建表名；
        这个时候，请设置delimitIdentifiers="true"即可保留大小写格式；

        可选：
        1，schema：数据库的schema；
        2，catalog：数据库的catalog；
        3，alias：为数据表设置的别名，如果设置了alias，那么生成的所有的SELECT SQL语句中，列名会变成：alias_actualColumnName
        4，domainObjectName：生成的domain类的名字，如果不设置，直接使用表名作为domain类的名字；可以设置为somepck.domainName，那么会自动把domainName类再放到somepck包里面；
        5，enableInsert（默认true）：指定是否生成insert语句；
        6，enableSelectByPrimaryKey（默认true）：指定是否生成按照主键查询对象的语句（就是getById或get）；
        7，enableSelectByExample（默认true）：MyBatis3Simple为false，指定是否生成动态查询语句；
        8，enableUpdateByPrimaryKey（默认true）：指定是否生成按照主键修改对象的语句（即update)；
        9，enableDeleteByPrimaryKey（默认true）：指定是否生成按照主键删除对象的语句（即delete）；
        10，enableDeleteByExample（默认true）：MyBatis3Simple为false，指定是否生成动态删除语句；
        11，enableCountByExample（默认true）：MyBatis3Simple为false，指定是否生成动态查询总条数语句（用于分页的总条数查询）；
        12，enableUpdateByExample（默认true）：MyBatis3Simple为false，指定是否生成动态修改语句（只修改对象中不为空的属性）；
        13，modelType：参考context元素的defaultModelType，相当于覆盖；
        14，delimitIdentifiers：参考tableName的解释，注意，默认的delimitIdentifiers是双引号，如果类似MYSQL这样的数据库，使用的是`（反引号，那么还需要设置context的beginningDelimiter和endingDelimiter属性）
        15，delimitAllColumns：设置是否所有生成的SQL中的列名都使用标识符引起来。默认为false，delimitIdentifiers参考context的属性

        注意，table里面很多参数都是对javaModelGenerator，context等元素的默认属性的一个复写；
     -->
    <table tableName="userinfo" >

        <!-- 参考 javaModelGenerator 的 constructorBased属性-->
        <property name="constructorBased" value="false"/>

        <!-- 默认为false，如果设置为true，在生成的SQL中，table名字不会加上catalog或schema； -->
        <property name="ignoreQualifiersAtRuntime" value="false"/>

        <!-- 参考 javaModelGenerator 的 immutable 属性 -->
        <property name="immutable" value="false"/>

        <!-- 指定是否只生成domain类，如果设置为true，只生成domain类，如果还配置了sqlMapGenerator，那么在mapper XML文件中，只生成resultMap元素 -->
        <property name="modelOnly" value="false"/>

        <!-- 参考 javaModelGenerator 的 rootClass 属性 
        <property name="rootClass" value=""/>
         -->

        <!-- 参考javaClientGenerator 的  rootInterface 属性
        <property name="rootInterface" value=""/>
        -->

        <!-- 如果设置了runtimeCatalog，那么在生成的SQL中，使用该指定的catalog，而不是table元素上的catalog 
        <property name="runtimeCatalog" value=""/>
        -->

        <!-- 如果设置了runtimeSchema，那么在生成的SQL中，使用该指定的schema，而不是table元素上的schema 
        <property name="runtimeSchema" value=""/>
        -->

        <!-- 如果设置了runtimeTableName，那么在生成的SQL中，使用该指定的tablename，而不是table元素上的tablename 
        <property name="runtimeTableName" value=""/>
        -->

        <!-- 注意，该属性只针对MyBatis3Simple有用；
            如果选择的runtime是MyBatis3Simple，那么会生成一个SelectAll方法，如果指定了selectAllOrderByClause，那么会在该SQL中添加指定的这个order条件；
         -->
        <property name="selectAllOrderByClause" value="age desc,username asc"/>

        <!-- 如果设置为true，生成的model类会直接使用column本身的名字，而不会再使用驼峰命名方法，比如BORN_DATE，生成的属性名字就是BORN_DATE,而不会是bornDate -->
        <property name="useActualColumnNames" value="false"/>


        <!-- generatedKey用于生成生成主键的方法，
            如果设置了该元素，MBG会在生成的<insert>元素中生成一条正确的<selectKey>元素，该元素可选
            column:主键的列名；
            sqlStatement：要生成的selectKey语句，有以下可选项：
                Cloudscape:相当于selectKey的SQL为： VALUES IDENTITY_VAL_LOCAL()
                DB2       :相当于selectKey的SQL为： VALUES IDENTITY_VAL_LOCAL()
                DB2_MF    :相当于selectKey的SQL为：SELECT IDENTITY_VAL_LOCAL() FROM SYSIBM.SYSDUMMY1
                Derby      :相当于selectKey的SQL为：VALUES IDENTITY_VAL_LOCAL()
                HSQLDB      :相当于selectKey的SQL为：CALL IDENTITY()
                Informix  :相当于selectKey的SQL为：select dbinfo('sqlca.sqlerrd1') from systables where tabid=1
                MySql      :相当于selectKey的SQL为：SELECT LAST_INSERT_ID()
                SqlServer :相当于selectKey的SQL为：SELECT SCOPE_IDENTITY()
                SYBASE      :相当于selectKey的SQL为：SELECT @@IDENTITY
                JDBC      :相当于在生成的insert元素上添加useGeneratedKeys="true"和keyProperty属性
        <generatedKey column="" sqlStatement=""/>
         -->

        <!-- 
            该元素会在根据表中列名计算对象属性名之前先重命名列名，非常适合用于表中的列都有公用的前缀字符串的时候，
            比如列名为：CUST_ID,CUST_NAME,CUST_EMAIL,CUST_ADDRESS等；
            那么就可以设置searchString为"^CUST_"，并使用空白替换，那么生成的Customer对象中的属性名称就不是
            custId,custName等，而是先被替换为ID,NAME,EMAIL,然后变成属性：id，name，email；

            注意，MBG是使用java.util.regex.Matcher.replaceAll来替换searchString和replaceString的，
            如果使用了columnOverride元素，该属性无效；

        <columnRenamingRule searchString="" replaceString=""/>
         -->


         <!-- 用来修改表中某个列的属性，MBG会使用修改后的列来生成domain的属性；
             column:要重新设置的列名；
             注意，一个table元素中可以有多个columnOverride元素哈~
          -->
         <columnOverride column="username">
             <!-- 使用property属性来指定列要生成的属性名称 -->
             <property name="property" value="userName"/>

             <!-- javaType用于指定生成的domain的属性类型，使用类型的全限定名
             <property name="javaType" value=""/>
              -->

             <!-- jdbcType用于指定该列的JDBC类型 
             <property name="jdbcType" value=""/>
              -->

             <!-- typeHandler 用于指定该列使用到的TypeHandler，如果要指定，配置类型处理器的全限定名
                 注意，mybatis中，不会生成到mybatis-config.xml中的typeHandler
                 只会生成类似：where id = #{id,jdbcType=BIGINT,typeHandler=com._520it.mybatis.MyTypeHandler}的参数描述
             <property name="jdbcType" value=""/>
             -->

             <!-- 参考table元素的delimitAllColumns配置，默认为false
             <property name="delimitedColumnName" value=""/>
              -->
         </columnOverride>

         <!-- ignoreColumn设置一个MGB忽略的列，如果设置了改列，那么在生成的domain中，生成的SQL中，都不会有该列出现 
             column:指定要忽略的列的名字；
             delimitedColumnName：参考table元素的delimitAllColumns配置，默认为false

             注意，一个table元素中可以有多个ignoreColumn元素
         <ignoreColumn column="deptId" delimitedColumnName=""/>
         -->
    </table>

</context>
</generatorConfiguration>
```

# 映射枚举类

# 简单的SQL注入原理
登录界面用户名输入：`or 1=1#`，密码随便输入，密码随便输，有可能进入系统。
通常我们的登录语句是这样的
`select * from users where username = '' and password = ''`
而#在mysql中是注释符，#号后面的内容将呗Mysql视为注释内容，上边的sql会变成这样：
`select * from users where username = '' or 1=1`

# 批量新增返回id
-  mybatis版本必须在3.3.1及以上
-  dao层方法返回值依旧是int
-  dao层接口如果只有一个参数，可以不用`@Param`注解，但是xml中`<foreach>`标签的`collection`参数名必须是`list`
-  dao层接口如果多个参数，需要用到`@Param`注解，value值必须是`list`，与`<foreach>`标签的`collection`参数名对应。
- xml中insert方法的参数类型`parameterType="list"`可写可不写。
- 和一条新增返回id一样，从传入的参数list中获取id

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

# Pagehelper没有效果
pagehelper只对紧跟着的第一个sql语句起作用，所以直接把`PageHelper.startPage(pageNum,pageSize)`放在需要分页的语句前边。

# MyBatis的xml判断
mybatis判断int类型时，不可以加`!= ''`非空判断，否则判断会失效，不会进入到判断中。

# Mybatis报错
```
The content of element type "resultMap" must match "(constructor?,id*,result*,association*,collection*,discriminator?)"
```
resultMap中标签的顺序需要和错误信息中标签出现的顺序一致，一对一映射(association)必须写在一对多映射(collection)前边。

# 数据库int类型被格式化为Long
使用map.get("type")获取，并用Integer强转时，报错，错误说明是long类型不能转为Integer。what？怎么会变成long类型了？
后来查了下，貌似mybatis偶尔会抽风，数据库里int类型的字段会时不时的被转成long或者int，所以强转就有问题了。
那怎么办呢，有个思路，是不管什么类型，统统转为string类型，然后再解析为Integer或者Long。所以可以使用以下方法：
Integer type = Integer.parseInt(map.get("type").toString());

# mybatis 多个条件 批量删除
```
<delete id="deleteList" parameterType="java.util.List">
	delete from table where
	<foreach collection="list"  item="item" separator=" or " index="index">
		(id= #{item.id}and itemcode= #{item.itemcode})
	</foreach>
</delete>
```

# mybatis+pageHelper，一对多分页
有三种解决方案：
- 使用collection的select标签
- 先在java代码中根据分页查出`一`的列表，然后将id放到一个数组中作为参数，把多条数据查出来，然后根据id循环匹配一对多条
- [自己实现分页插件 ](https://www.jianshu.com/p/1a0c48e8bfb7)
- group by id ????

# mybatis 日期时间格式化的问题
mysql 数据库 datetime类型字段，实体中类型为Date，映射出来的格式是:`2019-03-11T11:03:53.000+0000`
在实体类属性上加上该注解：
`@JsonFormat(timezone = "GMT+8",pattern = "yyyy-MM-dd HH:mm:ss")`

# @Autowired注入mapper文件报错
在mapper文件上加@Repository注解

# 通用mapper

## 问题
使用通用mapper的`updateByPrimaryKeySelective`方法时，`UPDATE role SET id = ?,name = ? WHERE id = ? AND name = ?`，发现where条件后，不仅有id，还有其他条件，导致更新失败。
解决：
原来是在实体类中的主键id字段没有加@id注解，导致找不到主键。