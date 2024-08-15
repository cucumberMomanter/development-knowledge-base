#Filter #Spring_boot_Filter
> Регистрация фильтра в Spring Boot app

### Материалы
[Видео примера](https://yandex.ru/video/preview/3067962800311768759)

### Example
#### Filter
```java
@Slf4j  
public class HandleFilter implements Filter {
  
    @Override  
    public void doFilter(ServletRequest servletRequest,
					     ServletResponse servletResponse,
					     FilterChain filterChain) throws IOException,
													     ServletException {  
        log.info("Filter start");
        filterChain.doFilter(servletRequest, servletResponse);  
        log.info("Filter finish");  
    }
}
```

#### Регистрация фильтра в Spring Boot
```java
@Configuration  
public class FilterConfig {
  
    @Bean  
    public FilterRegistrationBean<HandleFilter> registrationBean() {
        FilterRegistrationBean<HandleFilter> registrationBean =
									        new FilterRegistrationBean<>();
        registrationBean.setFilter(new HandleFilter());
        registrationBean.addUrlPatterns("/*");
  
        return registrationBean;
    }
}
```