# ЛР 3. HA Postgres Cluster

**Задача:**
Развернуть и настроить высокодоступный кластер Postgres.

## Часть 1. Поднимаем Postgres

1. Создали директорию проекта, зашли в нее через Powershell, и создали там dockerfile.
2. Создали compose файл.
3. Создали postgres0.yml и postgres1.eml файлы.
4. Задеплоили

![image](media/4f280952-ab9b-47f3-beba-c2863e93ed5e%20(1).jfif)
![image](media/4f280952-ab9b-47f3-beba-c2863e93ed5e%20(1).jfif)
![image](media/1c350cdc-2ddc-4a9a-ba2b-6afc0ac28dd3.jfif)
![image](media/91f14a0d-aadb-44df-a2b9-e22f0eebfede.jfif)
![image](media/4c7090cd-3ce8-4d5c-b418-f5eab2cc3d14.jfif)
![image](media/c2030d7e-d5bd-4b8b-81b3-62f42e8c1e9b.jfif)

## Часть 2. Проверяем репликацию

Для упрощения дальнейших проверок и избежания проблем с фаерволом при внешних
подключениях, мы перенесли всю лабораторную на виртуальную машину. Пришлось установить
docker и docker-compose, но ничего не поменялось. Мы заново произвели

```bash
docker-compose up -d
docker-compose ps 
```

По итогу мы получили 3 работающих контейнера. Далее мы подключились к master и slave
базам данных и проверили репликацию и доступ к записи.

**Созданная таблица реплицировалась с мастера на слэйв и унаследовала вставленные данные**

![photo_2024-12-11 15.19.01.jpeg](media/photo_2024-12-11%2015.19.01.jpeg)

**pg-slave выдает ошибку при попытке записи данных**
![photo_2024-12-11 15.19.02.jpeg](media/photo_2024-12-11%2015.19.02.jpeg)

## Часть 3. Делаем среднего роста высокую доступность

Мы добавили в `docker-compose.yml` блок для haproxy, который сел на (традиционный для
бд) порт 5432. Далее мы создали конфиг файл (занятно, что он не работал, пока мы не
добавили новую пустую строку в конце файла) и перезапустили docker-compose. При
подключении я создал новую таблицу, заполнил ее и проверил репликацию на обоих узлах.

**Таблица отображается на всех 3 подключениях**

![photo_2024-12-11 15.19.03.jpeg](media/photo_2024-12-11%2015.19.03.jpeg)

**Внесли и прочитали данные из таблицы**

![photo_2024-12-11 15.19.06.jpeg](media/photo_2024-12-11%2015.19.06.jpeg)

![photo_2024-12-11 15.19.04.jpeg](media/photo_2024-12-11%2015.19.04.jpeg)


Все конфигурационные файлы можно посмотреть в папке files, это копия файлов с 
удаленного сервера. 

## Задание

Любым способом выключаем доступ до ноды, которая сейчас является мастером (например, через docker stop). Некоторое время ждем, после этого анализируем логи и так же пытаемся считать/записать что-то в БД через entrypoint подключение. Затем необходимо расписать, получилось или нет, а так же объяснить, что в итоге произошло после принудительного выключения мастера (со скриншотами)

```sh
root@oo5:~# cd lab-3/
root@oo5:~/lab-3# docker-compose ps  # проверяем крутящиеся контейнеры
       Name                      Command               State                                         Ports                                       
-------------------------------------------------------------------------------------------------------------------------------------------------
pg-master             docker-entrypoint.sh patro ...   Up      0.0.0.0:5433->5432/tcp,:::5433->5432/tcp, 8008/tcp                                
pg-slave              docker-entrypoint.sh patro ...   Up      0.0.0.0:5434->5432/tcp,:::5434->5432/tcp, 8008/tcp                                
postgres_entrypoint   docker-entrypoint.sh hapro ...   Up      0.0.0.0:5432->5432/tcp,:::5432->5432/tcp, 0.0.0.0:7000->7000/tcp,:::7000->7000/tcp
zoo                   /etc/confluent/docker/run        Up      0.0.0.0:2181->2181/tcp,:::2181->2181/tcp, 2888/tcp, 3888/tcp                      
root@oo5:~/lab-3# docker exec -it pg-slave patronictl -c /postgres1.yml list  # проверяем точку зрения слэйва на текущую ситуацию
+ Cluster: my_cluster (7447122583266881560) --+----+-----------+
| Member      | Host      | Role    | State   | TL | Lag in MB |
+-------------+-----------+---------+---------+----+-----------+
| postgresql0 | pg-master | Leader  | running |  3 |           |
| postgresql1 | pg-slave  | Replica | running |  4 |         0 |
+-------------+-----------+---------+---------+----+-----------+
root@oo5:~/lab-3# docker stop pg-master  # Убиваем мастера
pg-master
root@oo5:~/lab-3# docker-compose ps
       Name                      Command                State                                           Ports                                       
----------------------------------------------------------------------------------------------------------------------------------------------------
pg-master             docker-entrypoint.sh patro ...   Exit 137                                                                                     
pg-slave              docker-entrypoint.sh patro ...   Up         0.0.0.0:5434->5432/tcp,:::5434->5432/tcp, 8008/tcp                                
postgres_entrypoint   docker-entrypoint.sh hapro ...   Up         0.0.0.0:5432->5432/tcp,:::5432->5432/tcp, 0.0.0.0:7000->7000/tcp,:::7000->7000/tcp
zoo                   /etc/confluent/docker/run        Up         0.0.0.0:2181->2181/tcp,:::2181->2181/tcp, 2888/tcp, 3888/tcp                      
root@oo5:~/lab-3# docker exec -it pg-slave patronictl -c /postgres1.yml list
+ Cluster: my_cluster (7447122583266881560) +----+-----------+
| Member      | Host     | Role   | State   | TL | Lag in MB |
+-------------+----------+--------+---------+----+-----------+
| postgresql1 | pg-slave | Leader | running |  4 |           |
+-------------+----------+--------+---------+----+-----------+
```

Слейв нода теперь имеет роль мастера.

В файлы postgres0/1.yml была добавлена строка 

```yml
bootstrap:
  dcs:
    synchronous_mode_strict: false
```

## Ответы на вопросы:

1) *Порты 8008 и 5432 вынесены в разные директивы, expose и ports. По сути, если записать
   8008 в ports, то он тоже станет exposed. В
   чем разница?*

- Порты, вынесенные в expose, доступны только для других контейнеров, а вынесенные в
  ports доступны извне.

2) *При обычном перезапуске композ-проекта, будет ли сбилден заново образ? А если
   предварительно отредактировать файлы
   postgresX.yml? А если содержимое самого Dockerfile? Почему?*

- Нет, во всех случаях образ не будет сбилден заново, потому что образы в docker
  кэшируются для оптимизации.
  
