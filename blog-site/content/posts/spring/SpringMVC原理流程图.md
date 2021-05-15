---
title: "Springmvc原理流程图"
date: 2020-12-25
draft: false
tags: ["spring"]
slug: "spring-mvc"
---

![springmvc原理图](/myblog/posts/images/application/springmvc原理.jpg)

流程描述:
- 用户发送请求至前端控制器 `DispatcherServlet`;
- `DispatcherServlet` 收到请求调用 `HandlerMapping` 处理器映射器;
- 处理器映射器找到具体的处理器(可以根据xml配置、注解进行查找)，生成处理器对象及处理器拦截器(如果有则生成)一并返回给 `DispatcherServlet`;
- `DispatcherServlet` 调用 `HandlerAdapter` 处理器适配器;
- `HandlerAdapter` 经过适配调用具体的处理器(`Controller`，也叫后端控制器);
- `Controller` 执行完成返回 `ModelAndView`;
- `HandlerAdapter` 将 `controller执行结果` `ModelAndView` 返回给 `DispatcherServlet`;
- `DispatcherServlet` 将 `ModelAndView` 传给 `ViewReslover` 视图解析器;
- `ViewReslover` 解析后返回具体 `View`;
- `DispatcherServlet` 根据 View 进行渲染视图（即将模型数据填充至视图中）;
- `DispatcherServlet` 响应用户;

