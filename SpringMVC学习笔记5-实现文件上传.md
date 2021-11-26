---
title: SpringMVC学习笔记5:实现文件上传
date: 2020-03-05 16:41:10
tags:
- Java
- SpringMVC
categories:
- Java
- Spring
- SpringMVC
description: 实现文件上传。
---

## 1.文件上传回顾

- 导入文件上传的jar包。

```xml
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.1</version>
</dependency>
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.4</version>
</dependency>
```

- 编写文件上传的JSP页面。

```html
<h3>文件上传</h3>
<form action="user/fileupload" method="post" enctype="multipart/form-data">
    选择文件：<input type="file" name="upload"/><br/>
    <input type="submit" value="上传文件"/>
</form>
```

- 编写文件上传的Controller控制器。

```java
/**
* 文件上传
* @throws Exception
*/
@RequestMapping(value="/fileupload")
public String fileupload(HttpServletRequest request) throws Exception {
    // 先获取到要上传的文件目录
    String path = request.getSession().
        getServletContext().getRealPath("/uploads");
    // 创建File对象，一会向该路径下上传文件
    File file = new File(path);
    // 判断路径是否存在，如果不存在，创建该路径
    if(!file.exists()) {
    	file.mkdirs();
    }
    // 创建磁盘文件项工厂
    DiskFileItemFactory factory = new DiskFileItemFactory();
    ServletFileUpload fileUpload = new ServletFileUpload(factory);
    // 解析request对象
    List<FileItem> list = fileUpload.parseRequest(request);
    // 遍历
    for (FileItem fileItem : list) {
        // 判断文件项是普通字段，还是上传的文件
        if(fileItem.isFormField()) {
            
        }else {
            // 上传文件项
            // 获取到上传文件的名称
            String filename = fileItem.getName();
            // 上传文件
            fileItem.write(new File(file, filename));
            // 删除临时文件
            fileItem.delete();
    	}
    }
    return "success";
}
```

## 2.SpringMVC 传统方式文件上传

SpringMVC 框架提供了 **MultipartFile** 对象，该对象表示上传的文件，要求变量名称必须和表单 file 标签的 name 属性名称相同。

代码如下

```java
/**
* SpringMVC方式的文件上传
*
* @param request
* @return
* @throws Exception
*/
@RequestMapping(value="/fileupload2")
public String fileupload2(HttpServletRequest request,MultipartFile upload) throws
Exception {
    System.out.println("SpringMVC方式的文件上传...");
    // 先获取到要上传的文件目录
    String path = request.getSession().getServletContext().getRealPath("/uploads");
    // 创建File对象，一会向该路径下上传文件
    File file = new File(path);
    // 判断路径是否存在，如果不存在，创建该路径
    if(!file.exists()) {
    	file.mkdirs();
    }
    // 获取到上传文件的名称
    String filename = upload.getOriginalFilename();
    String uuid = UUID.randomUUID().toString().replaceAll("-", "").toUpperCase();
    // 把文件的名称唯一化
    filename = uuid+"_"+filename;
    // 上传文件
    upload.transferTo(new File(file,filename));
    return "success";
}
```

配置文件解析器对象

```xml
<!-- 配置文件解析器对象，要求id名称必须是multipartResolver -->
<bean id="multipartResolver"
class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
	<property name="maxUploadSize" value="10485760"/>
</bean>
```

![](https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-03-%E5%8E%9F%E7%90%86-f78b86.bmp)

## 3.SpringMVC 跨服务器方式文件上传

- 搭建图片服务器。

- 实现 SpringMVC 跨服务器方式文件上传。

  - 导入开发需要的 jar 包。

  ```xml
  <dependency>
      <groupId>com.sun.jersey</groupId>
      <artifactId>jersey-core</artifactId>
      <version>1.18.1</version>
  </dependency>
  <dependency>
      <groupId>com.sun.jersey</groupId>
      <artifactId>jersey-client</artifactId>
      <version>1.18.1</version>
  </dependency>
  ```

  - 编写文件上传的 JSP 页面。

  ```html
  <h3>跨服务器的文件上传</h3>
  <form action="user/fileupload3" method="post" enctype="multipart/form-data">
      选择文件：<input type="file" name="upload"/><br/>
      <input type="submit" value="上传文件"/>
  </form>
  ```

  - 编写控制器。

  ```java
  /**
  * SpringMVC跨服务器方式的文件上传
  *
  * @param request
  * @return
  * @throws Exception
  */
  @RequestMapping(value="/fileupload3")
  public String fileupload3(MultipartFile upload) throws Exception {
      System.out.println("SpringMVC跨服务器方式的文件上传...");
     
      // 定义图片服务器的请求路径
      String path = "http://localhost:9090/day02_springmvc5_02image/uploads/";
      
      // 获取到上传文件的名称
      String filename = upload.getOriginalFilename();
      String uuid = UUID.randomUUID().toString().replaceAll("-", "").toUpperCase();
      // 把文件的名称唯一化
      filename = uuid+"_"+filename;
      // 向图片服务器上传文件
      
      // 创建客户端对象
      Client client = Client.create();
      // 连接图片服务器
      WebResource webResource = client.resource(path+filename);
      // 上传文件
      webResource.put(upload.getBytes());
      return "success";
  }
  ```

  

