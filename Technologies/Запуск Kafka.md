#kafka
## Локальный запуск
### Распаковка kafka на windows  
Распакованную директорию с kafka необходимо разместить в корне диска или близко к корню.  
Иначе из за длинного пути к дирректории с кафкой будет вылетать ошибка: `Слишком длинная входная строка.  
Ошибка в синтаксисе команды.`  
  
### Запуск kafka  
1. Запускаем ZooKeeper  
`bin/windows/zookeeper-server-start.bat config/zookeeper.properties`  
2. Запускаем Kafka  
`bin/windows/kafka-server-start.bat config/server.properties`  
  
### Просмотр что отправляется в конкретный топик  
Команда:`bin/windows/kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic data-temperature --from-beginning`  
- `--bootstrap-server` указываем адрес bootstrap сервера  
- `--topic` указываем имя топика, сообщения из которого мы будем просматривать