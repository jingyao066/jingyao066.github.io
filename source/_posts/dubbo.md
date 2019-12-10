---
title: dubbo
tags: java
date: 2019-04-17 11:05:53
---

# dubbo控制台


# dubbo
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