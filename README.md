###Домашнее задание

Для того, чтобы запретить всем пользоватлемя, кроме группы admin логин в выходные (суббота и воскресенье), я сделал следующее:

1. В файл ```/etc/pam.d/sshd``` внес строку ```account required pam_time.so``` для подключения модуля pam_time.
2. В файл ```/etc/security/time.conf``` внес строку ```*;*;!admin:Wk```, которая позволяет всем пользователям, кроме admin логин только в будние дни.



### Лабораторная работа

## Создаем пользователей

```bash
man useradd
useradd -m -s /bin/bash user1
useradd -m -s /bin/bash user2 
```
* Какие UID создались у пользователей?
```
[root@localhost ~]# id user1
uid=1000(user1) gid=1000(user1) группы=1000(user1)
[root@localhost ~]# id user2
uid=1001(user2) gid=1001(user2) группы=1001(user2)
[root@localhost ~]#
```
* Что означают опции -m и -s

Опция -m означает, что для пользователя будет создан домашний каталог
Опция -s устанавливает командную оболочку пользователя (в данном случае /bin/bash)

## Создаем группу и добавляем туда пользователей

```bash
man groupadd
man gpasswd
groupadd admins
gpasswd -a user1 admins
gpasswd -a user2 admins
id user1
id user2
```

* Результаты команды id добавить в README.md
```
[root@localhost ~]# id user1
uid=1000(user1) gid=1000(user1) группы=1000(user1),1002(admins)
[root@localhost ~]# id user2
uid=1001(user2) gid=1001(user2) группы=1001(user2),1002(admins)
```
* (*) Через usermod сделайте группу admins основной для  user1. Результат id приложить в README.md

Делаем основной группу admins:
```
[root@localhost ~]# usermod user1 -g 1002
```
Добавляем вторичную группу user1:
```
[root@localhost ~]# usermod user1 -G 1000
[root@localhost ~]# id user1
uid=1000(user1) gid=1002(admins) группы=1002(admins),1000(user1)
```

## Создать каталог от рута и дать права группе admins туда писать

```bash
mkdir /opt/upload
chmod 770 /opt/upload
chgrp admins /opt/upload
```

* что означают права 770 ?

Права 770 означает доступ на чтение, изменение и исполнение для владельца и группы и полный запрет доступа для остальных.

* создать по файлу от пользователей user1 и user2 в каталоге /opt/uploads
```
[root@localhost ~]# su user1
[user1@localhost root]$ touch /opt/upload/file1
[user1@localhost root]$ exit
exit
[root@localhost ~]# su user2
[user2@localhost root]$ touch /opt/upload/file2
[user2@localhost root]$ exit
exit
[root@localhost ~]# ls /opt/upload/
file1  file2
```
* проверьте с какой группой создались файлы от каждого пользователя. Как думаете - почему?
```
[root@localhost ~]# ll /opt/upload/
итого 0
-rw-r--r--. 1 user1 admins 0 янв 28 15:07 file1
-rw-rw-r--. 1 user2 user2  0 янв 28 15:07 file2
```
файл file1 создался с группой admins, потому что для пользователя user1 это основная группа
файл file2 создался с группой user2, потому что для пользователя user2 это основная группа

* (*) попробуйте сменить текущую группу пользователя  ```newgrp admins``` у пользователя user2 и создайте еще файл
```
[root@localhost ~]# man newgrp
[root@localhost ~]# su user2
[user2@localhost root]$ newgrp admins
[user2@localhost root]$ touch /opt/upload/file3
[user2@localhost root]$ exit
```
* приложить ```ls -l /opt/upload```  в  README.md
```
[user2@localhost root]$ ls -l /opt/upload
итого 0
-rw-r--r--. 1 user1 admins 0 янв 28 15:07 file1
-rw-rw-r--. 1 user2 user2  0 янв 28 15:07 file2
-rw-r--r--. 1 user2 admins 0 янв 28 15:11 file3
```

## Создать пользователя user3 и дать ему права писать в /opt/uploads

* Создайте пользователя user3
```
[root@localhost ~]# useradd -m -s /bin/bash user3
```
* Попробуйте записать из под него файл в /opt/uploads. Должны получить ошибку
```
[root@localhost ~]# su user3
[user3@localhost root]$ touch /opt/upload/file4
touch: невозможно выполнить touch для «/opt/upload/file4»: Отказано в доступе
```
* Считайте acl с каталога. Добавьте черерз  setfacl права на запись в каталог.
```bash
man getfacl
man setfacl
getfacl /opt/upload
setfacl -m u:user3:rwx /opt/upload
su - user3
touch /opt/upload/user3_file
ls -l /opt/upload/user3_file
```
```
[root@localhost ~]# man getfacl
[root@localhost ~]# man setfacl
[root@localhost ~]# getfacl /opt/upload/
getfacl: Removing leading '/' from absolute path names
# file: opt/upload/
# owner: root
# group: admins
user::rwx
group::rwx
other::---
[root@localhost ~]# setfacl -m u:user3:rwx /opt/upload/
[root@localhost ~]# su user3
[user3@localhost root]$ touch /opt/upload/user3_file
[user3@localhost root]$ ls -l /opt/upload/user3_file
-rw-rw-r--. 1 user3 user3 0 янв 28 15:14 /opt/upload/user3_file
```
* приложить ```ls -l /opt/upload```  в  README.md
```
[user3@localhost root]$ ls -l /opt/upload
итого 0
-rw-r--r--. 1 user1 admins 0 янв 28 15:07 file1
-rw-rw-r--. 1 user2 user2  0 янв 28 15:07 file2
-rw-r--r--. 1 user2 admins 0 янв 28 15:11 file3
-rw-rw-r--. 1 user3 user3  0 янв 28 15:14 user3_file
```
* приложить финишный acl  директории в README.md
```
[user3@localhost root]$ getfacl /opt/upload
getfacl: Removing leading '/' from absolute path names
# file: opt/upload
# owner: root
# group: admins
user::rwx
user:user3:rwx
group::rwx
mask::rwx
other::---
```

## Установить GUID флаг на директорию /opt/uploads

```bash
chmod g+s /opt/upload
su - user3
touch /opt/upload/user3_file2
ls -l /opt/upload/user3_file2
```
```
[root@localhost ~]# chmod g+s /opt/upload/
[root@localhost ~]# su - user3
[user3@localhost ~]$ touch /opt/upload/user3_file2
[user3@localhost ~]$ ls -l /opt/upload/user3_file2
-rw-rw-r--. 1 user3 admins 0 янв 28 15:17 /opt/upload/user3_file2
```
* Приложить ```ls -l /opt/upload```  в  README.md
```
[user3@localhost ~]$ ls -l /opt/upload
итого 0
-rw-r--r--. 1 user1 admins 0 янв 28 15:07 file1
-rw-rw-r--. 1 user2 user2  0 янв 28 15:07 file2
-rw-r--r--. 1 user2 admins 0 янв 28 15:11 file3
-rw-rw-r--. 1 user3 user3  0 янв 28 15:14 user3_file
-rw-rw-r--. 1 user3 admins 0 янв 28 15:17 user3_file2
```
* Объяснить почему изменилась группа при создании

Потому что командой chmod g+s мы установили на директорию GIUD флаг и для всех новых файлов группа владельца будет устанавливаться та, которая является группой владельца папки.

## Установить  SUID  флаг на выполняемый файл
* Установим suid бит на просмотрщик cat
```
chmod u+s /bin/cat
```
* В начале  попробуйте прочитать cat /etc/shadow  из под пользователя user3
```
[root@localhost ~]# su user3
[user3@localhost root]$ cat /etc/shadow
root:$6$vd2O/R0qYhW4xUp.$.5HTAcLMAqVD.GvjgmSADEd3ikJ6hw5ROGP8WmpmEy2XLN.tWJfpenKEHlCfL7lBzUVcLs4YFWShJyDIIGJ/g/::0:99999:7:::
user1:!!:18289:0:99999:7:::
user2:!!:18289:0:99999:7:::
user3:!!:18289:0:99999:7:::
```
* Установить suid /bin/cat и прочитайте снова из под user3 (Возможно, опечатка? SUID уже установлен, на данном шаге я его удалил)
```
[root@localhost ~]# chmod u-s /bin/cat
```
* В README.md добавьте оба результат
* Объясните почему
```
[root@localhost ~]# su user3
[user3@localhost root]$ cat /etc/shadow
cat: /etc/shadow: Отказано в доступе
```
В первом случае я установил SUID флаг на /bin/cat, соответственно данная утилита всегда запускалась от имени пользователя root и мне было возможно просмотреть содержимое /etc/shadow.
Когда же я данный SUID флаг удалил, то утилита начала запускаться от имени user3 (под которым я работал). У него прав на просмотр содержимого /etc/shadow нет.

##  Сменить владельца  /opt/uploads  на user3 и добавить sticky bit

```bash
chown user3 /opt/upload
chmod +t /opt/upload
su - user1
touch /opt/upload/user1_file_test
ls -l /opt/upload/user1_file_test
su - user3
rm -f  /opt/upload/user1_file_test
```

* Объясните почему user3 смог удалить файл, который ему не принадлежит

Мы установили sticky-бит t на директорию /opt/upload, таким образом удалять из этой директории файлы может только владелец данных файлов,
но мы установили ранее командой chown user3 /opt/upload владельцем этой директории пользователя user3, и он также может удалять файлы.

* Создайте теперь файл от user1 и удалите его пользователем user1

Создание и удаление файла прошло успешно, так как пользователь user1 является владельцем созданного файла.

## Записи в sudoers

* попробуйте из под user3 выполнить ```sudo ls -l /root```
```
[user3@localhost root]$ sudo ls -l /root

Мы полагаем, что ваш системный администратор изложил вам основы
безопасности. Как правило, всё сводится к трём следующим правилам:

    №1) Уважайте частную жизнь других.
    №2) Думайте, прежде что-то вводить.
    №3) С большой властью приходит большая ответственность.

[sudo] пароль для user3:
```
* для редактирования sudoers используйте  visudo
* почему у вас не получилось?

Потому что мы пытаемся выполнить команду ls из под суперпользователя, для этого нам необходимо ввести его пароль.

```
user3	ALL=NOPASSWD:/bin/ls
```
После создания данного файла, мы можем выполнять команду ls из под суперпользователя без ввода пароля.
* добавьте запись в /etc/sudoers.d/admins разрешающий группе admins любые команды с вводом пароля

для этого я добавил в этот файл запись
```
%admins ALL=(ALL) ALL
```


