---
title: '#Spring Cors以及Filter和Interceptor'
date: 2021-09-01 11:35:41
tags:
---

# Cors 在spring中配置方法

## 1. @CrossOrigin 注解

```java
@RestController
class HelloController {
    @GetMapping("hello")
    @CrossOrigin(origins = ["http://localhost:8080"]) #针对方法handler
    fun hello(): String {
        return "Hello, CORS!"
    }
}
```

## 2. 实现 WebMvcConfigurer.addCorsMappings 方法

增加全局配置
```java
@Configuration
@EnableWebMvc
class MvcConfig: WebMvcConfigurer {
    override fun addCorsMappings(registry: CorsRegistry) {
        registry.addMapping("/hello")
                .allowedOrigins("http://localhost:8080")
    }
}
```

此种方法使用了CorsRegistry 和 CorsRegistration

## 3. 注入 CorsFilter

```java
@Configuration
class CORSConfiguration {
    @Bean
    fun corsFilter(): CorsFilter {
        val configuration = CorsConfiguration()
        configuration.allowedOrigins = listOf("http://localhost:8080")
        val source = UrlBasedCorsConfigurationSource()
        source.registerCorsConfiguration("/hello", configuration)
        return CorsFilter(source)
    }
}
```

也可以实现FilterRegistrationBean
```java
@Bean
public FilterRegistrationBean<RequestResponseLoggingFilter> loggingFilter(){
    FilterRegistrationBean<RequestResponseLoggingFilter> registrationBean 
      = new FilterRegistrationBean<>();
        
    registrationBean.setFilter(new RequestResponseLoggingFilter());
    registrationBean.addUrlPatterns("/users/*");
        
    return registrationBean;    
}
```

## 4. Spring Security 中的配置

引入 spring.security后，以上方式都会失效，需要增加配置

```java
@Configuration
class SecurityConfig : WebSecurityConfigurerAdapter() {
    override fun configure(http: HttpSecurity?) {
        http?.cors()
    }
}
```

或者与corsConfigurationSource配合

```java
@Bean
fun corsConfigurationSource(): CorsConfigurationSource {
    val configuration = CorsConfiguration()
    configuration.allowedOrigins = listOf("http://localhost:8080")
    val source = UrlBasedCorsConfigurationSource()
    source.registerCorsConfiguration("/hello", configuration)
    return source
}
```


# 区别 

**handler ->
interceptor ->
Dispatch Servlet ->
Filter ->
Web Container ->
Client**
---


- 实现 WebMvcConfigurer.addCorsMappings 方法来进行的 CORS 配置，最后会在 Spring 的 Interceptor 或 Handler 中生效

- 注入 CorsFilter 的方式会让 CORS 验证在 Filter 中生效

- 引入 Spring Security 后，需要调用 HttpSecurity.cors 方法以保证 CorsFilter 会在身份验证相关的 Filter 之前执行

- HttpSecurity.cors + WebMvcConfigurer.addCorsMappings 是一种相对低效的方式，会导致跨域请求分别在 Filter 和 Interceptor 层各经历一次 CORS 验证

- HttpSecurity.cors + 注册 CorsFilter 与 HttpSecurity.cors + 注册 CorsConfigurationSource 在运行的时候是等效的

- 在 Spring 中，没有通过 CORS 验证的请求会得到状态码为 403 的响应

[Reference详解](https://segmentfault.com/a/1190000019485883)