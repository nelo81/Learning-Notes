## spring boot 统一异常处理

- 很简单，只要自己重写一个 ErrorController 的子类配置就好，如下：

```
@RestController
@EnableConfigurationProperties({ ServerProperties.class })
public class FilterController implements ErrorController {

	private ErrorAttributes errorAttributes;
	private @Autowired ServerProperties serverProperties;

	public FilterController(ErrorAttributes errorAttributes) {
		Assert.notNull(errorAttributes, "ErrorAttributes must not be null");
		this.errorAttributes = errorAttributes;
	}

	@Override
	public String getErrorPath() {
		return "/error";
	}

	@RequestMapping("/error")
	public ResponseEntity<ApiResult> error(HttpServletRequest request) {
		Map<String, Object> body = getErrorAttributes(request, isIncludeStackTrace(request, MediaType.ALL));
		HttpStatus status = getStatus(request);
		String msg = "";
		if(body.containsKey("message")) msg = body.remove("message").toString();
		return new ResponseEntity<ApiResult>(ApiResult.custom(status.value(), msg, body, null), status);
	}

	protected Map<String, Object> getErrorAttributes(HttpServletRequest request, boolean includeStackTrace) {
		RequestAttributes requestAttributes = new ServletRequestAttributes(request);
		return this.errorAttributes.getErrorAttributes(requestAttributes, includeStackTrace);
	}

	protected boolean getTraceParameter(HttpServletRequest request) {
		String parameter = request.getParameter("trace");
		if (parameter == null) {
			return false;
		}
		return !"false".equals(parameter.toLowerCase());
	}

	protected HttpStatus getStatus(HttpServletRequest request) {
		Integer statusCode = (Integer) request.getAttribute("javax.servlet.error.status_code");
		if (statusCode == null) {
			return HttpStatus.INTERNAL_SERVER_ERROR;
		}
		try {
			return HttpStatus.valueOf(statusCode);
		} catch (Exception ex) {
			return HttpStatus.INTERNAL_SERVER_ERROR;
		}
	}

	protected boolean isIncludeStackTrace(HttpServletRequest request, MediaType produces) {
		ErrorProperties.IncludeStacktrace include = this.serverProperties.getError().getIncludeStacktrace();
		if (include == ErrorProperties.IncludeStacktrace.ALWAYS) {
			return true;
		}
		if (include == ErrorProperties.IncludeStacktrace.ON_TRACE_PARAM) {
			return getTraceParameter(request);
		}
		return false;
	}
}
```
- 这个类本身也是个 controller
- 这个类 90% 都是复制，主要注意的是 getErrorPath() 方法，这个类表示在遇到 http 异常时应该访问哪个 action，上面这个例子是 "/error"，因此请求出现异常时会访问带有 `@RequestMapping("/error")` 注解的方法并以那个方法返回。
- error 方法中，`getErrorAttributes(request, isIncludeStackTrace(request, MediaType.ALL))` 用于获取请求的错误信息，
- `getStatus(HttpServletRequest request)`  方法用于获取 http 状态码