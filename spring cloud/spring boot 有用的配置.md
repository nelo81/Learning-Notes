## spring boot 有用的配置

1. 加载静态文件目录
```
spring:
  mvc:
    static-path-pattern: /static/**
  resources:
    static-locations: |-
     classpath:/META-INF/resources/,
     classpath:/resources/,
     classpath:/static/,
     classpath:/public/,
     file:${file.static-dir}
```

- spring.mvc.static-path-pattern: 表示访问静态文件的规则，例如上面 "/static/\*\*" 那么在访问静态资源 log.txt 时就要访问 "/static/log.txt"，如果设置成 "/\*\*" 那么就可以直接访问 "/log.txt"
- spring.resources.static-locations：表示静态文件目录列表，上面 file 那个是我自己加的，file 代表文件目录，classpath 就是项目目录。
- 注意： |- 是 yaml 语法，代表下面可以换行，因此不是有很多个 spring.resources.static-locations.classpath，而是 `spring.resources.static-locations = "classpath:/META-INF/resources/, classpath:/resources/, classpath:/static/, ...."`

2. 限制请求大小
```
spring:
  http:
    multipart:
      enabled: true
      max-file-size: 10MB
      max-request-size: 10MB
```
- spring.http.multipart.max-file-size: 文件上传最大值
- spring.http.multipart.max-request-size: http 请求最大值

3. 对请求使用 utf-8 编码
```
spring:
  http:
    encoding:
      force: true
      charset: UTF-8
      enabled: true
```

4. 日志输出
```
logging:
  level.root: warn
  level.package.classname: debug
  file: /tmp/log/spring.log
```
- logging.level.root: 整个项目的日志输出级别
- logging.level.package.classname: 某个类的输出级别
- logging.file: 输出到本地文件，默认不输出到文件