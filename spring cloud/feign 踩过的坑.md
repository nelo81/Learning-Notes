## feign 踩过的坑

1. feign 发送带文件的请求

- 在项目中添加以下配置
```
@Configuration
public class MultipartSupportConfig {
	
	@Autowired
    private ObjectFactory<HttpMessageConverters> messageConverters;

    @Bean
    public Encoder feignFormEncoder() {
        return new SpringFormEncoder(new SpringEncoder(messageConverters));
    }
}
```
- 然后在 feignclient 接口处

```
@FeignClient("file")
public interface FeignFileDao {
  @RequestMapping(value = "/file/upload", method = RequestMethod.POST,
			consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
	public boolean upload(@RequestPart("file") MultipartFile file);
}
```

2. feign 发送多个参数

- 在 feignclient 接口处，不带任何 annotation 或者带有 @RequestBody 的参数被认为是请求的 request body，因此类似以下写法是会报错："too many body paramters'...."
```
@RequestMapping(value = "/path", method = RequestMethod.POST)
	public Resource resource(String x1, String x2);
```

解决方法一：把多个参数封装成一个对象发送
```
@RequestMapping(value = "/path", method = RequestMethod.POST)
	public Resource resource(@RequestBody Map<String,Object> req);
```
但是这样做的话，提供这个接口的服务的参数也要从 `@RequestBody Map<String,Object> req` 拿出来，换成其他对象也一样

解决方法二：用 `@RequestParam("param")`
```
@RequestMapping(value = "/path", method = RequestMethod.POST)
	public Resource resource(@RequestParam("x1")String x1, @RequestParam("x2")String x2);
```
但是这样做的话参数会放在 url 后面，就是 `http://xxx.com/path?x1=x1&x2=x2` 这种格式，我之前这样做就出现过请求头过长的错误。

3. 不要在 feignclient 接口类加上 `@RequestMapping` 注解，否则会被 springmvc 扫描到认为是 controller，这样的话，这些接口就可以被外部访问，有二级目录的情况下，也放在方法上，例如下面：

```
@FeignClient("file")
@RequestMapping("/file")
public interface FeignFileDao {
  @RequestMapping(value = "/upload", method = RequestMethod.POST,
			consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
	public boolean upload(@RequestPart("file") MultipartFile file);
}
```
要改成
```
@FeignClient("file")
public interface FeignFileDao {
  @RequestMapping(value = "/file/upload", method = RequestMethod.POST,
			consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
	public boolean upload(@RequestPart("file") MultipartFile file);
}
```

4. feignclient 请求超时

- 在 "application.yml" 配置中加入：`hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds: 5000`