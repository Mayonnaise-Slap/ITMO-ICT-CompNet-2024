# ЛР 1. Loki + Zabbix + Grafana

**Задача:**
Подключить к тестовому сервису Nextcloud мониторинг + логирование. Осуществить визуализацию через Grafana.

## Часть 1. Логирование
1. Создали файл docker-compose.yml, который содержит в себе тестовый сервис Nextcloud, Loki, Promtail, Grafana, Zabbix и Postgres для него.
2. Создали файл promtail_config.yml
3. Запустили compose файл
   
   ![image](https://github.com/user-attachments/assets/3ccde7ed-84a3-4703-8e1d-7affe959a280)
   ![image](https://github.com/user-attachments/assets/dbefc283-f692-4936-9ab5-6f8dbf707fb8)

5. Иницианизировали Nextcloud, проверили, куда идут логи.
  ![image](https://github.com/user-attachments/assets/a50f1c60-a891-459c-898a-42f1bfda5c08)
6. Log файл, который был нам нужен, "подцеплен".
   ![image](https://github.com/user-attachments/assets/875a5a87-02ca-4c5e-a8ef-91f4b9f1e1db)

## Часть 2. Мониторинг
1. Настроили zabbix.
2. Сделали импорт темплейта для мониторинга Nextcloud.
   ![image](https://github.com/user-attachments/assets/b308d8ae-bf55-4c16-965e-81764b3ea9fa)
3. ![image](https://github.com/user-attachments/assets/4613c960-27eb-45e4-acf4-967b44b258b8)
4. Настроили хост.
   ![image_2024-10-16_13-12-37](https://github.com/user-attachments/assets/0ff46623-698e-4b11-a853-e160afe3e480)
5. Проверяем настройку
   ![image](https://github.com/user-attachments/assets/b2348ff3-9053-4aeb-bf86-ae7b7e142deb)
   ![image](https://github.com/user-attachments/assets/8b6971d4-3f87-4735-8bf8-af2d15ea81ef)
   ![image_2024-10-16_13-19-50](https://github.com/user-attachments/assets/4d0ac9f8-cf1d-40b9-b9b1-2868d88890e2)

## Часть 3. Визуализация 
1. ![image](https://github.com/user-attachments/assets/20c1aa92-88ba-496a-a39b-f7fe03644fa8)
2. ![image](https://github.com/user-attachments/assets/21522d5a-c3c3-4b4a-97b8-ca529bb438b8)
3. ![image](https://github.com/user-attachments/assets/e5cfe738-18a9-45d6-8bf4-6373eb87cfaa)
4. ![image](https://github.com/user-attachments/assets/b29a5cfe-578f-4e6b-9154-25b0fbff6a51)
5. ![image](https://github.com/user-attachments/assets/ea06ec45-5c8e-4f76-80e1-04ffe94f6531)

## Ответы на вопросы:
1. *Чем SLO отличается от SLA?*
   SLA - это соглашение об измеримых показателях, а SLO  - это соглашение о конкретном показателе в рамках SLA (цель уровня обслуживания).
2. *Чем отличается инкрементальный бэкап от дифференциального?*
    Инкрементное резервное копирование сохраняет все изменения, сделанные с момента последнего резервного копирования а дифференциальное резервное копирование сохраняет изменения, сделанные с момента последнего ПОЛНОГО резервного копирования
3. *В чем разница между мониторингом и observability?* Мониторинг — это процесс отслеживания поведения системы через сбор определенных метрик и событий. Observability (наблюдаемость) — это процесс регулярной, всесторонней диагностики внутреннего состояния ИТ-системы, на основе таких данных как метрики, события, логи, трассировки транзакций. Основная цель мониторинга — оперативно ответить на вопрос "что произошло?", о цель observability - ответить на вопросы "почему это произошло?" и "как именно это произошло?".






   
