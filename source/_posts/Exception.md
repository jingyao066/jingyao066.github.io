---
title: Exception
tags: java
date: 2019-04-17 11:05:53
---

# dubbo
## com.alibaba.com.caucho.hessian.io.HessianProtocolException: 'xxx' could not be instantiated
Hessian反序列化时，使用反射调用构造函数生成对象时，传入的参数不合法，造成了上面的异常。
知道了原因，解决的方法也很简单，就是添加了一个无参的构造器给CountObject，于是上面的问题就解决了。
```
public ResponseUtil() {

}
```
https://hittyt.iteye.com/blog/1691772

