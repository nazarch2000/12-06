# Домашнее задание к занятию «Репликация и масштабирование. Часть 1»

### Инструкция по выполнению домашнего задания

1. Сделайте fork [репозитория c шаблоном решения](https://github.com/netology-code/sys-pattern-homework) к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/gitlab-hw или https://github.com/имя-вашего-репозитория/8-03-hw).
2. Выполните клонирование этого репозитория к себе на ПК с помощью команды `git clone`.
3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
   - впишите вверху название занятия и ваши фамилию и имя;
   - в каждом задании добавьте решение в требуемом виде: текст/код/скриншоты/ссылка;
   - для корректного добавления скриншотов воспользуйтесь инструкцией [«Как вставить скриншот в шаблон с решением»](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md);
   - при оформлении используйте возможности языка разметки md. Коротко об этом можно посмотреть в [инструкции по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md).
4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`).
5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
6. Любые вопросы задавайте в чате учебной группы и/или в разделе «Вопросы по заданию» в личном кабинете.

Желаем успехов в выполнении домашнего задания.

---

### Задание 1

На лекции рассматривались режимы репликации master-slave, master-master, опишите их различия.
```
master-slave - запись в бд можно производить только на master сервере, slave же может только выполнять выборку данных из бд, он воспроизводит изменения из журнала ретрансляции применяя их к своей бд.

master-master - запись и выборку можно производить из любого сервера master. Повышает эффективность при обращении к данным.
```
---

### Задание 2

Выполните конфигурацию master-slave репликации, примером можно пользоваться из лекции.

Настройка мастера
```sql
CREATE USER 'replication'@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO 'replication'@'%';
FLUSH PRIVILEGES;
```
```
mysqldump sakila > sakila.sql
# Скинул этот дамп на слейв сервер
```
```sql
UNLOCK TABLES;
```
Настройка слейва
```sql
CREATE DATABASE `sakila`;
```
```
mysql sakila < sakila.sql
```
```sql
CHANGE MASTER TO MASTER_HOST='192.168.0.20',
MASTER_USER='replication', MASTER_PASSWORD='password', MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=157;
START SLAVE;
```


![image](https://user-images.githubusercontent.com/106932460/236699002-e47c0ace-4a4f-4240-b53b-3409c8a7093d.png)
![image](https://user-images.githubusercontent.com/106932460/236698991-6fc583cb-f5f4-4df9-a895-b7c135c750b2.png)
![image](https://user-images.githubusercontent.com/106932460/236699016-46278919-679a-494d-b6d6-249b982a2b49.png)
![image](https://user-images.githubusercontent.com/106932460/236699021-56fae366-e718-4eb8-b8a5-3cdd51753fec.png)

---

## Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

---

### Задание 3* 

Выполните конфигурацию master-master репликации. Произведите проверку.

На первом мастере выполнил
```sql
slave stop;
CHANGE MASTER TO MASTER_HOST = '192.168.0.10', MASTER_USER =
'replicator', MASTER_PASSWORD = 'password', MASTER_LOG_FILE =
'mysql-bin.000002', MASTER_LOG_POS = 408;
slave start;
```
На втором мастере выполнил
```sql
CREATE USER 'replication'@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO 'replication'@'%';
FLUSH PRIVILEGES;
slave stop;
CHANGE MASTER TO MASTER_HOST = '192.168.0.20', MASTER_USER =
'replicator', MASTER_PASSWORD = 'password', MASTER_LOG_FILE =
'mysql-bin.000002', MASTER_LOG_POS = 1044;
slave start;
```
*Приложите скриншоты конфигурации, выполнения работы: состояния и режимы работы серверов.*
![image](https://user-images.githubusercontent.com/106932460/236699851-8143fb87-c161-41a0-9fae-dda86a7f0953.png)
![image](https://user-images.githubusercontent.com/106932460/236699820-f6b695a4-9ae0-4aa0-8254-fcbea77c5740.png)
```
Все получилось все нормально, НОООООООООООООООООООООООООООООООООООООООООООООООООООО
Красным пометил, как это исправить? 
На одной мастер ноде я сделал INSERT с городом Vyborg, на другой с городом Severodvinsk, 
почему-то они не обменялись друг с другом этим, хотя тогда уже репликация была настроена, 
Это вручную исправлять или есть какой-то способ, который автоматически исправит, если вдруг такая ошибка не одна в базе?
Просто нужно чтобы на обеих нодах был и Vyborg и Severodvinsk.
```
![image](https://user-images.githubusercontent.com/106932460/236700896-f0edecca-2c21-4dcc-95da-ef6662166823.png)


