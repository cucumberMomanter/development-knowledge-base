Обертки осуществляющие кеширование тела `Request`/`Response`.

### Материалы
[HttpServletRequest](https://www.baeldung.com/spring-reading-httpservletrequest-multiple-times)
[HttpServletResponse](https://www.baeldung.com/spring-boot-filter-response-body)

### ContentCachingRequestWrapper
> Этот класс предоставляет метод `getContentAsByteArray()` для многократного чтения тела.

Однако у этого класса есть ограничение: **мы не можем многократно читать тело, используя `getInputStream()` и `getReader()` методы.**
<font style="color:red">НО</font> можно расширить класс `HttpServletRequestWrapper`.

#### Реализация HttpServletRequestWrapper
> Класс `HttpServletRequestWrapper` имеет два абстрактных метода `getInputStream()` и `getReader()`.
> Мы переопределим оба этих метода и создадим новый конструктор.

```java
public class CachedBodyHttpServletRequest extends HttpServletRequestWrapper {
	
	private final byte[] cachedBody;
	
	public CachedBodyHttpServletRequest(HttpServletRequest request)
														throws IOException {
		super(request);
		InputStream requestInputStream = request.getInputStream();
		this.cachedBody = StreamUtils.copyToByteArray(requestInputStream);
	}

	@Override
	public ServletInputStream getInputStream() {
		return new CachedBodyServletInputStream(this.cachedBody);
	}

	@Override
	public BufferedReader getReader() {
		ByteArrayInputStream byteArrayInputStream =
								new ByteArrayInputStream(this.cachedBody);
		return new BufferedReader(new InputStreamReader(byteArrayInputStream));
	}

}
```

> Метод `getInputStream()` возвращает **ServletInputStream**, его так же необходимо реализовывать (пример ниже).

#### Реализация ServletInputStream
В этом классе мы создадим новый конструктор, а также переопределим методы:
- `isFinished()`
  Указывает, остались ли в `InputStream` данных для чтения или нет.
  Возвращает `true`, когда все байты вычитаны.
- `isReady()`
  Этот метод указывает, готов ли `InputStream` к чтению или нет.
  Поскольку мы уже скопировали `InputStream` в массив байтов, мы вернем `true`, чтобы указать, что он всегда доступен.
- `read()`

```java
public class CachedBodyServletInputStream extends ServletInputStream {

	private final ByteArrayInputStream cachedBodyInputStream;
	
	public CachedBodyServletInputStream(byte[] cachedBody) {
		this.cachedBodyInputStream = new ByteArrayInputStream(cachedBody);
	}

	@Override
	public int read() {
		return cachedBodyInputStream.read();
	}

	@Override
	public boolean isFinished() {
		return cachedBody.available() == 0;
	}
	
	@Override
	public boolean isReady() {
		return true;
	}

	@Override
	public void setReadListener(ReadListener readListener) {
	    // В примере реализация не определена,
	    // но метод обязательный к переопределению
    }
}
```

### ContentCachingResponseWrapper
> Этот класс предоставляет метод `getContentAsByteArray()` для многократного чтения тела.

#### Использование ContentCachingResponseWrapper в фильтре
> Класс-оболочка позволяет нам обернуть `HttpServletResponse` для кэширования содержимого тела ответа и вызвать `doFilter()` для передачи запроса следующему фильтру.
```java
@Override
public void doFilter(ServletRequest servletRequest,
					 ServletResponse servletResponse,
					 FilterChain filterChain) throws IOException,
													 ServletException {
	ContentCachingResponseWrapper responseCacheWrapperObject =
		new ContentCachingResponseWrapper(
									(HttpServletResponse) servletResponse);
									
	filterChain.doFilter(servletRequest, responseCacheWrapperObject);
	
	byte[] responseBody = responseCacheWrapperObject.getContentAsByteArray();'
	//что то делаем с body
	
	responseCacheWrapperObject.copyBodyToResponse();
}	
```
- Мы не должны забывать о вызове здесь `doFilter()`. В противном случае входящий запрос не перейдет к следующему фильтру в цепочке spring filter , и приложение не обработает запрос так, как мы ожидали.
- Фактически, **не вызов функции `doFilter()` является нарушением спецификации сервлета**.

<font style="color:red">КРАЙНЕ ВАЖНО</font> **вызвать `copyBodyToResponse()` перед выходом из метода `doFilter()`.
В противном случае клиент не получит полный ответ**.