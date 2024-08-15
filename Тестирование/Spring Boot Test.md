### Материалы
[Тестовые аннотации](https://javarush.com/quests/lectures/questspring.level02.lecture19)
### Тезисы
#### Пример теста записи сообщения в kafka
```java
@DirtiesContext
@EmbeddedKafka(partitions = 1, bootstrapServersProperty = "spring.kafka.bootstrap-servers")
@SpringBootTest(classes = {BrokerClickHouseLog.class, TestController.class, KafkaConsumer.class})
@AutoConfigureMockMvc
public class RequestLoggingTest {
  
    private static final String PATH = "/api/test";  
  
    @Autowired  
    private MockMvc mockMvc;  
  
    @Autowired  
    private KafkaConsumer consumer;  
  
    @Test  
    void contextLoads() {  
    }  
    @Test  
    void logKafkaWriting() throws Exception {  
        final String responseBody = "get-test";  
  
        mockMvc.perform(get(PATH))
                .andDo(print())
                .andExpect(content().string(responseBody));
  
        consumer.getLatch().await(5000, TimeUnit.MILLISECONDS);
        Message message = consumer.getMessage();
  
        assertThat(consumer.getLatch().getCount()).isEqualTo(0L);
        assertThat(message.getRequestMethod()).isEqualTo("GET"); 
        assertThat(message.getRequestPath()).isEqualTo(PATH);
        assertThat(message.getResponseBody()).isEqualTo(responseBody);
    }
}
```

##### @SpringBootTest
> `@SpringBootTest` используется в тестовых классах в Spring Framework для указания, что класс теста нужно выполнить в контексте Spring приложения.

`@SpringBootTest(classes = {BrokerClickHouseLog.class, TestController.class, KafkaConsumer.class})` 
Добавление в тестовый контекст спринга через параметр `classes`:
- конфигурационных классов
- отдельных бинов

Класс `BrokerClickHouseLog.class`, помеченный аннотацией `@SpringBootApplication`, добавляет контекст всего Spring boot app (или даже стартера) в тестовый контейнер.
Но обычно его указание не требуется и спринг сам понимает от куда взять контекст.
При этом он может располагаться в тестовом пакете:
![[Pasted image 20240616132614.png|400]]

#####  @DirtiesContext
> `@DirtiesContext` указывает, что базовый `ApplicationContext` из Spring был "загрязнен" во время выполнения теста (то есть тест изменил или испортил его каким-то образом – например, изменив состояние бина-одиночки) и его необходимо закрыть и создать новый.

Установив `MethodMode` или `ClassMode` , **мы можем контролировать, когда Spring помечает контекст для закрытия**. По дефолту установлено:

- Для уровня метода: `@DirtiesContext(methodMode = MethodMode.AFTER_METHOD)`
- Для уровня класса: `@DirtiesContext(classMode = MethodMode.AFTER_CLASS)`

##### @AutoConfigureMockMvc
> `@AutoConfigureMockMvc` аннотация из Spring Boot Test, которая позволяет автоматически настроить `MockMvc` для интеграционного тестирования контроллеров в Spring приложении.

`MockMvc` - это специальный инструмент, который позволяет тестировать контроллеры без необходимости запуска всего приложения. Он эмулирует HTTP запросы к контроллерам и позволяет проверять их поведение.

При использовании аннотации `@AutoConfigureMockMvc`, Spring Boot автоматически настраивает MockMvc и делает его доступным для использования в тестах.

##### @EmbeddedKafka
> `@EmbeddedKafka` - это аннотация из Spring Kafka Test, которая предназначена для интеграционного тестирования компонентов, связанных с Apache Kafka, в Spring Boot приложениях.

При использовании аннотации `@EmbeddedKafka`, Spring автоматически создает встроенный Kafka брокер в памяти для использования во время выполнения интеграционных тестов. Это позволяет тестировать функциональность, связанную с обработкой сообщений через Apache Kafka, без необходимости настройки и использования реального Kafka брокера.

```java
`@EmbeddedKafka(partitions = 1,
				bootstrapServersProperty = "spring.kafka.bootstrap-servers")`

```
Св-ва, поставляемые `@EmbeddedKafka`:
- `count`: количество встроенных Kafka брокеров;
- `partitions`: количество разделов (partitions) в каждом созданном брокере;
- `topics`: топики, которые будут созданы на встроенных брокерах;
- `bootstrapServersProperty`: установка адресов встроенных кафка брокеров