# MANUAL №0: Запуск Postgres c помощью Docker. Docker-compose, import, backup (на примере демонстрационной базы данных)

![docker_and_postgres](pictures/docker_and_postgres.jpg)

#### Данную пошаговую инструкцию я пишу в первую очередь для себя, но возможно вам из этого что-то пригодится.

---
### 0. Установите Docker и скачайте демонстрационную базу данных:
- https://totaku.ru/ustanovka-docker-i-docker-compose-na-ubuntu-22-04/
- https://postgrespro.ru/docs/postgrespro/9.6/demodb-bookings-installation

### 1. docker-compose:
```
version: '3'
services:
  postgres:
    image: postgres
    restart: always
    environment:
      POSTGRES_PASSWORD: 1917
      POSTGRES_DB: demo
    volumes:
      - ./postgres-backup:/postgres-backup
    ports:
      - 5432:5432
```
 - `version` - дефолт
 - `image` - используем образ postgres
 - `restart` - дефолт
 - `POSTGRES_PASSWORD` - обязательное требование к заполнению пароля (иначе postgres работать не будет)
 - `POSTGRES_DB` - указываем имя сразу, чтобы потом можно было сделать импорт в одну строчку и не создавать базу руками (называем demo т.к. именно такое имя используется в демонстрационной базе)
 - `valumes` - монтируем свою директорию (в которой уже есть бэкап демонстрационной базы), чтобы потом не заниматься копированием. Сюда же мы потом будем складывать новые бэкапы и они будут сохроняться на нашей машине, что очень удобно
 - `ports` - дефолт (если у вас что-то работает на 5432 (ваша домашняя postgres обычно занимает этот порт), то поменяйте host_port (левый) на другой)

 ##### Пример действий (весь нейминг, кроме docker-compose.yml, условный):
  - Создаём директорию postgres-docker
  - Внутри создаём файл docker-compose.yml (содержимое описано выше)
  - Рядом с docker-compose.yml создаём директорию postgres-backup
  - Внутрь директории postgres-backup кладём файл backup.sql (демонстрационная база)
  - Находясь на уровне файла docker-compose.yml вводим команду `docker compose up -d`
  - Отлично! - Вы запустили контейнер, осталость сделать импорт базы

### 2. import:
 - Наш импорт будет всего в одну строчку:     
`docker exec -it postgres-docker-postgres-1 psql -U postgres -d demo -f /postgres-backup/backup.sql`    
ОПИСАНИЕ: Мы заходим в контейнер с помощью `docker exec -it postgres-docker-postgres-1` (postgres-docker-postgres-1 это дефолтное имя контейнера, если вы повторяли за мной. Вы всегда можете узнать имя вашего контейнера с помощью `docker ps`), после мы заходим в psql под пользователем postgres (дефолтный пользователь, если в docker-compose не указан иной) и импортируем наш бэкап в базу demo (которую мы заранее обозначали в docker-compose)

##### Надо немного подождать т.к. импорт занимает несколько минут (от 5 до 20 в среднем)

  - Готово! - Теперь вы можете зайти в базу (к примеру через dbeaver) и спокойно создавать в ней новые сущности, удалять их, делать селекты и прочее. 
  ```
  host - localhost
  port - 5432
  database - demo
  login - postgres (дефолтное значение, можно выставить конкретное в docker-compose.yml)
  password - 1917
  ```

### 3. backup:
##### Если по каким-то причинам вам надо сделать бэкап и при этом случайно не забыть его в контейнере навечно, то просто делайте его в директорию postgres-backup, ведь мы её монтировали!
 - ```docker exec -it postgres-docker-postgres-1 pg_dump -U postgres -d demo -f /postgres-backup/new_backup.sql```  
 ОПИСАНИЕ: Всё очень просто - используем уже знакомую конструкцию `docker exec...`, только теперь мы пользуемся инструментом для бэкапа и говорим ему какую базу нам надо сохранить (и куда её сохранить)
 - После этого действия вы сможете полностью удалить контейнер, но данные останутся у вас (в директории postgres-backup)


---
 #### P.S
 ##### Для удобтсва я создал директории (postgres-docker, postgres-backup) и docker-compose.yml прямо в этом репозитории. Вам останется только перенести ваш бэкап (хочу отметить, что демонстрационная база это по сути и есть бэкап) в директорию postgres-backup и действовать согласно инструкции. Удачи!