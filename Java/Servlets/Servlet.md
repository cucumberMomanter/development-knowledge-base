### Материалы
- [Простой видео курс (9 видео)](https://www.youtube.com/watch?v=XiSXn_q3mgc&list=PL7Bt6mWpiizZq71c4wuBl7lmY-M7nen_J&index=1)
- [Java Rush лекции](https://javarush.com/quests/lectures/questservlets.level12.lecture00)

### Введение
> Веб-сервер (например Tomcat) - это контейнер сервлетов. А сервлет - это загружаемый в контейнер объект.

Жизненный цикл сервлета:
1. Класс сервлета загружается контейнером.
2. Контейнер создает экземпляр класса (объект) сервлета.
3. Контейнер вызывает метод `init()` у объекта сервлета. Метод вызывается только один раз.

Стандартный цикл работы — **обслуживание клиентского запроса**:

4. Каждый запрос обрабатывается в отдельном потоке.
5. Контейнер вызывает метод `service()` у сервлета и передает туда объекты ServletRequest и ServletResponse.
6. Для завершения работы сервлета вызывается метод `destroy()` у объекта сервлета. Вызывается он всего один раз.

<font style="color:red">ВАЖНО</font>: веб-сервер и его сервлеты должны без сбоев и перезагрузки работать месяцами, обслуживая тысячи запросов в минуту. Поэтому код загрузки, работы и выгрузки сервлета всегда нужно писать очень качественно.

### HttpServlet
> Каждый сервлет является `singleton` и работает в многопоточной среде, очень важно это учитывать при написании сервлета. Instance variable должны быть потокобезопасными.

#### service()
Если смотреть на обработку клиентского запроса с точки зрения сервлета, то дела обстоят примерно так:
- для каждого клиентского запроса контейнер (веб-сервер) создает объекты `HttpServletRequest` и `HttpServletResponse`;
- затем вызывает метод `service(HttpServletRequest request, HttpServletResponse response)` у соответствующего сервлета;
- в сервлет передаются эти объекты, чтобы метод мог взять нужные данные из `request’а` и положить результат работы в `response`.

#### init()
> После того, как веб-сервер создал объект сервлета и поместил его в контейнер, он вызывает у сервлета метод `init()`.

У метода `init()` есть две перегруженные версии:
- `init()` - его можно переопределить и добавить необходимую инициализацию.
- `init(ServletConfig config)` - при его переопределении важно не забыть вызвать супер метод и передать ему конфигурацию: `super.init(config)`. Лучше переопределять `init()` без параметров.

> Оба эти метода вызываются при инициализации сервлета.

<br>

### HttpSession
> Зачем нужна сессия? В ней можно хранить информацию о клиенте между вызовами. У нее внутри есть что-то вроде HashMap, в котором можно хранить объекты по ключам.
> 
> Под именем `JSESSIONID` в куках хранится ID сессии.

Если несколько запросов идут от одного клиента, то говорят, что между клиентом и сервером установилась сессия. Для контроля этого процесса у контейнера есть специальный объект `HttpSession`.

Когда клиент обращается к сервлету, то контейнер сервлетов проверяет, есть ли в запросе параметр ID сессии. Если такой параметр отсутствует (например, клиент первый раз обращается к серверу), тогда контейнер сервлетов создает новый объект `HttpSession`, а также присваивает ему уникальный ID.

Объект сессии сохраняется на сервере, а ID отправляется в ответе клиенту и по умолчанию сохраняется на клиенте в куках. Затем, когда приходит новый запрос от того же клиента, то контейнер сервлетов достает из него ID, и по этому ID находит правильный объект `HttpSession` на сервере.

Получить объект сессии можно из запроса (объект `HttpServletRequest`), у которого нужно вызвать метод `getSession()`. Он возвращает объект `HttpSession`.

#### Метод setMaxInactiveInterval()
> `setMaxInactiveInterval(int seconds)`- устанавливает интервал неактивности сессии в секундах.

Если в течение interval времени сессией никто не пользовался, то она самоочищается — из нее удаляются все объекты, которые она хранила. Это сделано для экономии памяти. По умолчанию этот интервал равен 1800 секундам == 30 минутам.

### ServletConfig
> это мета-информация о сервлете, которая была излечена из `web.xml`.
> `ServletConfig` можно получить, вызвав у сервлета метод `getServletConfig()`.

### ServletContext
> обеспечивает область видимости всего web-приложения. Он общий для всех сервлетов.

### ServletContextListener
> это интерфейс, при помощи которого можно записывать в контекст (`ServletContext`) всего web-приложения какие-л данные, еще <font style="color:red">до того, как первый сервлет будет создан</font>.
> 
> Используется для инициализации(init) и закрытия(destroy) контекста.

Пример (добавляем хранилище пользователей для всех сервлетов):
```java
// можно конфигурировать как через @WebListener, так и через web.xml
@WebListener
public class ContextListener implements ServletContextListener {

    private Map<Integer, User> users;

    @Override
    public void contextInitialized(ServletContextEvent servletContextEvent) {

        final ServletContext servletContext =
                servletContextEvent.getServletContext();

        users = new ConcurrentHashMap<>();

        servletContext.setAttribute("users", users);
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        //Close recourse.
        users = null;
    }
}
```


### RequestDispatcher
>  - это объект, который получает запросы от клиента и отправляет их любому ресурсу (например, сервлету, HTML-файлу или JSP-файлу) на сервере.
> Контейнер сервлета создает объект RequestDispatcher, который используется в качестве оболочки для серверного ресурса, расположенного по определенному пути или заданного определенным именем.
> 
> Этот объект можно использовать для того, чтобы перенаправить существующий запрос на другой сервлет. Например, выяснилось, что пользователь не авторизован и мы хотим показать ему страницу с авторизацией. Ну или произошла ошибка на сервере и мы хотим отобразить пользователю error-страницу.

`RequestDispatcher` можно получить двумя способами:
- У объекта `HttpServletRequest`
- У объекта `ServletContext`

```java
public class HelloServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request,
					     HttpServletResponse response) throws Exception {
        String path = "/error.html";  
        ServletContext servletContext = this.getServletContext();  
        RequestDispatcher requestDispatcher =
			    servletContext.getRequestDispatcher(path);  
        requestDispatcher.forward(request, response);  
    }  
}
```

#### Сравнение редиректа и форварда
Если ты хочешь в своем сервлете перенаправить пользователя на другой URI, то сделать это можно двумя способами:
- `redirect` (вызывается у response)
- `forward`

Когда ты выполняешь **redirect** через вызов `response.sendRedirect("ссылка")`, то сервер отсылает браузеру (клиенту) ответ `302` и указанную тобой ссылку. А браузер, проанализировав ответ сервера, загружает переданную тобой ссылку. То есть ссылка в браузере меняется на новую.

Если ты выполняешь **forward** через вызов `requestDispatcher.forward()`, то новый запрос выполняется внутри контейнера, <font style="color:red">и его ответ твой сервлет отсылает браузеру (клиенту)</font> как ответ твоего сервлета. При этом браузер получает ответ от нового сервлета, но ссылка в браузере не меняется.

### Filter
> **Фильтры** — “служебные сервлеты” . Они очень похожи на сервлеты, но их основная задача — помогать сервлетам обрабатывать запросы.
> 
> В фильтрах можно делать обработку данных (служебную логику и т.д.) перед и после обработки запроса сервлетом.
> 
> Фильтр — это как секретарь, а сервлет – директор. Прежде чем документ попадет на стол директору, он пройдет через руки секретаря. И после того, как директор его подпишет, он снова попадет секретарю, уже как исходящая корреспонденция, например.

![[Pasted image 20240531233635.png]]

У фильтра так же есть методы `init()` и `destroy()`. Вместо метода `service()` у фильтра есть метод `doFilter()`. И даже есть свой класс `FilterConfig`. Фильтр также добавляется в сервлет в файле `web.xml` или же с помощью аннотации `@WebFilter`.

#### В чем же отличие сервлета и фильтра?
Фильтров может быть несколько, и они последовательно обрабатывают запрос (и ответ). Они объединены в так называемую цепочку — и для них даже есть специальный класс `FilterChain`.
После обработки запроса в методе `doFilter()` нужно вызвать метод `doFilter()` следующего фильтра в цепочке (а именно - вызвать `filterChain.doFilter(req, resp);`).

```java
public class MyFilter implements Filter {

	public void init(FilterConfig arg0) throws ServletException {
	// инициализируем какие-л ресурсы
	}
  
    @Override  
    public void doFilter(ServletRequest servletRequest,
					     ServletResponse servletResponse,
					     FilterChain filterChain) throws IOException,
													     ServletException {  
        log.info("Filter start");
		// логика до обработки запроса

	    // происходит вызов следующего фильтра в цепочке,
	    // если это последний фильтр - вызывается сервлет
        filterChain.doFilter(requestWrapper, responseWrapper);
        // логика после обработки запроса
        log.info("Filter finish");  
    }

	public void destroy() {
	// закрываем какие-л ресурсы
	}
}
```

### Example
#### Структура проекта
![[Pasted image 20240601004144.png | 400]]
- `webapp` обязательно должна лежать в корне проекта(на уровне с пакетом `java` и `resources`) и включать в себя `WEB-INF`.
- `webapp` - как classpath для сервера приложений, все относительные пути строятся относительно нее.
- в `WEB-INF` должен лежать конфигурационный файл для сервера приложений (он же дискриптор развертывания) `web.xml` .

#### web.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">


    <!--Index page for app-->
    <servlet-mapping>
        <servlet-name>GetStartPageServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
    <servlet>
        <servlet-name>GetStartPageServlet</servlet-name>
        <servlet-class>ru.javavision.servlet.GetIndexPageServlet</servlet-class>
    </servlet>

	<!--Encoding filter UTF-8 for all requests-->
    <filter>
        <filter-name>MyFilter</filter-name>
        <filter-class>ru.javavision.servlet.MyFilter</filter-class>
    </filter>

    <filter-mapping>
        <filter-name>MyFilter</filter-name>
        <url-pattern>/</url-pattern>
    </filter-mapping>
</web-app>
```

#### Servlet
#### Сервлет с методом doGet()
```java
public class GetIndexPageServlet extends HttpServlet {

    private Map<Integer, User> users;

    @Override
    public void init() throws ServletException {

        final Object users = getServletContext().getAttribute("users");

        if (users == null || !(users instanceof ConcurrentHashMap)) {
            throw new IllegalStateException("You're repo does not initialize!");
        } else {
            this.users = (ConcurrentHashMap<Integer, User>) users;
        }
    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {

        req.setAttribute("users", users.values());
        req.getRequestDispatcher("/WEB-INF/view/index.jsp").forward(req, resp);
    }
}
```