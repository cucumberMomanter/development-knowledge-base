#HandlerInterceptor #Spring #Spring_mvc
### Материалы
- [Статья](https://for-each.dev/lessons/b/-spring-mvc-handlerinterceptor-vs-filter)
- [Видео примера реализации](https://yandex.ru/video/preview/6589425335468970512)

### Example
#### HandlerInterceptor
```java
@Slf4j
@Component
@RequiredArgsConstructor  
public class RequestHandleInterceptor implements HandlerInterceptor {
  
    @Override  
    public boolean preHandle(HttpServletRequest request,
						     HttpServletResponse response,
						     Object handler) throws Exception {
		// выполняется перед вызовом целевого обработчика(endpoint-а)
        log.info("preHandle");
    }  
  
    @Override  
    public void postHandle(HttpServletRequest request,
						   HttpServletResponse response,
						   Object handler,
						   ModelAndView modelAndView) throws Exception {
		// выполняется после целевого обработчика, но до того,
		// как `DispatcherServlet` отобразит представление
        log.info("postHandle");  
    }  
  
    @Override  
    public void afterCompletion(HttpServletRequest request,
							    HttpServletResponse response,
							    Object handler,
							    Exception ex) throws Exception {
		// Обратный вызов после завершения обработки запроса
		// и рендеринга представления
		log.info("afterCompletion");  
    }
```

#### Конфигурирование HandlerInterceptor
```java
@Configuration  
@RequiredArgsConstructor  
public class HandleInterceptorConfig implements WebMvcConfigurer {
  
    private final RequestHandleInterceptor requestHandleInterceptor; 
  
    @Override  
    public void addInterceptors(InterceptorRegistry registry) {  
        registry.addInterceptor(requestHandleInterceptor)  
		// ВАЖНО: совпадение с чем либо - это **
                .addPathPatterns("/**")
                .excludePathPatterns("/api/test/**", "/api/v1/**");  
    }  
}
```
### Выводы
- Нельзя достать тело у request и response.
  - У **request** - это `InputStream` и его можно вычитать только один раз.
  - У **response** - `OutputStream` и в него можно только записать. 

P.S: Выход писать `Filter`, в нем делать обертки для request и response (`RequestWrapper`, `ResponseWrapper`). А в этих обертках кешировать потоки ввода/вывода `body`.
- В `request` нельзя добавить заголовки(headers).