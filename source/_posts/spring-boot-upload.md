---
title: spring_boot_upload
tags: other
date: 2019-09-22 17:27:28
---

# 创建一个简单的包含WEB依赖的SpringBoot项目
```xml
<!-- Spring Boot web启动器 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- jsp -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
    <!--<scope>provided</scope>-->
</dependency>
```

# 配置文件上传的文件大小限制
```
# 上传文件总的最大值
spring.servlet.multipart.max-request-size=10MB
# 单个文件的最大值
spring.servlet.multipart.max-file-size=10MB

## jsp
spring.mvc.view.prefix=/WEB-INF/jsp/
spring.mvc.view.suffix=.jsp

#配置外部访问文件（把上传的图片视频文件放到E盘下的fileUpload文件夹下，linux需要更改为linux路径）
cbs.imagesPath=file:/E:/fileUpload/
```

# 单文件上传示例
```java
package com.songguoliang.springboot.controller;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.web.multipart.MultipartHttpServletRequest;

import javax.servlet.http.HttpServletRequest;
import java.io.File;
import java.io.IOException;
import java.util.List;

/**
 * @Description
 * @Author sgl
 * @Date 2018-05-15 14:04
 */
@Controller
public class UploadController {
    private static final Logger LOGGER = LoggerFactory.getLogger(UploadController.class);

    @GetMapping("/upload")
    public String upload() {
        return "upload";
    }

    @PostMapping("/upload")
    @ResponseBody
    public String upload(@RequestParam("file") MultipartFile file) {
        if (file.isEmpty()) {
            return "上传失败，请选择文件";
        }

        String fileName = file.getOriginalFilename();
		//String filePath = "D:/fileUpload/";   这是windows的路径
        String filePath = "/Users/itinypocket/workspace/temp/";
        File dest = new File(filePath + fileName);
        try {
            file.transferTo(dest);
            LOGGER.info("上传成功");
            return "上传成功";
        } catch (IOException e) {
            LOGGER.error(e.toString(), e);
        }
        return "上传失败！";
    }

    
}
```

## 创建upload.jsp文件
只有一个表单，选择文件，form的enctype为multipart/form-data:
```html
<%@ page contentType="text/html;charset=UTF-8" pageEncoding="UTF-8" %>
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-type" content="text/html; charset=UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1"/>
    <title>单文件上传</title>
</head>
<body>
<form method="post" action="/upload" enctype="multipart/form-data">
    <input type="file" name="file"><br>
    <input type="submit" value="提交">
</form>
</body>
</html>
```

## 通过springboot插件启动项目
浏览器输入`http://localhost:8080/upload`跳转到刚写的jsp页面。
选择文件点击提交按钮返回成功信息，我们上传的文件保存在/Users/itinypocket/workspace/temp路径下，然后上传成功。

# 多文件上传
创建多文件上传的jsp页面，多文件上传页面只是比单文件上传多了file选择的input而已，multiUpload.jsp内容如下：
```html
<%@ page contentType="text/html;charset=UTF-8" pageEncoding="UTF-8" %>
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-type" content="text/html; charset=UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1"/>
    <title>多文件上传</title>
</head>
<body>
<form method="post" action="/multiUpload" enctype="multipart/form-data">
    <input type="file" name="file"><br>
    <input type="file" name="file"><br>
    <input type="file" name="file"><br>
    <input type="submit" value="提交">
</form>
</body>
</html>
```

## 在UploadController里添加多文件上传的方法
```java
@GetMapping("/multiUpload")
public String multiUpload() {
    return "multiUpload";
}

@ResponseBody
@PostMapping("/multiUpload")
public String multiUpload(MultipartFile[] files) {
	//String filePath = "D:/fileUpload/";   这是windows的路径
    String filePath = "/Users/itinypocket/workspace/temp/";
    for (int i = 0; i < files.size(); i++) {
        MultipartFile file = files.get(i);
        if (file.isEmpty()) {
            return "上传第" + (i++) + "个文件失败";
        }
        String fileName = file.getOriginalFilename();

        File dest = new File(filePath + fileName);
		System.out.println( "文件大小是：" +files[i].getSize() + "byte");
        try {
            file.transferTo(dest);
            LOGGER.info("第" + (i + 1) + "个文件上传成功");
        } catch (IOException e) {
            LOGGER.error(e.toString(), e);
            return "上传第" + (i++) + "个文件失败";
        }
    }
    return "上传成功";
}
```
重启服务，浏览器输入`http://localhost:8080/multiUpload`然后选择要上传的文件，点击提交按钮，得到成功信息。
我们选择的三个文件已被成功上传到/Users/itinypocket/workspace/temp路径下。

多文件上传的方法只是最基础的版本，没有加入文件存在判断，文件夹不存在判断等，可以根据自己的需求改造。
其实只需要一个多文件上传就足够了，毕竟多文件包含了单文件，另外上面的多文件上传方法还可以当作文件上传接口使用，下面就用postman来测试这个接口。

`file.getSize()`方法可以直接获取到文件的大小， 单位是字节，可以转换成自己想要的单位。
下面转换成mb，保留两位小数：
`new BigDecimal(files[i].getSize()).divide(BigDecimal.valueOf(1024*1024),2,BigDecimal.ROUND_HALF_UP)`

配置外部访问服务器资源的。不配置的话，访问不了图片或视频等文件。
```java
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.web.servlet.MultipartConfigFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.*;
 
import javax.servlet.MultipartConfigElement;
 
//上传配置类
//图片放到/F:/fileUpload/后，从磁盘读取的图片数据scr将会变成images/picturename.jpg的格式
@Configuration
public class WebAppConfig extends WebMvcConfigurerAdapter {
//public class WebAppConfig extends WebMvcConfigurationSupport {
    /**
     * 在配置文件中配置的文件保存路径
     */
    @Value("${cbs.imagesPath}")
    private String mImagesPath;
 
    @Bean
    public MultipartConfigElement multipartConfigElement(){
        MultipartConfigFactory factory = new MultipartConfigFactory();
        //文件最大KB,MB
        factory.setMaxFileSize("1024MB");
        //设置总上传数据总大小
        factory.setMaxRequestSize("1024MB");
        return factory.createMultipartConfig();
    }
 
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        if(mImagesPath.equals("") || mImagesPath.equals("${cbs.imagesPath}")){
            String imagesPath = WebAppConfig.class.getClassLoader().getResource("").getPath();
            System.out.print("1.上传配置类imagesPath=="+imagesPath+"\n");
            if(imagesPath.indexOf(".jar")>0){
                imagesPath = imagesPath.substring(0, imagesPath.indexOf(".jar"));
            }else if(imagesPath.indexOf("classes")>0){
                imagesPath = "file:"+imagesPath.substring(0, imagesPath.indexOf("classes"));
            }
            imagesPath = imagesPath.substring(0, imagesPath.lastIndexOf("/"))+"/images/";
            mImagesPath = imagesPath;
        }
        System.out.print("imagesPath============="+mImagesPath+"\n");
        //LoggerFactory.getLogger(WebAppConfig.class).info("imagesPath============="+mImagesPath+"\n");
        registry.addResourceHandler("/images/**").addResourceLocations(mImagesPath);
        // TODO Auto-generated method stub
        System.out.print("2.上传配置类mImagesPath=="+mImagesPath+"\n");
        super.addResourceHandlers(registry);
    }
 
}
```

[参考地址1](https://blog.csdn.net/gnail_oug/article/details/80324120)
[参考地址2](https://blog.csdn.net/qq_37164847/article/details/81668242)

# postMan测试文件上传
- 选择post请求方式，输入请求地址

- 填写Headers
Key：Content-Type
Value：multipart/form-data

- 填写body
选择form-data，然后鼠标悬浮在输入框的末尾，默认Text，这里我们选择File。

注意：form-data的key值，一定要和controller参数的名字一致。