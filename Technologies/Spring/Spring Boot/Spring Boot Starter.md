#spring_boot #spring_boot_starter #starter
### Материалы
[Видео для Spring Boot](https://rutube.ru/video/d3b5236e675e73b72c0b1419847e47eb/?t=1510)
[Видео для Spring Boot версии 2.7 и ниже](https://www.youtube.com/watch?v=cjpQ-bu3_7Y&t=426s)
[Статья](https://struchkov.dev/blog/ru/create-spring-boot-starter/)

### Конфигурирование стартера дефолтными настройками
#### Автоматическое конфигурирование
> Чтобы дефолтные настройки автоматически подтянулись в приложение из Spring Boot Starter-а, файл с настройками должен называться `application.*`

<font style="color:red">НО</font>: файлы с настройками в стартере и в приложении, должны иметь разное расширение.
Например:
- в приложении: `application.properties`
- в стартере: `application.yaml`

Иначе, если эти файлы будут иметь одинаковое расширение, настройки со стартера автоматически не подтянутся в приложение.

#### Кастомное конфигурирование
> Для того, чтобы стартер всегда конфигурировался корректно, вне зависимости от расширения файла `application.*`, можно сделать кастомную загрузку конфигурации стартера.

<font style="color:red">ВАЖНО</font>: при этом название файла с настройками конфигурации должно быть кастомное (т.е. не `application.*`).

##### EnvironmentPostProcessor
> Пример с загрузкой дефолтной кофигурации стартера с расширением `yaml`.
> Для загрузки `properties` необходимо заменить загрузчик `YamlPropertySourceLoader` на `PropertiesPropertySourceLoader`.

```java
public class EnvPostProcessor implements EnvironmentPostProcessor {  
  
    private static final String STARTER_NAME = "starter-name";  
    private static final String CREATE_PROPERTY_FAILED =
	    String.format("Unable to create property source for the Spring Boot Starter [%s]", STARTER_NAME);  
  
    private final YamlPropertySourceLoader yamlLoader;
    //private final PropertiesPropertySourceLoader propertiesLoader;  
  
    public EnvPostProcessor() {  
        this.yamlLoader = new YamlPropertySourceLoader();  
    }  
  
    @Override  
    public void postProcessEnvironment(ConfigurableEnvironment environment,
									   SpringApplication application) {  
        var resource = new ClassPathResource("starter-name-default.yaml");  
        PropertySource<?> propertySource;  
  
        try {  
            propertySource = yamlLoader.load(STARTER_NAME, resource)
					.stream()  
                    .findFirst()  
                    .orElseThrow(() ->new NullPointerException(
								                CREATE_PROPERTY_FAILED));  
        } catch (Exception e) {  
            throw new RuntimeException(e);  
        }  
  
        environment.getPropertySources().addLast(propertySource);  
    }  
}
```

##### @PropertySource
[PropertySource и YAML](https://for-each.dev/lessons/b/-spring-yaml-propertysource)
[Свойства с Spring и Spring Boot](https://for-each.dev/lessons/b/-properties-with-spring)

> `@PropertySource("classpath:default.properties")` - регистрирует файл свойств в среде Spring app.
> После регистрации свойства доступны будут во всем приложении.
> `@PropertySource` вешается на конфигурационный класс или любой бин (класс помеченный `@Component`).

<font style="color:red">ВАЖНО</font>: на конфигурационном классе c `@PropertySource` не должно быть условий создания с фича флагами из регистрируемого файла (из остальных можно), иначе аннотация @PropertySource не сработает и конфиг класс тоже.
Пример такого кейса:
```java
@Configuration
@PropertySource(value = "classpath:default.properties")
@ConditionalOnProperty(value = "my-app.enabled", havingValue = "true")  
public class MyAppConfig {
// my-app.enabled из файла default.properties.
// в этом случае файл свойств default.properties не заригистрируется
// конфигурационный класс MyAppConfig тоже не будет создан
}
```

> `@EnableConfigurationProperties(MyAppProperties.class)` - добавляет бин `property-класса` в Spring container.
> Вешается на конфигурационный класс или любой бин (класс помеченный `@Component`).

<font style="color:red">ПРИМЕЧАНИЕ</font>: Если на классе висит **фича флаг** на его создание (напр., `@ConditionalOnProperty()`) из уже добавленного файла со свойствами **со значением `false`**, аннотации `@PropertySource` и `@EnableConfigurationProperties` не сработают.

###### @PropertySource и YAML
> **По умолчанию** **`@PropertySource`** **не загружает файлы YAML**
> 
> Начиная с Spring 4.3, `@PropertySource` поставляется с атрибутом `factory .` Мы можем использовать его, чтобы **предоставить собственную реализацию** **`PropertySourceFactory`** **, которая будет обрабатывать файлы YAML**

```java
@Configuration
@PropertySource(value = "classpath:default.yml",
				factory = YamlPropertySourceFactory.class)
public class MyAppConfig {}
```

```java
public class YamlPropertySourceFactory implements PropertySourceFactory {
  
    @Override  
    public PropertySource<?> createPropertySource(String name,
											      EncodedResource encResource) {
        YamlPropertiesFactoryBean factory = new YamlPropertiesFactoryBean();  
        factory.setResources(encResource.getResource());  
  
        Properties properties = factory.getObject();  
  
        if (properties == null) {  
            throw new NullPointerException(String.format("Unable to create property source [%s]. Reason: yaml resources is null", name));  
        }
        
        String filename = encResource.getResource().getFilename();
  
        if (filename == null) {  
            throw new NullPointerException(String.format("Unable to create property source [%s]. Reason: resource filename is null", name));  
        }  
  
        return new PropertiesPropertySource(filename, properties);  
    }  
}
```