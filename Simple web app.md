### Создание проекта
- [[Создание проекта через maven archetype]]
#### Добавляем директории/пакеты в иерархию классов
Иерархия проекта сразу после создания через <font style="color:green">maven-archetype-webapp</font>:
![[Pasted image 20240107003501.png]]
Иерархия проекта после модификации иерархии:
![[Pasted image 20240107005520.png]]
Последовательность изменений:
- В `main` добавляем директорию <font style="color:cyan">java</font> и маркируем ее как <font style="color:red">Source root</font>.
- В `java` будет находится вся основная логика нашего приложения. Дальнейшая иерархия пакетов в директории `java` будет зависеть от конкретного проекта и ваших предпочтений.
  В настоящем проекте иерархия пакетов будет следующая <font style="color:cyan">com.simple.maven.webapp</font>.
- В `WEB-INF` добавляем директорию <font style="color:cyan">view</font>, в ней будут храниться все наши view (визуальные представления веб страниц).
- Удаляем `index.jsp`, он нам не понадобится.

### Конфигурация проекта
#### Настройка maven pom.xml
- Добавляем необходимые `<propties>`
- Добавляем зависимости `<dependencies>`
- Добавляем плагин для сборки war файла

Итоговый pom.xml выглядит следующим образом:
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>org.example</groupId>
  <artifactId>maven-java-webapp</artifactId>

  <packaging>war</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>maven-java-webapp Maven Webapp</name>
  <url>http://maven.apache.org</url>

  <properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>6.0.11</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-core</artifactId>
      <version>6.0.11</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-beans</artifactId>
      <version>6.0.11</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>6.0.11</version>
    </dependency>

    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>jstl</artifactId>
      <version>1.2</version>
    </dependency>
  </dependencies>

  <build>
    <finalName>maven-java-webapp</finalName>

    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-war-plugin</artifactId>
        <version>3.2.3</version>
      </plugin>
    </plugins>
  </build>
</project>

```
- Теги в `pom.xml`
  - `<package>` - указывается формат файла, в который необходимо собрать проект (war, jar)
- Cекция `<properties>`
  В данной секции задаются переменные.
  Общий вид переменной:
  > <font style="color:red"><имя-переменной></font><font style="color:green">значение</font><font style="color:red"></имя-переменной></font>
  
  Добавим следующие переменные:
  - <font style="color:violet">maven.compiler.source</font> (Java версия исходников)
  - <font style="color:violet">maven.compiler.target</font> (версия Java-машины, под которую нужно скомпилировать классы)
  - <font style="color:violet">project.build.sourceEncoding</font> (указывается кодировка Java-файлов. Сейчас практически все исходники хранятся в кодировке `UTF-8`)
  
- Секция `<dependencies>`
  Все зависимости можно найти в [maven repository](https://mvnrepository.com/)
  Нам понадобятся следующие зависимости:
  - Spring Context
  - Spring Core
  - Spring Beans
  - Spring Web MVC
  - JSTL (необходима для создания view)
> Зависимости из одного проекта (например из Spring) должны быть все одной версии. При использовании разных версий могут возникнуть ошибки.
- Секция `<build>`
  Данная секция не является обязательной, но добавляет гибкости в настройку более-менее сложного проекта.
  Разберем несколько тегов секции `<build>`:
  - `<finalName>` - устанавливает имя, создаваемого maven файла сборки в фазе **package**
  - `<plagins>` - содержит в себе все подключаемые maven плагины
  - `<plagin>` - содержит  себе конкретный maven плагин (Например плагин для создания war-файла `maven-war-plugin`)
#### Конфигурация web.xml
При создании проекта уже был создан дефолтный <font style="color:green">web.xml</font>, по адресу `/src/main/webapp/WEB-INF/web.xml`.
Полностью заменяем его содержимое на следующее:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         id="WebApp_ID" version="3.1">

  <display-name>mvc-test</display-name>

  <absolute-ordering />

  <servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>/WEB-INF/applicationContext.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>

  <servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>

</web-app>
```
Файл <font style="color:green">web.xml</font> нужен для конфигурации <font style="color:orange">DispatcherServlet</font> (это Front Controller).
Его конфигурация осуществляется в тэгах `<servlet>`. 
Описание вложенных тегов:
- В `<servlet-name>` указываем имя этого <font style="color:orange">DispatcherServlet</font>, т.е. имя по которому будет идти обращение к Front Controller.
- В `<servlet-class>` указываем класс, который отвечает за этот <font style="color:orange">DispatcherServlet</font>.
- В `<param-value>` указываем, где находится наш файл **Application Context** (указываем путь). Этот файл отвечает за конфигурацию Spring приложения. Здесь мы указываем значение к тэгу выше `<param-name>`<font style="color:red">contextConfigLocation</font>`</param-name>`.
- В `<servlet-mapping>` прописываем URL-адрес для <font style="color:orange">DispatcherServlet</font>. Т.е. при наборе какого URL-адреса http запрос будет поступать на наш <font style="color:orange">DispatcherServlet</font>.
- В `<servlet-name>` указываем имя этого <font style="color:orange">DispatcherServlet</font>. (Такое же, как и было выше)
- В `<url-pattern>` прописываем слеш <font style="color:red">/</font> . Слеш означает, что какой бы адрес мы не прописали, по любому каждый http request придет на наш <font style="color:orange">DispatcherServlet</font>.

#### Конфигурация applicationContext.xml
В пакете **WEB-INF** создаем новый файл <font style="color:green">applicationContext.xml</font>.
И наполняем его следующим содержимым:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="
		http://www.springframework.org/schema/beans
    	http://www.springframework.org/schema/beans/spring-beans.xsd
    	http://www.springframework.org/schema/context
    	http://www.springframework.org/schema/context/spring-context.xsd
    	http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <context:component-scan base-package="com.simple.maven.webapp"/>

    <mvc:annotation-driven/>

    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/view/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>

```

Описание тегов:
- В `<context:component-scan base-package="….."/>` указываем пакет, в котором будет производиться сканирование и поиск компонентов. 
- `<mvc:annotation-driven/>` - это добавление поддержки форматирования, валидации и различных преобразований.
- Так же прописываем **bean** <font style="color:orange">ViewResolver</font>, т.е. прописываем то, как мы будет работать с нашими представлениями, с нашими View. В `class=”…”` мы прописываем уже готовый класс (тот, который указан в примере) и этот класс будет ответственен за весь этот процесс работы со View. Так же мы указываем <font style="color:red">prefix</font> и <font style="color:red">suffix</font>, чтобы обращаться к нашему View просто по имени, а <font style="color:red">prefix</font> и <font style="color:red">suffix</font> добавят путь и расширение к файлу с нашим View.

### Реализация Spring MVC web-приложения
