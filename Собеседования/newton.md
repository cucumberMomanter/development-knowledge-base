> live coding проходит через этот сервис [https://code.yandex-team.ru/](https://code.yandex-team.ru/)
### Задачки с решением
1. Понять, что хотел сделать разработчик. Объяснить что произойдет. В чем проблема, как исправить.  
```java
@Transactional  
public void changeName(long id, String name) {  
    User user = userRepository.getById(id);  
    user.setName(name);
    
    if (StringUtils.isNotEmpty(name)) {  
        userRepository.save(user);
    }  
}
```
   
   > Пояснение
   >  - Разработчик хотел обновлять имя у user, только в том случае, когда переданный в параметр метода name не пустой.
   >  - `user.setName(name)` - выполняется всегда, при любом сценарии. А т.к. транзакция на момент исполнения этого кода открыта - изменения, после комита, попадут в базу вне зависимости от вызова : `userRepository.save(user)`.
   >  - Как минимум, `user.setName(name)` необходимо переместить внутрь if-а.
   
2. Какая транзакция будет открыта при исполнении кода в `method2()`, когда он вызывается из `method1()`? Как сделать, чтобы обе транзакции сработали?

```java
@Service  
public class MyService {

    @Transactional
    public void method1() {
        //some code ...
        method2();
    }
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void method2() {
        //some code ...
    }
}
```

> Поясение
> - При использовании транзакций в Spring, вся логика управления транзакциями выполняется через прокси-объект, который оборачивает целевой бин (в данном случае `MyService`). Когда метод <font style="color:violet">method1()</font> вызывает метод <font style="color:violet">method2()</font> внутри себя, это является внутренним вызовом метода (<font style="color:red">this.method2()</font>), который не пройдет через новый прокси-объект.
> Поэтому Spring не создает новую транзакцию для <font style="color:violet">method2()</font> внутри <font style="color:violet">method1()</font>, так как оба метода вызываются в рамках одного и того же бина `MyService`. Новая транзакция с типом <b>REQUIRES_NEW</b> будет создана только при вызове метода <font style="color:violet">method2()</font> извне `MyService`, например, из другого компонента или сервиса.
> Таким образом, в данном контексте важно понимать, что внутренние вызовы методов внутри одного и того же бина <b>не будут обрабатываться через новый прокси-объект</b> для управления транзакциями, и поэтому новая транзакция не будет создана для <font style="color:violet">method2()</font> внутри <font style="color:violet">method1()</font>.
> - Для того, чтобы при вызове <font style="color:violet">method2()</font> внутри метода <font style="color:violet">method1()</font> создалась новая транзакция можно внедрить зависимость бина `MyService` сам в себя (Например, через `ObjectFactory<MyService>`) и сделать вызов <font style="color:violet">method2()</font> на этом бине.

3. Написать реализацию сервиса, отправляющего оповещения всем Notifier'ам
```java
   public interface Notifier {  
    void notify(MessageDto messageDto);  
}

@Service  
public class EmailNotifier implements Notifier {  
    @Override  
    public void notify(MessageDto messageDto) {  
        ...some code here...  
    }  
}  

@Service  
public class TelegramNotifier implements Notifier {  
    @Override  
    public void notify(MessageDto messageDto) {  
        ...some code here...  
    }  
}
```

> Реализация:
```java
@Service  
public class NotificationService {  
   @Autowired  
   private List<Notifier> notifiers;  
  
   public void notifyAll(MessageDto messageDto) {  
     for (Notifier notifier : notifiers) {  
       notifier.notify(messageDto);  
     }  
   }  
}
```
> Пояснение:
> Данная запись внедряет зависимость ко всем реализациям <code>Notifier</code>, добавляя их в лист <font style="color:violet">notifiers</font>:
```java
@Autowired
private List<Notifier> notifiers;
```
> Дополнительный вопрос:
> Как с помощью задания `фичи` (признака) задать определенные каналы отправки уведомлений, в зависимости от положения `фичи`?
> 
> Решение а уровне application context:
> Через аннотацию Condition (например, через <font style="color:orange">@ConditionOnProperty</font> или другую) задать условие создания бинов всех реализаций Notifier:
```java
   public interface Notifier {  
    void notify(MessageDto messageDto);  
}

@Service
@ConditionalOnProperty(name = "feature.email.enabled", havingValue = "true")
public class EmailNotifier implements Notifier {  
    @Override  
    public void notify(MessageDto messageDto) {  
        ...some code here...  
    }  
}  

@Service
@ConditionalOnProperty(name = "feature.telegram.enabled", havingValue = "true")
public class TelegramNotifier implements Notifier {  
    @Override  
    public void notify(MessageDto messageDto) {  
        ...some code here...  
    }  
}
```
- <font style="color:orange">@ConditionalOnProperty</font> позволяет задавать условия для создания бина на основе наличия и значения определенного свойства в файле конфигурации приложения (`application.properties` или `application.yml`).

4. Создать таблицу для установления связи many-to-many (Join Table) между таблицами author  и book
```sql
create table author();  
create table book();  
  
create table author_book (  
author_id int,  
book_id int,  
PRIMARY KEY (author_id, book_id),  
FOREIGN KEY (author_id) references author,  
FOREIGN KEY (book_id) references book  
);
```

- Дополнительно требовалось написать JOIN TABLE.

### Задачки без решения
 1. Вот такой код, вызывается метод первый раз, параллельно вызывается второй. Во время первого выполнения данные обновляются, и второй раз уже проверка не должна пройти, по задумке автора) Но этого не происходит, и тело метода полностью выполняется 2 раза.
```java
    @Transactional  
    public void synchronized method() {//Проверяем условие  
        bool cond = repository.checkCondition();  
        if (!cond) {  
            return;  
        }  
        //...do some code…  
        //Обновляем условие, теперь оно вернет false  
        repository.updateConditionData();  
    }
```
 2. Подсчитать общее кол-во книг. Найти первого автора, у которого имя Павел, и вернуть все его книги.
```java
    public void count(List<Author> authors) {  
  
    }  
    
    @Data  
    @Builder(toBuilder = true)  
    public class Author {  
       private String name;  
       private List<String> bookNames;  
    }  
    
    Stream.iterate(0, i -> i + 1)  
            .peek(System.out::println)  
            .sorted()  
            .findFirst()  
            .ifPresent(System.out::println);
```
 3. 
```java
public class ExampleStream {  
  
    /*  
    Сколько времени займет выполнение данного метода, почему? Какой будет вывод в консоли?  
     */  
    public static void main(String[] args) {  
        List<Integer> integers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);  
        ExecutorService executor = Executors.newCachedThreadPool();  
  
        List<String> results = [integers.stream](https://integers.stream/ "https://integers.stream")()  
                .map(integer -> integer * 500)  
                .map(integer -> CompletableFuture.supplyAsync(() -> process(integer), executor))  
                .map(CompletableFuture::join)  
                .collect(Collectors.toList());  
          
        System.out.println(results);  
    }  
  
    @SneakyThrows  
    private static String process(Integer integer) {  
        System.out.println("integer " + integer + " is being processed");  
        Thread.sleep(integer);  
        System.out.println("integer: " + integer + " was processed");  
        return String.valueOf(integer);  
    }  
}
```
 4. Написать метод для выполнения запроса к authorService в 15 потоков.
```java
public class SomeClass {  
  
  private final AuthorService authorService;  
  
  public void process(List<Author> authors) {  
    List<ProcessResult> results = new ArrayList<>();  
    for(Author author : authors) {  
      ProcessResult result = authorService.process(author);  
      results.add(result);  
    }  
  }  
  
}  
  
//Во время апдейта индекса, он становится недоступен для чтения.  
@Service  
public class SomeService {  
    
  public String readFromIntex() {}  
  
  @Scheduler(cron = “0 */5 * * * *”)  
  public void updateIndex() {};  
  
}
```

### Вопросы
1. Equals and Hashcode  
2. HashMap  
3. Stream. Operation types. Intermediate, terminal  
4. Parallel stream, forkJoinPool  
5. Какую структуру данных использовать для самописного кэширования?  
ConcurrentHashMap<Key, SoftReference<>>  
6. ThreadPool  
7. ThreadLocal  
8. Spring. Context initialization  
9. Spring. Bean scopes:  
singleton, prototype, request, session, application, web socket  
10. Prototype into Singleton:  
@Lookup, ObjectFactory, ProxyMode = TARGET_CLASS  
11. Spring. @Controller, @RestController  
12. Spring. How to @Autowired works.  
13. Spring. @Transactional. How to works. Call method inside class.  
14. Hibernate. 1st level cache  
15. Hibernate. Dirty checking. Entity states. (transient, managed, detached or deleted state)  
16. Hibernate. Fetch types: Eager and Lazy. Which one to use and why.  
17. Hibernate. Pessimistic and Optimistic lock.  
18. Как найти дубли в поле email?  
19. Как найти уникальные email?  
20. Структура данных для индексов БД.  
21. Kafka. Общее описание  
22. Kafka. ConsumerGroup, Message key, Partition listener