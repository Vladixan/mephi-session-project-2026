Раздел 1. Установка дистрибутива
1.2. Настройка имени хоста

hostnamectl set-hostname mephi-2026.domain.local // для установки имени хоста mephi-2026.domain.local

В /etc/hosts добавлена запись 127.0.1.1   mephi-2026.domain.local mephi-2026

exec bash // принять изменения

hostname -f // для проверки

1. 3. Проверка сетевой связности

ping -c 4 8.8.8.8 // проверка связи с внешним миром

ping -c 4 8.8.8.8 > ~/ping.out // сохранить вывод команды в файл

Раздел 2. Управление программным обеспечением
2.1. Обновление дистрибутива

dnf update -y // для обновления пакетов дистрибутива

2.2. Установка пакетов из репозиториев

dnf install -y nginx и dnf install -y libcap-ng-utils // установка пакетов nginx и libcap-ng-utils

2.3. Установка локального RPM-пакета

dnf download --resolve tcpdump --destdir=/tmp // скачать пакет tcpdump и его зависимости в /tmp

rpm -ivh /tmp/*.rpm // выполнить установку всех скачанных пакетов

dnf history > dnf.out // сохранить историю транзакций менеджера пакетов в файл

Раздел 3. Управление файловыми системами

lsblk // проверка подключения диска к системе (подключен как sdb)

3.1. Создание файловой системы

parted /dev/sdb // создать раздел на диске

partprobe /dev/sdb // обновить информацию о разделах для подключенного диска

lsblk // проверка появления sdb1

mkfs.ext4 -L "MEPHI_WEB" /dev/sdb1 // форматирование раздела в ext4 с меткой МЕРНІ_WEB

3.2. Монтирование файловой системы

mkdir -p /mephi-web // создание точки монтирования

В /etc/fstab добавлена запись LABEL=МЕРНІ_WEB /mephi-web ext4 defaults 0 2

mount /mephi-web // ручного монтирование раздела

Раздел 4. Управление сервисами
4.1. Управление веб-сервером

systemctl start nginx // запуск nginx

systemctl enable nginx // активация автозагрузки

systemctl status nginx // проверка статуса

4.2. Журналирование

journalctl -u nginx -b > ~/journalctl.out // найдены все сообщения журнала, относящиеся к сервису nginx, для текущей загрузки системы, вывод команды направлен в файл

Раздел 5. Управление доступом
5.1. Дискреционное управление доступом (DAC)

mkdir -p /data/mephi-2026 // создание директории

groupadd -g 4444 curators и groupadd -g 4445 mephi-dev // создание групп для разработчиков и кураторов

useradd -u 5501 -G mephi-dev user1, useradd -u 5502 -G mephi-dev user2, useradd -u 5503 -G mephi-dev user3, useradd -u 5504 -G curators curator1, useradd -u 5505 -G curators curator2 // добавление новых пользователей

passwd user1, passwd user2, passwd user3, passwd curator1, passwd curator2 // задать пароли новым пользователям

chown root:mephi-dev /data/mephi-2026 // установить владельцем директории /data/mephi-2026 пользователя root и группу mephi-dev

chmod 2770 /data/mephi-2026 // установить базовые права: rwx для владельца и группы, 0 для остальных

setfacl -m g:curators:r-X /data/mephi-2026 и setfacl -d -m g:curators:r-X /data/mephi-2026 // настройка ACL для директории (у кураторов чтение и исполнение для директории)

5.2. Привилегии (уменьшение количества set-UID-программ)

setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump // выдать необходимые capabilities (cap_net_raw и cap_net_admin)

getcap /usr/sbin/tcpdump > ~/getcap.out // проверка, что capabilities установлены, вывод направлен в файл

5.3. Мандатное управление доступом (MAC)

getenforce > ~/getenforce.out // проверить, что SELinux работает в режиме "Enforcing". Вывод направлен в файл (команда выполнена в конце работы)

vi /mephi-web/index.html // создать /mephi-web/index.html и добавить запись "Hello from Student: M265810"

В /etc/nginx/conf.d/mephi.conf добавлена запись:

server {
    listen 80;
    server_name _;
    root /mephi-web;
    index index.html;
}

В /etc/selinux/targeted/contexts/files/file_contexts.local добавлена запись /mephi-web(/.*)? system_u:object_r:httpd_sys_content_t:s0

restorecon -Rv /mephi-web // применить контекст рекурсивно

systemctl restart nginx // перезапустить nginx

Раздел 6. Аутентификация
6.1. Ограничение входа для пользователей

В /etc/security/access.conf добавлена строка -:curators:LOCAL для запрета локального входа для пользователей группы curators

При попытке залогиниться за curator1 или curator2 выводится "Доступ отклонён"

6.2. Управление паролями

Отредактирован /etc/security/pwquality.conf // строка minlen = 8 заменена на minlen = 12, строка раскомментирована (минимальная длина пароля в 12 символов)

chage -M 90 user1, chage -M 90 user2, chage -M 90 user3, chage -M 90 curator1, chage -M 90 curator2 // для новых пользователей задана необходимость смены пароля через 90 дней

Раздел 7. Тестирование
7.1. Создание web-страницы

cat /mephi-web/index.html // файл index.html уже был создан, проверка записи в файле

curl http://localhost // проверка вывода
