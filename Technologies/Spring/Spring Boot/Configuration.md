#spring_boot #config
### Приоритет применения application.properties
#application_properties
1. <font style="color:green">application-local.properties</font> - настройки в этом файле имеют наивысший приоритет и перекрывают любые другие настройки.  
2. <font style="color:green">application.properties</font> - настройки из этого файла применяются в случае, если их нет в <font style="color:green">application-local.properties</font>.  
3. Другие файлы настроек, такие как <font style="color:green">application-{profile}.properties</font>, где `{profile}` - профиль приложения, имеют приоритет ниже и применяются в зависимости от активированного профиля.  
  
Если в <font style="color:green">application-local.properties</font> нет необходимых настроек, то будут использоваться настройки из application.properties.