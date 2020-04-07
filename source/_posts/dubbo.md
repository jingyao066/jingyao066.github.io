---
title: dubbo
tags: java
date: 2019-04-17 11:05:53
---

[dubbo官网](http://dubbo.apache.org/zh-cn/index.html)

# dubbo控制台
[dubbo admin官网介绍](http://dubbo.apache.org/zh-cn/blog/dubbo-admin.html)
[github地址](https://github.com/apache/dubbo-admin)
- 下载代码:`git clone https://github.com/apache/dubbo-admin.git`
- 在dubbo-admin-server/src/main/resources/application.properties中指定注册中心地址
- 构建`mvn clean package`
- 启动
`mvn --projects dubbo-admin-server spring-boot:run`
或
`cd dubbo-admin-distribution/target; java -jar dubbo-admin-0.1.jar`
- 访问
`http://localhost:8080`

修改访问端口：
`vim dubbo-admin\dubbo-admin-server\src\main\resources\application.properties`
在任意位置添加：
`server.port=9000`

目前新版buddo-admin还没有用户名密码访问功能。\
## 老版控制台
[github](https://github.com/apache/dubbo/tree/dubbo-2.6.0)
将项目整体下载，解压并进入到`dubbo-admin`
打包`mvn  package -Dmaven.skip.test=true`
进入target，找到打好的war包，复制到tomcat的webapps下，并更名为`ROOT.war`
启动tomcat，启动完了再停止(这一步是要通过tomcat将war包解开。)
进入到`webapps\dubbo-admin-2.6.0\WEB-INF`，修改`dubbo.properties`
```
dubbo.registry.address=zookeeper://127.0.0.1:2181
dubbo.admin.root.password=root
dubbo.admin.guest.password=guest
```
修改你的zookeeper地址。
修改你的用户名和密码，需要注意用户名在=号前边，等号后边的是密码。
然后再启动tomcat，访问即可。

[参考](https://blog.csdn.net/qq_28988969/article/details/79866111)
# 超时与重试
在provider和consumer端，都提供了超时(timeout)和重试(retries)的参数配置。
timeout默认值为1000，单位毫秒，表示超时时间是1秒；
retries默认值为2，表示重试2次，加上本身调用1次，一共有3次调用。
在org.apache.dubbo.common.Constants类可以找到：
`public static final int DEFAULT_TIMEOUT = 1000;`
`public static final int DEFAULT_RETRIES = 2;`

consumer端配置优先于provider端配置。

总结：
case 1： 默认情况，provider和consumer端的timeout和retires均不设置，使用默认值
在不设置timeout和retries的时候，如果provider端接口一直出现超时，provider端会调用3次，而日志中没有任何警告或错误信息；
consumer端虽然重试了2次，加本身调用的1次，一共发起3次调用，如果provider3次全部超时，consumer端会打印超时异常信息；
provider端日志中没有任何警告或错误信息不利于发现问题；前面的调用并未终止，如果是非查询类接口且接口没有实现幂等性时，可能产生重复数据。

case 2： provider端retries=0，consumer端不设置
在provider端设置retries=0已经生效，接口仅调用了1次。

case 3： provider端retries=0，consumer端retries=1
consumer的retries优先级较高，两端都设置的情况下，以consumer端的retries为准。

case 4： provider端timeout=3000, retries=0，consumer端retries=0
provider端设置timeout已生效。

case 5： provider端timeout=1000, retries=0，consumer端retries=0
provider端设置了timeout，如果接口调用超时，provider会打印WRAN信息。

case 6： provider端timeout=1000, retries=0，consumer端timeout=3000, retries=0
consumer的timeout优先级较高，两端都设置的情况下，以consumer端的timeout为准。

参考dubbo官网的文档，并结合工作中项目实践，对超时和重试这2个参数做个总结：

- 超时(timeout)默认1000毫秒，重试(retries)默认2次(即一共调用3次)；
- provider端在<dubbo:service>中配置，consumer端在<dubbo:reference>中配置，consumer端的配置会覆盖provider配置；
- 超时(timeout)建议在provider端配置，因为作为提供方，它更清楚自己接口的耗时情况，并且provider端设置了timeout，在日志中有TimeoutFilter的WARN信息；
- 在provider端一般接口timeout设置为5秒或者10秒，如果是复杂查询、导出报表、调用第三方接口、本身是最上游的接口等，根据情况考虑设置大一点；
- 在consumer端配置设置timeout会覆盖provider设置，但有时设置timeout能够让consumer快速失败，而不因为下游provider服务接口的问题拖垮consumer本身；
- retries建议在provider端设置为0，consumer根据情况也可以设置为0，因为重试可能因非幂等性原因导致重复数据，并且超时情况即便重试成功consumer端可能也收不到成功响应；

[参考](https://www.cnblogs.com/cdfive2018/p/10204720.html)
[参考2](https://blog.csdn.net/u013940812/article/details/80041669)

# dubbo报错
## com.alibaba.com.caucho.hessian.io.HessianProtocolException: 'xxx' could not be instantiated
Hessian反序列化时，使用反射调用构造函数生成对象时，传入的参数不合法，造成了上面的异常。
知道了原因，解决的方法也很简单，就是添加了一个无参的构造器给CountObject，于是上面的问题就解决了。
```
public ResponseUtil() {

}
```
https://hittyt.iteye.com/blog/1691772

## Tried 3 times of the providers
意思是服务端被重复调用三次，然而provider并没有报错。
我觉得是provider的响应时间问题，就是说consumer等待provider默认是有时间限制的(默认1000ms)，即如果1秒内provider没有响应，就会再次调用。
如果三次调用都在1秒内没有响应，就是调用失败，会报上边的错误。这时加上timeout配置即可：
```
@Reference(version = "1.0.0",timeout = 60000)
private GrowthService growthService;
```