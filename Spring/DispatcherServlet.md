### @RequestMapping

value：可以模糊匹配

* ？匹配一个字符 /an?/222

header：请求头里面的信息匹配才会接收

comsume：检查 Content-Type 字段。检查请求的数据类型

prosume：检查 Accept。客户端接收的数据类型



### REST

资源的表现层状态转化

以一个 URL 来实现增删查改。以请求的方式来区分

@GetMapping、@PostMapping



### @RequestHeader

获取请求头的信息

![image-20200221225127738](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200221225127738.png)





### @CookieValue

获取请求头中 cookie 的值

![image-20200221225411050](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200221225411050.png)

![image-20200221225550373](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200221225550373.png)





### POJO 对象参数自己赋值

级联对象时候的 name 值

![image-20200221231331398](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200221231331398.png)





### Spring MVC 使用原生 API

![image-20200221231653421](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200221231653421.png)



### Map、Model、ModelMap(携带数据放在 request 域中 )

![image-20200221232208298](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200221232208298.png)

比如用 Model 做参数，其实传入的是一个 BindingAwareModelMap 类型的参数

![image-20200221232414090](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200221232414090.png)



### ModelAndView(携带数据放在 request 域中 )

![image-20200221232644676](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200221232644676.png)



### @SessionAttribute 给 Session 中放数据





### @DispatcherServlet

![image-20200309125212584](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200309125212584.png)

```java
//检查文件是否上传请求
processedRequest = this.checkMultipart(request);

//根据当前的请求地址找到哪个类能来处理
//会返回目标处理器类的执行链
//这里的属性也包括拦截器
mappedHandler = this.getHandler(processedRequest);

//如果没有找到哪个处理器能处理这个请求就404，或者抛异常
if (mappedHandler == null || mappedHandler.getHandler() == null) {
    this.noHandlerFound(processedRequest, response);
    return;
}

//拿到能执行这个类的所有方法的适配器(反射工具AnnotationMethodHandlerAdapter)
HandlerAdapter ha = this.getHandlerAdapter(mappedHandler.getHandler());

//执行拦截器的方法
if (!mappedHandler.applyPreHandle(processedRequest, response)) {
    return;
}


//处理器（控制器）的方法被调用
//控制器（Controller），处理器（Handler）
//适配器来执行目标方法；将目标方法执行完成后的返回值作为视图名，设置保存到ModelAndView 中
//目标方法无论怎么写，最终适配器执行完成后都会将执行后的信息封装成 ModelAndView
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());


// 如果 View 为空则弄一个默认的视图名
this.applyDefaultViewName(processedRequest, mv);

//转发到目标页面
//根据方法最终执行完成后封装的 ModelAndView，转发到对应的页面，而且 ModelAndView 中的数据可以从请求域中获取
this.processDispatchResult(processedRequest, response, mappedHandler, mv, (Exception)dispatchException);

```

![image-20200222224151576](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200222224151576.png)



### 异常处理

```java
//在处理方法时抛出异常
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

//异常会被捕获
//然后来到渲染页面方法
this.processDispatchResult(processedRequest, response, mappedHandler, mv, (Exception)dispatchException);
	//里面有这个处理异常的方法
	mv = this.processHandlerException(request, response, handler, exception);

//如何处理呢?
//拿所有的 HandlerExceptionResolver （处理异常解析器）来处理异常
Iterator var6 = this.handlerExceptionResolvers.iterator();
while(var6.hasNext()) {
            HandlerExceptionResolver handlerExceptionResolver = (HandlerExceptionResolver)var6.next();
            exMv = handlerExceptionResolver.resolveException(request, response, handler, ex);
    		//判断是否能处理
            if (exMv != null) {
                break;
            }
        }

```

```java
ExceptionHandlerExceptionResolver：@ExceptionHandler
ResponseStatusExceptionResolver:  @ResponseStatus
DefaultHandlerExceptionResolver  判断是否 Spring MVC 自带的异常
```

#### @ExceptionHandle 

value 是个数组

参数只能写 exception；即错误的信息；不能写 Model 这些

当想传参给视图的时候，我们可以用 ModelAndView，然后返回即可

全局与本类有，本类的异常优先

![image-20200222202247254](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200222202247254.png)

#### @ResponseStatus(reason="原因"，value=状态码)

标在异常类上



### 视图解析

```java
// 渲染视图
this.render(mv, request, response); 

// View与ViewResolver：ViewResolver 的作用是根据视图名（方法的返回值）得到 View 对象
view = this.resolveViewName(viewName, mv.getModelInternal(), locale, request);


// 根据 ViewName 去获取 View 视图
 protected View resolveViewName(String viewName, @Nullable Map<String, Object> model, Locale locale, HttpServletRequest request) throws Exception {
        if (this.viewResolvers != null) {
            Iterator var5 = this.viewResolvers.iterator();
			//遍历所有的 ViewResolver
            while(var5.hasNext()) {
                ViewResolver viewResolver = (ViewResolver)var5.next();
                // ViewResolver 视图解析器根据方法的返回值，得到一个 View 对象；
                View view = viewResolver.resolveViewName(viewName, locale);
                if (view != null) {
                    return view;
                }
            }
        }

        return null;
    }


//根据方法的返回值创建出视图对象
view = this.createView(viewName, locale);
```

```java
createView(viewName, locale);
//判断是否有前缀 如redirect forward 没有就创建
            if (viewName.startsWith("redirect:")) {
                forwardUrl = viewName.substring("redirect:".length());
                RedirectView view = new RedirectView(forwardUrl, this.isRedirectContextRelative(), this.isRedirectHttp10Compatible());
                String[] hosts = this.getRedirectHosts();
                if (hosts != null) {
                    view.setHosts(hosts);
                }

                return this.applyLifecycleMethods("redirect:", view);
            } else if (viewName.startsWith("forward:")) {
                forwardUrl = viewName.substring("forward:".length());
                InternalResourceView view = new InternalResourceView(forwardUrl);
                return this.applyLifecycleMethods("forward:", view);
            } else {
                return super.createView(viewName, locale);
            }
```



视图解析器只是为了得到视图对象；视图对象才能真正的转发（将模型数据全部放在请求域中）或者重定向到页面



### 重定向时失去参数问题

#### 两种办法：通过 URL 模板进行重定向；

只能传 String 和数字的值

```java
@PostMapping(value = "/ss")
public String success(Spitter spitter, Model model) {
    model.addAttribute("username", spitter.getUsername());
    model.addAttribute("spitterId", spitter.getId());
    return "redirect:/spitter/{username}";
}

//重定向的 URL 为
/spitter/habuma?spitterId=42
//即加在 model 中的数据会一起加载过去
```

#### 使用 flash 属性

可以传对象，即把属性放在了会话中

```
@PostMapping(value = "/ss")
public String success(Spitter spitter, RedirectAttributes model) {
    model.addAttribute("username", spitter.getUsername());
    //可以把整个对象传过去
    model.addFlashAttribute("spitter", spitter);
    return "redirect:/spitter/{username}";
}
```



### HttpEntity

可以拿到请求头和请求体

![image-20200222180140257](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200222180140257.png)

![image-20200222181027899](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200222181027899.png)



### @ResponseBody

将返回的内容放在响应体中，检查请求中 Accept 字段，看客户端需要什么类型的数据，再用消息转换器进行转换

### @RequestBody

查看请求中的 Content-Type 头部信息，查找消息转换器进行转换



### ResponseEntity

可以自定义响应头、响应体和状态码

不需要加 @ResponseBody 注解 

ResponseEntity 包含了 @ResponseBody 的语义

![image-20200222181847814](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200222181847814.png)



### Interceptor

在多个拦截器情况下，只要对应的拦截器放行，即它对应的afterCompletion 会执行





### RestTemplate

```java
// method 方法
// requestEntity 请求实体，内容可以放在 body 中，头部字段也在这里修改
// responseType 返回的 ResponseEntity 类型
// uriVariables 拼接在 url 中的参数 如：/zz/ss/{cc} 
// uriVariables.put("cc","aa")
// 会自动拼接成 /zz/ss/aa

public <T> ResponseEntity<T> exchange(String url, HttpMethod method, @Nullable HttpEntity<?> requestEntity, Class<T> responseType, Map<String, ?> uriVariables) throws RestClientException
```

