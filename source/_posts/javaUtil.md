---
title: javaUtil
tags: java
date: 2019-07-02 20:39:07
---

# 时间
## 时间戳
- 判断两个时间戳是否是同一天
```java
public static boolean isSameDay(int beginTimestamp,int endTimestamp) {
	SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
	String ds1 = sdf.format(beginTimestamp * 1000l);
	String ds2 = sdf.format(endTimestamp * 1000l);
	if (ds1.equals(ds2)) {
		return true;
	} else {
		return false;
	}
}
```

- 计算两个时间戳差值(秒级时间戳)
```java
public static int getTimeDifference(int beginTimestamp, int endTimestamp, int type) {
	int result = 0;
	if (beginTimestamp == 0 || endTimestamp == 0) {
		return 0;
	}
	try {
		int millisecond = endTimestamp - beginTimestamp;
		/**
		 * Math.abs((int)(millisecond/1000)); 绝对值 1秒 = 1000毫秒
		 * millisecond/1000 --> 秒
		 * millisecond/1000*60 - > 分钟
		 * millisecond/(1000*60*60) -- > 小时
		 * millisecond/(1000*60*60*24) --> 天
		 * */
		switch (type) {
			case 0: // second
				return  millisecond;
			case 1: // minute
				return (millisecond / 60);
			case 2: // hour
				return  (millisecond / (60 * 60));
			case 3: // day
				return (millisecond / (60 * 60 * 24));
		}
	} catch (Exception e) {
		e.printStackTrace();
	}
	return result;
}
```