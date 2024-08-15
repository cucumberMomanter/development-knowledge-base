#spring_boot #config
### Приоритет применения application.properties
#application_properties
1. <font style="color:green">application-local.properties</font> - настройки в этом файле имеют наивысший приоритет и перекрывают любые другие настройки.  
2. <font style="color:green">application.properties</font> - настройки из этого файла применяются в случае, если их нет в <font style="color:green">application-local.properties</font>.  
3. Другие файлы настроек, такие как <font style="color:green">application-{profile}.properties</font>, где `{profile}` - профиль приложения, имеют приоритет ниже и применяются в зависимости от активированного профиля.  
  
Если в <font style="color:green">application-local.properties</font> нет необходимых настроек, то будут использоваться настройки из application.properties.

### Связь application.properties(yaml) и java класса
> Для установления связи м/ду проперти файлом и проперти классом, используется аннотация `@ConfigurationProperties(prefix = "..")`

```properties
in.request.logging.logEnabled=true  
in.request.logging.kafkaEnabled=true  
in.request.logging.kafkaTopic=test-topic
```

```java
@ConfigurationProperties(prefix = "in.request.logging")  
public class LogStarterProperties {  
  
    private boolean logEnabled;  
    private boolean kafkaEnabled;  
    private String kafkaTopic;  
}
```

- с помощью `@ConfigurationProperties(prefix = "in.request.logging")` связываются (мапятся) поля класса `LogStarterProperties` с контентом секции `in.request.logging` в `application.properties`.
<br>

> Для выноса группы настроек во (вложенный) java класс необходимо соблюсти правила наименования группы настроек в `application.properties` и наименования поля, которое, будет хранить объект, содержащий эту группу настроек.

```properties
my-app.in-class.enabled=
my-app.in-class.name=

my-app.kafka.enabled=
my-app.kafka.topic=
```

```java
@ConfigurationProperties(prefix = "my-app")
public class AppProperties {
	private InnerClass inClass;
	private MyAppKafkaProperties kafka;
}
	
public class InnerClass {
	private boolean enabled;
	private String name;
}

public class MyAppKafkaProperties {
	private boolean enabled;
	private String topic;
}
```
> `in-class == inClass`
> `kafka == kafka`

Пример:
```properties
in.request.logging.logEnabled=true  
in.request.logging.kafka-config.enabled=true  
in.request.logging.kafka-config.topic=test-topic
```

```java
public class KafkaProps {
	private boolean enabled;
	private String topic;
}
```

```java
@ConfigurationProperties(prefix = "in.request.logging")  
public class LogStarterProperties {  
  
    private boolean logEnabled;  
    private KafkaProps kafkaConfig;  
}
```

### Добавление Property java класса в Spring container
```java
@Configuration  
@EnableConfigurationProperties(LogStarterProperties.class)  
public class LogStarterConfig {
@Bean  
public LogInterceptor getLogInterceptor(LogStarterProperties properties) {  
    return new LogInterceptor(properties.isKafkaEnabled());  
}
}
```
- с помощью `@EnableConfigurationProperties(LogStarterProperties.class)` мы говорим Spring, что необходимо загрузить проперти `LogStarterProperties.class` в контекст и далее через `@Autowired` его можно внедрить в любой другой класс.