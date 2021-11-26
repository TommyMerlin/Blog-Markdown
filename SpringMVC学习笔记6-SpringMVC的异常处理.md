---
title: SpringMVC学习笔记6:SpringMVC的异常处理
date: 2020-03-05 17:34:26
tags:
- Java
- SpringMVC
categories:
- Java
- Spring
- SpringMVC
---

## 1.异常处理思路

Controller 调用 service，service 调用 dao，异常都是向上抛出的，最终由 DispatcherServlet 找异常处理器进行异常的处理。

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-05-2f9f46.bmp" alt="05" width="70%" />

<!-- more -->

## 2.SpringMVC的异常处理

- 自定义异常类

```java
public class SysException extends Exception{
    
    private static final long serialVersionUID = 4055945147128016300L;
    
    // 异常提示信息
    private String message;
    
    public String getMessage() {
    	return message;
    }
    
    public void setMessage(String message) {
    	this.message = message;
    }
    
    public SysException(String message) {
    	this.message = message;
    }
}
```

- 自定义异常处理器

```java
/**
* 异常处理器
*/
public class SysExceptionResolver implements HandlerExceptionResolver{
    /**
    * 跳转到具体的错误页面的方法
    */
    public ModelAndView resolveException(HttpServletRequest request, 
                                         HttpServletResponse response, 
                                         Object handler,Exception ex) {
        ex.printStackTrace();
        SysException e = null;
        // 获取到异常对象
        if(ex instanceof SysException) {
        	e = (SysException) ex;
        }else {
        	e = new SysException("请联系管理员");
        }
        ModelAndView mv = new ModelAndView();
        // 存入错误的提示信息
        mv.addObject("message", e.getMessage());
        // 跳转的Jsp页面
        mv.setViewName("error");
        return mv;
    }
}
```

- 配置异常处理器

```xml
<!-- 配置异常处理器 -->
<bean id="sysExceptionResolver" class="cn.itcast.exception.SysExceptionResolver"/>
```

