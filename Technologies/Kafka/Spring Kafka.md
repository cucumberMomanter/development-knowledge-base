### Конфигурирование Consumer
#### Варианты конфигурации
Следующие конфигурации `ComsumerFactory` идентичны:
1. Конфигурирование с передачей десериализаторов в конструктор `ConsumerFaktory`
   ![[Pasted image 20240318140651.png]]
   - В параметр `JsonDeserializer` передается класс, в который необходимо десериализовать JSON.
2. Конфигурирование с указанием десериализаторов в `consumerProperties`
   ![[Pasted image 20240318131655.png]]
   - `*_DESERIALIZER_CLASS_CONFIG` - указывается класс десериализатора.
   - `*_DEFAULT_TYPE` - указывается класс, в который должны преобразоваться данные после десериализации.

##### ConsumerProperties
> это коллекция пар `key -> value`, с данными конфигурации kafka consumer.

При инициализации Spring Boot приложения создается автоматически и хранит в себе, прописанные в `application.properties`, значения пар для конфигурации kafka consumer.
Получить `consumerProperties` можно из бина `KafkaProperties` :
```java
Map<String, Object> props = kafkaProperties.buildConsumerProperties();
```
<font style="color:red">Примечание:</font>
При создании `ConsumerFactory`, в параметр конструктора, передаются `consumerProperties`.
И если мы сделаем новый ассоциативный массив для `consumerProperties`, добавим туда нужные нам пары конфигурации и передадим в конструктор `ConsumerFactory`, то указанные в `application.properties` пары туда не попадут:
```properties
spring.kafka.bootstrap-servers=localhost:29092  
spring.kafka.instrument_data_reader.group_id=instrumentDataReaderGroupId
```
В этом случае придется вручную добавлять все необходимые пары конфирурации при создании `ConsumerFactory` в ново-созданный `consumerProperties`(<font style="color:red">props</font>):
![[Pasted image 20240318164502.png]]

Или же можно получить заполненный при инициализации Spring Boot app `consumerProperties` из бина `KafkaProperties`, дополнить его нужными значениями пар конфигурации и передать в конструктор `ConsumerFactory`:
![[Pasted image 20240318164339.png]]

#### Пример излишней (двойной) конфигурации десериализатора
В такой, двойной, конфигурации <font style="color:red">десериализации</font> нет смысла:
![[Pasted image 20240318141645.png]]
Так как приоритет будет отдан десериализаторам, переданным в параметр конструктора `ConsumerFactory` (на примере выше `DefaultKafkaConsumerFactory`).

##### Процесс создания и конфигурации десериализаторов при создании KafkaConsumer
Создание `KafkaConsumer` в `ConsumerFactory` :
1.  В `ConsumerFactory.createRawConsumer(..)` вызывается конструктор `KafkaConsumer`:
   ```java
	KafkaConsumer(ConsumerConfig config,
				  Deserializer<K> keyDeserializer,
				  Deserializer<V> valueDeserializer) {
			.
			.
	}
   ```
Значения параметров этого конструктора передаются из данных, переданных в конструктор `ConsumerFactory`:
![[Pasted image 20240318143201.png]]

2.  Если `keyDeserializer` и/или `valueDeserializer` :
   - `!= null` :
     - параметр `*_DESERIALIZER_CLASS_CONFIG` в `consumerProperties` игнорируется;
     - создается копия `consumerProperties`, в нее записываются данные из переданных десериализаторов в параметр `*_DESERIALIZER_CLASS_CONFIG`;
     - при этом далее переданный десериализатор не конфигурируется параметрами из `ConsumerConfig`(который создается на основе `consumerProperties`).
   - `null` :
     - создается десериализатор на основе значения в `*_DESERIALIZER_CLASS_CONFIG`;
     - созданный десериализатор конфигурируется на основе `ConsumerConfig`;
     - <font style="color:red">Для JsonDeserializer (а может и не только)</font>: Если не заполнить параметр `*_DEFAULT_TYPE` в `consumerProperties`, то при инициализации десериализатора не будет создан дефолтный `ObjectReader reader` (с помощью которого и осуществляется непосредственное преобразование данных).


