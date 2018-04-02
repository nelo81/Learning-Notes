## spring boot 内存优化

1. 自定义加载类（不过这种方法需要对每个配置都很了解）

- 随着 spring 社区的壮大，spring boot 中的 `@EnableAutoConfiguration` 会帮我们自动加载很多类，这样确实很方便但是，由于加载了很多我们可能不需要的类，不仅降低了启动速度，还占用更多了内存，因此我们需要自己定义加载类来优化。

- 只要修改 spring boot 的启动类，一般自动配置的我们会用下面的代码
```
@SpringBootApplication  // @SpringBootApplication 包含了 @EnableAutoConfiguration 和 @Configuration
public class App {
	public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}
```

我们需要改成如下（下面的类都是最基本的常用类，也可以自行增减）
```
@Configuration
@Import({  // @Import 中定义我们要加载的类
    DispatcherServletAutoConfiguration.class,
    EmbeddedServletContainerAutoConfiguration.class,
    ErrorMvcAutoConfiguration.class,
    HttpEncodingAutoConfiguration.class,
    HttpMessageConvertersAutoConfiguration.class,
    JacksonAutoConfiguration.class,
    JmxAutoConfiguration.class,
    MultipartAutoConfiguration.class,
    ServerPropertiesAutoConfiguration.class,
    PropertyPlaceholderAutoConfiguration.class,
    ThymeleafAutoConfiguration.class,
    WebMvcAutoConfiguration.class,
    WebSocketAutoConfiguration.class,
})
public class App {
	public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}
```

2. 运行 springboot 时增加 JVM 参数：
```
java \
-Xms32m \
-Xmx32m \
-XX:MaxMetaspaceSize=64m \
-XX:CompressedClassSpaceSize=8m \
-Xss256k \
-Xmn8m \
-XX:InitialCodeCacheSize=4m \
-XX:ReservedCodeCacheSize=8m \
-XX:MaxDirectMemorySize=16m \
-Djava.security.egd=file:/dev/./urandom \
-jar app.jar \
```