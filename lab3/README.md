# ЛР 3. HA Postgres Cluster

**Задача:** 
Развернуть и настроить высокодоступный кластер Postgres.

## Часть 1. Поднимаем Postgres
1. Создали директорию проекта, зашли в нее через Powershell, и создали там dockerfile.
2. Создали compose файл.
3. Создали postgres0.yml и postgres1.eml файлы.
4. Задеплоили
   
   ![image](https://github.com/Mayonnaise-Slap/ITMO-ICT-CompNet-2024/blob/lab3/lab3/4f280952-ab9b-47f3-beba-c2863e93ed5e%20(1).jfif)
   ![image](https://github.com/Mayonnaise-Slap/ITMO-ICT-CompNet-2024/blob/lab3/lab3/4f280952-ab9b-47f3-beba-c2863e93ed5e%20(1).jfif)
   ![image](https://github.com/Mayonnaise-Slap/ITMO-ICT-CompNet-2024/blob/lab3/lab3/1c350cdc-2ddc-4a9a-ba2b-6afc0ac28dd3.jfif)
   ![image](https://github.com/Mayonnaise-Slap/ITMO-ICT-CompNet-2024/blob/lab3/lab3/91f14a0d-aadb-44df-a2b9-e22f0eebfede.jfif)
   ![image](https://github.com/Mayonnaise-Slap/ITMO-ICT-CompNet-2024/blob/lab3/lab3/4c7090cd-3ce8-4d5c-b418-f5eab2cc3d14.jfif)
   ![image](https://github.com/Mayonnaise-Slap/ITMO-ICT-CompNet-2024/blob/lab3/lab3/c2030d7e-d5bd-4b8b-81b3-62f42e8c1e9b.jfif)

## Часть 2. Проверяем репликацию

## Часть 3. Делаем среднего роста высокую доступность

## Ответы на вопросы:
1. *Порты 8008 и 5432 вынесены в разные директивы, expose и ports. По сути, если записать 8008 в ports, то он тоже станет exposed. В
чем разница?*

   Порты, вынесенные в expose, доступны только для других контейнеров, а вынесенные в ports доступны извне.

3. *При обычном перезапуске композ-проекта, будет ли сбилден заново образ? А если предварительно отредактировать файлы
postgresX.yml? А если содержимое самого Dockerfile? Почему?*

   Нет, во всех случаях образ не будет сбилден заново, потому что образы в docker кэшируются для оптимизации.
