---
title: Установка PostgreSQL на Ubuntu
date: '2023-05-08'
lastmod: '2023-05-08'
tags:
- PostgreSQL
- Ubuntu
- инструкция
categories:
- Linux
translationKey: postgresql-on-ubuntu
aliases:
- /postgresq_on_ubuntu/
---

В статье описана установка PostgreSQL 13 на Ubuntu 20.04, создание базы данных и настройка удаленного к ней подключения.
Обратите внимание, если вы хотите установить актуальную версию, воспользуйтесь командой `apt install postgresql`

Перед установкой обновите систему командой
```shell
sudo apt update
sudo apt -y upgrade
```
После обновления системы выполните перезагрузку, чтобы запустить новое ядро. Можно использовать команды `reboot` или `init 6`.

## Добавление репозитория PostgreSQL 13 в Ubuntu
Добавьте репозиторий PostgreSQL
```shell
sudo apt -y install vim shell-completion wget net-tools
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
echo "deb http://apt.postgresql.org/pub/repos/apt/  '`lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
```

# Установка PostgreSQL 13
Установите PostgreSQL 13:
```shell
sudo apt update
sudo apt install postgresql-13 postgresql-client-13
```
Если после установки сервер баз данных не запустился, то выполните запуск командой:
```shell
sudo pg_ctlcluster 13 main start
```
## Проверка соединения с PostgreSQL
Во время установки автоматически создается пользователь postgres. Этот пользователь имеет полный доступ суперадминистратора ко всему экземпляру PostgreSQL. Системный пользователь, вошедший в систему, должен иметь права sudo перед переключением на эту учетную запись.
```shell
sudo su - postgres
```
Чтобы изменить пароль пользователя на надёжный, введите:
```shell
psql -c "alter user postgres with password 'YourSuperPass';"
```
Если PostgreSQL уже запущен, то укажите запрос:
```sql
alter user postgres with password 'YourSuperPass';
```
Запустите командную строку PostgreSQL:
```shell
psql
```
В результате вы должны увидеть что-то такое:
```
psql (13.3 (Ubuntu 13.3-1.pgdg20.04+1))00
Введите `help`, чтобы получить справку.
```
Если хотите посмотреть детали подключения, есть такая команда:
```shell
\conninfo
```
И в результате получите такой ответ:
```
Вы подключены к базе данных "postgres" как пользователь "postgres" через сокет в "/var/run/postgresql", порт "5432".
```
Теперь переходим к созданию базы данных PostgreSQL.
## Создание базы данных
Создайте базу данных:
```sql
CREATE DATABASE MYDB;
```
Создайте пользователя базы данных с паролем. Вместо `myuser` и пароля `123` подставьте свои значения.
```sql
CREATE USER myuser WITH ENCRYPTED PASSWORD '123';
```
в ответ получите 
```
CREATE ROLE
```
Теперь назначьте права созданному пользователю в рамках указанной базы данных
```sql
GRANT ALL PRIVILEGES ON DATABASE MYDB to myuser;   
```
По необходимости, предоставьте созданному пользователю права создавать роли:
```sql
alter role myuser with createrole;
```

Подготовительный этап завершён. Давайте теперь попробуем вывести список баз данных:
```sql
\l
```
В ответ получим:
```

	postgres=# \l 
 
   Имя    | Владелец | Кодировка | LC_COLLATE  |  LC_CTYPE   |     Права доступа      
-----------+----------+-----------+-------------+-------------+----------------------- 
postgres  | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 |  
MYDB      | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 |  
template0 | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 | =c/postgres          + 
          |          |           |             |             | postgres=CTc/postgres 
template1 | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 | =c/postgres          + 
          |          |           |             |             | postgres=CTc/postgres 
(4 строки)
```
Выход из psql 
```sql
\q
```
Выход из суперпользователя postgres
```sql
exit
```
# Настройка удалённого подключения
По желанию, можно настроить удаленное подключение к базе данных. Если у вас приложение работает на том же сервере, этот пункт можете пропустить. 
PostgreSQL 13 на Ubuntu по умолчанию принимает соединения только от localhost. В производственных средах обычно используют сервер базы данных и подключенные к нему удаленные клиенты.
## Разрешение удаленного подключения
Чтобы разрешить удаленные подключения, отредактируйте файл конфигурации PostgreSQL:
```shell
sudo nano /etc/postgresql/13/main/postgresql.conf
```
Откроется конфигурационный файл. В нем раскоментируйте или добавьте строчку на выбор:
- прослушивать все интерфейсы:
```shell
listen_addresses = '*' # Listen on all interfaces
```
- прослушивать только на заданных адресах:
```shell
listen_addresses = '192.168.1.101' # Listen on specified private IP address
```
В нашем случае выбран адрес `192.168.1.101`, у вас будет другой.

Для текстового редактора nano:
- Чтобы сохранить изменения в файле используйте сочетание клавиш `ctrl + o`
- Для выхода из режима редактирования файла используйте `ctrl + x`.

## Разрешение приема удаленных подключений
Настройте PostgreSQL на прием удаленных подключений от разрешенных хостов.
Перейдите в конфигурационный файл:
```shell
sudo nano /etc/postgresql/13/main/pg_hba.conf
```
Раскоментируйте или добавьте новую строчку на выбор:
- прием удаленных подключений от всех
```shell
host all all 0.0.0.0/0 md5   # Досnуп отовсюду
```
- прием удаленных подключений от разрешенных подсетей
```shell
host all all 10.10.10.0/24 md5    # Доступ из выбранной подсети
```
Не забудьте указать свои подсети, вместо моих.

После изменения перезапустите службу postgresql.
```shell
sudo systemctl restart postgresql
```
Проверьте адрес прослушивания.
```shell
netstat -tunelp | grep 5432
```
В результате получите такой ответ:
```
tcp 0  0 0.0.0.0:5432   0.0.0.0:*     LISTEN   123    4698674  -  tcp6  0   0 :::5432    :::*    LISTEN   123     4698675
```
PostgreSQL настроен и готовк к подключению удаленных клиентов.

Если есть вопросы, можете задать их в комментариях в [моем канале](https://t.me/Press_Any).