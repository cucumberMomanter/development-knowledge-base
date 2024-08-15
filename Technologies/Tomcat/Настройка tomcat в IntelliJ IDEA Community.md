#tomcat
### Скачиваем tomcat
Скачать можно с [офф сайта](https://tomcat.apache.org/)
> **Важный момент**
> В <font style="color:red">9-х</font> версиях tomcat в качестве сервлетов используется <font style="color:violet">javax.servlet</font>
> Начиная с <font style="color:red">10-й</font> версии tomcat - <font style="color:violet">jakarta.servet</font>
> 
> В Spring Framework до <font style="color:red">5</font> версии включительно используются сервлеты <font style="color:violet">javax.servlet</font>
> Начина с <font style="color:red">6</font> версии Spring Framework - <font style="color:violet">jakarta.servet</font>
> 
> Используемые сервлеты в нашем web приложении должны быть такие же, как и в используемой версии tomcat. Иначе будет конфликт и приложение не поднимется.

![[Pasted image 20240105134618.png]]
1. В блоке Download выбираем версию tamcat
2. Из Core скачиваем zip архив
3. Скачанный архив распаковываем в любое удобное место на локальной машине

### Настройка IntelliJ IDEA
[Видео гайд](https://yandex.ru/video/preview/14183360484042116822)
1. В IDEA заходим в <font style="color:green">Settings -> Tools -> External Tools -> </font><font style="color:red">+</font>
   ![[Pasted image 20240105150756.png]]
2. В открывшемся окне заполняем следующие поля:
   ![[Pasted image 20240105151543.png]]
   1. Вводим имя tool's
   2. Указываем путь до файла <font style="color:violet">catalina</font>. Для windows с расширением <font style="color:red">.bat</font>, для linux - <font style="color:red">.sh</font> .
   3. Прописываем команду <font style="color:red">jpda run</font>
Tomcat настроен.

### Запуск tomcat сервера
1. Собираем war-файл (mvn clean package)
2. Копируем собранный war-файл в директорию <font style="color:green">webapps</font>, находящуюся в корне директории самого tomcat, которую мы скачали и разахивировали ранее. В общем виде путь выглядит так :
   > ~\apache-tomcat-<font style="color:red">version</font> \ <font style="color:green">webapps</font>
3. Запускаем tomcat сервер через External Tools в IDEA:
   > <font style="color:green">Tools -> External Tools -> выбираем нужный Tomcat сервер</font>

   ![[Pasted image 20240105153618.png]]

   