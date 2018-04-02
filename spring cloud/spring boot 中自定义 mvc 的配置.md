## spring boot 中自定义 mvc 的配置

1. 用于加载自定义 mvc 配置的类

```
@Configuration
public class WebMvcConfig {
	
	@Bean
	public WebMvcConfigurerAdapter webMvcConfigurerAdapter() {
		return new TokenMvcConfig();
	}
}
```

其中 TokenMvcConfig.class 要继承 WebMvcConfigurerAdapter，重写 WebMvcConfigurerAdapter 的方法类导入自定义的配置

```
public class TokenMvcConfig extends WebMvcConfigurerAdapter{
	@Override
	....
}
```

2. 自定义 mvc 拦截器，在 WebMvcConfigurerAdapter 子类重写 addInterceptors 方法

```
public class TokenMvcConfig extends WebMvcConfigurerAdapter{
	@Override
	public void addInterceptors(InterceptorRegistry registry) {
		registry.addInterceptor(new TokenIntercepter()).addPathPatterns("/**");
	}
}
```
- "/\*\*" 表示拦截所有请求，"/path/\*\*" 表示拦截 "/path/" 后加任何字符的请求
- TokenIntercepter 是我们自定义的拦截器，如下

```
public class TokenIntercepter implements HandlerInterceptor{

	//执行对应controller方法之前，检验用户权限是否可以执行此方法，返回true通过执行，false拦截
	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		Method action = ((HandlerMethod) handler).getMethod();
		return true
	}
	
	@Override
	public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
			ModelAndView modelAndView) throws Exception {
	}

	@Override
	public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
			throws Exception {
	}
}
```
- handler 表示处理这次请求的 controller，可以通过 `((HandlerMethod) handler).getMethod()` 获取那个 controller 的方法
- request 就是请求内容
- response 就是返回内容，在 return false 拦截请求的情况下，会返回这个 response

3. 自定义返回值拦截和处理类，在 WebMvcConfigurerAdapter 子类重写 addReturnValueHandlers 方法

```
public class TokenMvcConfig extends WebMvcConfigurerAdapter{
	@Override
	public void addReturnValueHandlers(List<HandlerMethodReturnValueHandler> returnValueHandlers) {
		returnValueHandlers.add(new TokenReturnValueHandler());
	}
}
```
- 要注意的是，在要用这个自定义类处理的 controller 中不能使用 @ResponseBody, @RestController 等继承了@ResponseBody的注解，否则 spring mvc 会先用自己的 returnValueHandler 处理并返回，轮不到你这个自定义的来处理了。
- TokenReturnValueHandler 是我们自定义的处理器，如下：

```
public class TokenReturnValueHandler implements HandlerMethodReturnValueHandler{
	@Override
	public boolean supportsReturnType(MethodParameter returnType) {
		return returnType.getParameterType() == ApiResult.class;
	}

	@Override
	public void handleReturnValue(Object returnValue, MethodParameter returnType, ModelAndViewContainer mavContainer,
			NativeWebRequest webRequest) throws Exception {
		mavContainer.setRequestHandled(true);//禁止返回 ModelAndView，直接在这个方法返回结果
		HttpServletResponse response = webRequest.getNativeResponse(HttpServletResponse.class);
	}
}
```
- 处理顺序是先执行 `supportsReturnType(MethodParameter returnType)` 其中，参数 returnType 是 controller 的返回值类型，这个函数返回 true 才执行 handleReturnValue 方法。
- handleReturnValue 方法中，`mavContainer.setRequestHandled(true)` 中的作用有点类似于 `@ResponseBody` ，即直接在这个方法返回 response 而不是用 spring mvc 返回 ModelAndView。
- handleReturnValue 方法的参数 webRequest 是请求内容，可以通过 webRequest.getParameter() 等方法获取请求参数
- handleReturnValue 方法中，`webRequest.getNativeResponse(HttpServletResponse.class)` 可以获得 response 的返回内容，类型是 HttpServletResponse。