# Модуль Б. (Настройка технических и программных средств информационно-коммуникационных систем)

Время на выполнение модуля 5 часов.  

Рисунок 1 – Схема сети модуля Б
![image](https://github.com/user-attachments/assets/7df8d97e-b954-424d-930e-9664f50d9825)
Доступ к ISP вы не имеете!!
 
| Название устройства | ОС | FQDN |
|:-|:-|:-|
| R-DT | EcoRouter | r-dt.au.team |
| FW-DT | Ideco ngfw | fw-dt |
| ADMIN-DT | Альт рабочая станция 10 | admin-dt.au.team |
| SW-DT | Виртуальный коммутатор | - |
| SRV1-DT | Альт Сервер 10 | srv1-dt.au.team |
| SRV2-DT | Альт Сервер 10 | srv2-dt.au.team |
| SRV3-DT | Альт Сервер 10 | srv3-dt.au.team |
| CLI-DT | Альт рабочая станция 10 | cli-dt.au.team |
| R-HQ | EcoRouter | r-hq.au.team |
| SW1-HQ | Альт сервер 10 | sw1-hq.au.team |
| SW2-HQ | Альт сервер 10 | sw2-hq.au.team |
| SW3-HQ | Альт сервер 10 | sw3-hq.au.team |
| ADMIN-HQ | Альт рабочая станция 10 | admin-hq.au.team |
| SRV1-HQ | Альт Сервер 10 | srv1-hq.au.team |
| CLI-HQ | Альт рабочая станция 10 | cli-hq.au.team |
| CLI | Альт рабочая станция 10 | cli |

| Название устройства | IP-адрес |
|:-|:-|
| R-DT | 192.168.33.1, dhcp, 10.10.10.2 |
| R-HQ | 192.168.11.1, dhcp, 10.10.10.1 |
| SRV1-HQ | 192.168.11.254 |
| SRV1-DT | 192.168.33.254 |
| SRV2-DT | 192.168.33.253 |
| SRV3-DT | 192.168.33.252 |
| ADMIN-HQ | 192.168.11.2 |
| ADMIN-DT | 192.168.33.2 |
| CLI-HQ | dhcp |
| CLI-DT | dhcp |

# Часть 1
## Настройка адаптеров
  <details>
    <summary>НАЖМИ</summary>

Для того чтобы посмотреть какие адаптеры подключены к устройству прописываем:
```
ip link show
```
Далее необходимо создать директорию по пути
```
mkdir /etc/net/ifaces/
```
1) На адаптере для локальной сети файл "options" должен выглядить так:
   
![image](https://github.com/user-attachments/assets/39004dc1-d143-4a41-939c-0b4112b226e2)

2) Для выхода в интернет так:
   
![image](https://github.com/user-attachments/assets/9054222e-80d8-4b08-af23-5fe98b2b1188)


Для того чтобы при перезапуске не сбрасывался адреса устройства необходимо в папке /etc/systemd/system создать файл сервиса:
1) network-restart.timer:
```
[Unit]
Description=Restart Network Timer

[Timer]
OnStartupSec=10s
Unit=network-restart.service

[Install]
WantedBy=timers.target
```
2) network-restart.service:
```
[Unit]
Description=Restart Network Service

[Service]
Type=oneshot
ExecStart=/bin/systemctl restart network

[Install]
WantedBy=multi-user.target
```
Чтобы запустить службу прописываем:
```
systemctl enable network-restart.timer
```

  </details>
  
## Созданаие пользователя
  <details>
    <summary>НАЖМИ</summary>
    
Для того чтобы создать пользователя прописываем:
```
adduser sshuser
```
Задаем пароль:
```
passwd sshuser
```
Добавляем в группу sudo:
```
usermod -aG wheel sshuser
```
Для того чтобы при выполнении команды sudo не запрашивался пароль необходимо отредактировать файл /etc/sudoers. Вписываем:
```
sshuser ALL=(ALL) NOPASSWD: ALL
```

  </details>

## Настройка динамической трансляции адресов
  <details>
    <summary>НАЖМИ</summary>
    
Откройте файл /etc/sysctl.conf и добавьте строку:
```
net.ipv4.ip_forward=1
```
Отредактируйте строчку в файле /etc/net/sysctl.conf:
```
net.ipv4.ip_forward=1
```
Пропишите команду для настройки динамической трансляции адресов:
```
iptables -t nat -A POSTROUTING -o ens37 -j MASQUERADE
sysctl -p
```
Далее необходимо сохранить настройки:
```
mkdir /etc/iptables
iptables-save>/etc/iptables/rules.v4
```
Для того чтобы после перезагрузки роутера не сбрасывались настройки необходимо прописать systemd-юнит iptables-restore.service:
```
[Unit]
Description=Restore iptables rules
Before=network.target

[Service]
Type=oneshot
ExecStart=/sbin/iptables-restore /etc/iptables/rules.v4

[Install]
WantedBy=multi-user.target
```
Далее необходимо включить юнит:
```
systemctl enable iptables-restore.service
systemctl start iptables-restore.service
```

</details>

## Настройка протокола динамической конфигурации хостов
<details>
    <summary>НАЖМИ</summary>
  
  Для начала укажем сетевой интерфейс, через который будет работать DHCP-сервер:
```
vim /etc/sysconfig/dhcpd
```
![image](https://github.com/user-attachments/assets/db8ed7d5-0088-4872-929e-0a6e904ca657)

В папке /etc/dhcp/ необходимо создать файл dhcpd.conf:
```
cp dhcpd.conf.example dhcpd.conf
```
Отредактируйте файл dhcpd.conf следующим образом:

![image](https://github.com/user-attachments/assets/c96ecbb5-faae-4fb4-a9cb-89e5ded75565)

Перезагружаем службу:
```
systemctl restart dhcpd
```

Чтобы служба включалась после перезапуска устройства можно добавить ее в systemd юнит следующим образом(редактируется служба network-restart.service):

![image](https://github.com/user-attachments/assets/e929cbfd-2d7e-49c1-9a27-db636dfb165c)


</details>

## Настройка GRE туннеля
<details>
    <summary>НАЖМИ</summary>

 Создаем директорию для туннеля:
 ```
mkdir /etc/net/ifaces/tun1
```
Редактируем файл options следующим образом:

![image](https://github.com/user-attachments/assets/e6e0a7c4-6e93-4d32-a328-1d0949e78c63)

Здесь TUNLOCAL - IP адресс адаптера с NAT на настраиваемом роутере, TUNREMOTE на другом роутере.

Задаем IP адрес:
```
echo 10.10.10.1/30 > /etc/net/ifaces/tun1/ipv4address
```
Перезагружаем сеть:
```
systemctl restart network
```

</details>

## Настройка динамической маршутизации OSPF
<details>
    <summary>НАЖМИ</summary>

Устаналиваем пакет quagga:
```
apt-get install quagga
```
Редактируем файл /etc/quagga/ospfd.conf следующим образом:

![image](https://github.com/user-attachments/assets/16e7fa71-9dd3-4db0-b3b8-d2a8c66ebe67)

Редактируем файл /etc/quagga/zebra.conf следующим образом:

![image](https://github.com/user-attachments/assets/2ac82a2c-ee19-49a9-9f70-290c658b7164)

Включаем ospfd и zebra:
```
systemctl start ospfd zebra
```
Проверяем туннель:
```
vtysh -c "show ip ospf neighbor"
```
Для того чтобы сохранить настройки и добавить ospfd и zebra в автозагрузку необходимо создать unit-файлы:

  Создайте файл /etc/systemd/system/ospfd.service:
  ```
[Unit]
Description=Open Shortest Path First daemon

[Service]
ExecStart=/usr/sbin/ospfd
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
И файл /etc/systemd/system/zebra.service:
```
[Unit]
Description=Zebra daemon

[Service]
ExecStart=/usr/sbin/zebra
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
Необходимо запустить службы:
```
systemctl daemon-reload
systemctl enable ospfd zebra
systemctl start ospfd zebra
```

</details>

## Настройка DNS
<details>
    <summary>НАЖМИ</summary>

### SRV1-HQ
Для начала необходимо отредактировать файл /etc/bind/options.conf:
```
listen-on { any; };
allow-query { any; };
allow-transfer { 192.168.33.254; };
```
Включаем resolv:
```
nano /etc/net/ifaces/ens33/resolv.conf
```
```
systemctl restart network
```
Автозагрузка bind:
```
systemctl enable --now bind
```
Создаем прямую и обратную зону в /etc/bind/local.conf:

![image](https://github.com/user-attachments/assets/e2363b2a-676e-4b05-aabd-22fd82610b84)

Копируем дефолты:
```
cp /etc/bind/zone/{localhost,au.db}
cp /etc/bind/zone/127.in-addr.arpa /etc/bind/zone/11.168.192.in-addr.arpa.db
cp /etc/bind/zone/127.in-addr.arpa /etc/bind/zone/33.168.192.in-addr.arpa.db
```
Назначаем права:
```
chown root:named /etc/bind/zone/au.db
chown root:named /etc/bind/zone/11.168.192.in-addr.arpa.db
chown root:named /etc/bind/zone/33.168.192.in-addr.arpa.db
```
Настраиваем зону прямого просмотра /etc/bind/zone/au.db:

![image](https://github.com/user-attachments/assets/a19e12c8-f4b8-4fbe-b3b7-03db2661374a)

Настраиваем зону обратного просмотра /etc/bind/zone/11.168.192.in-addr.arpa.db:

![image](https://github.com/user-attachments/assets/dabffe3f-e53e-482b-99d7-965558bb529d)

Настраиваем зону обратного просмотра /etc/bind/zone/33.168.192.in-addr.arpa.db:

![image](https://github.com/user-attachments/assets/1fb3f96e-11ab-4d50-b964-1cd835737882)

Проверяем зоны:
```
named-checkconf -z
```

![image](https://github.com/user-attachments/assets/accf29f3-8684-4295-a4a7-c4c25ec49010)

### SRV1-DT
Конфиг
```
vim /etc/bind/options.conf
```

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/e90d49ce-6735-4fdb-b44a-0b1c62b8305a)

Добавляем зоны

![image](https://github.com/user-attachments/assets/d56f5eab-0717-4cdb-b4f3-67fe0e46345c)

Резолв `/etc/net/ifaces/ens33/resolv.conf`:
```
search au.team
nameserver 192.168.11.254
nameserver 192.168.33.254
```
Перезапуск адаптера:
```
systemctl restart network
```
Автозагрузка:
```
systemctl enable --now bind
```
SLAVE:
```
control bind-slave enabled
```

</details>

## Настройка синхронизации времени между сетевыми устройствами по протоколу NTP. 
  <details>
    <summary>НАЖМИ</summary>
    
Установка chrony:
```
apt-get install chrony
```
Редактируем конфиг /etc/chrony.conf:
```
server ntp2.vniiftri.ru iburst
allow 192.168.11.0/24
allow 192.168.33.0/24
local stratum 5
```
Запускаем chrony:
```
systemctl enable chronyd
systemctl start chronyd
```
Если успешно то вывод chronyc sources будет следующим:

![image](https://github.com/user-attachments/assets/ef643891-3879-4a21-a0a1-6a06d7339222)

На клиентах в конфиге редактируем:
```
server 192.168.11.254 iburst
```
chronyc source:

![image](https://github.com/user-attachments/assets/d19babd1-5d37-4e95-bae0-ecffbae41aee)

Альтернативно можно настроить через NTP:
```
apt-get install ntp
vim /etc/ntp.conf
```
Указываем:
```
server 127.127.1.0
server ntp2.vniiftri.ru iburst
fudge 127.127.1.0 stratum 5
restrict 192.168.11.0 mask 255.255.255.0 nomodify notrap
restrict 192.168.33.0 mask 255.255.255.0 nomodify notrap
```
Запускаем и проверяем:
```
systemctl enable ntp
systemctl start ntp
ntpq -p
```

 </details>

## Настройка SAMBA 
  <details>
    <summary>НАЖМИ</summary>
    
Устанавливаем SAMBA:
```
apt-get update
apt-get install samba samba-dc
```
Создаем домен:
```
rm /etc/samba/smb.conf
samba-tool domain provision --use-rfc2307 --interactive
```
Настройка BIND9_DLZ:
```
vim /etc/bind/named.conf.local
```
Вставляем:
```
dlz "AD DNS Zone" {
  database "dlopen /usr/lib/x86_64-linux-gnu/samba/bind9/dlz_bind9.so";
};
```
Создаем группы:
```
samba-tool group add group1
samba-tool group add group2
samba-tool group add group3
```
Создаем пользователей:
```
for i in {1..30}; do
    samba-tool user create user$i P@ssw0rd
    if [ $i -le 10 ]; then
        samba-tool group addmembers group1 user$i
    elif [ $i -le 20 ]; then
        samba-tool group addmembers group2 user$i
    else
        samba-tool group addmembers group3 user$i
    fi
done
```
Создаем подразделения:
```
samba-tool ou create OU=CLI
samba-tool ou create OU=ADMIN
```
Добавляем зоны обратного просмотра:
```
samba-tool dns zonecreate 127.0.0.1 11.168.192.in-addr.arpa -U administrator
samba-tool dns zonecreate 127.0.0.1 33.168.192.in-addr.arpa -U administrator
samba-tool dns zonelist 127.0.0.1 -U administrator
```
Проверяем Kerberos (Если не будет подключаться):
```
vim /etc/krb5.conf
```
Редактирум:
```
[libdefaults]
 default_realm = AU.TEAM
 dns_lookup_kdc = true
[realms]
 AU.TEAM = {
  kdc = srv1-hq.au.team
  admin_server = srv1-hq.au.team
 }
[domain_realm]
 .au.team = AU.TEAM
 au.team = AU.TEAM
```
Получаем билет Kerberos:
```
kinit administrator@AU.TEAM
klist
```
Добавляем устройства в домен:
```
apt-get update
apt-get install samba samba-dc
rm /etc/samba/smb.conf
sudo samba-tool domain join au.team DC -U administrator
```
Помещаем компьютеры в подразделения:
```
samba-tool computer move ADMIN-DT OU=ADMIN
samba-tool computer move CLI-DT OU=CLI
samba-tool computer move ADMIN-HQ OU=ADMIN
samba-tool computer move CLI-HQ OU=CLI
```
Создаем общую папку:
```
sudo mkdir -p /opt/data/SAMBA
sudo chmod -R 777 /opt/data/SAMBA
```
Редактируем /etc/samba/smb.conf:
```
[SAMBA]
path = /opt/data/SAMBA
read only = no
browsable = yes
valid users = @group1, @group2, @group3
```

 </details>

# Часть 2

## Управление доменом в ADMC
<details>
    <summary>НАЖМИ</summary>
    Делать все в конфигурации компьютера.

  Картинку можно разместить в папке клиента заранее и настроить политику на нее, потому что сетевые папки со старта компьютер не видит и фон не прогрузится.
    
</details>

## Реализация бекапа общей папки на сервере SRV1-HQ с использованием systemctl
<details>
    <summary>НАЖМИ</summary>
Создаем скрипт:
  
  ```
  vim /usr/local/bin/backup.sh
  ```
  
Cкрипт:
  ```
#!/bin/bash

# Указываем путь к исходной папке и целевому каталогу для бэкапов
SOURCE_DIR="/opt/data/SAMBA"
BACKUP_DIR="/var/backups"
DATE=$(date +"%Y-%m-%d_%H%M")

# Проверяем наличие каталога для бэкапов
if [ ! -d "$BACKUP_DIR" ]; then
  mkdir -p "$BACKUP_DIR"
fi

# Выполняем архивацию
tar -czf "${BACKUP_DIR}/backup_${DATE}.tar.gz" "$SOURCE_DIR"
```

Делаем скрипт исполняемым:
```
chmod +x /usr/local/bin/backup.sh
```

Создаем файл сервис:
```
vim /etc/systemd/system/backup.service
```

Файл:
```
[Unit]
Description=Backup service for SAMBA shared folder
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/backup.sh

[Install]
WantedBy=multi-user.target
```
Создаем таймер:
```
vim /etc/systemd/system/backup.timer
```

Таймер:
```
[Unit]
Description=Run backup daily at 8 PM

[Timer]
OnCalendar=20:00
Persistent=true
Unit=backup.service

[Install]
WantedBy=timers.target
```

Запускаем службы:
```
systemctl daemon-reload
systemctl enable backup.service backup.timer
systemctl start backup.service backup.timer
```

</details>

## Развертывание приложений в Docker на SRV2-DT
<details>
    <summary>НАЖМИ</summary>

Устанавливаем если не установлен:
```
apt-get update
apt-get install docker-ce
```

Запускаем локальный docker registry:
```
docker run -d -p 5000:5000 --restart=always --name registry registry:2
docker ps
```

Создаем директорию для WEB:
```
mkdir ~/web-app
cd ~/web-app
```

Создаем index.html
```
vim index.html
```

Файл:
```
<html>
    <body>
        <center><h1><b>WEB</b></h1></center>
    </body>
</html>
```

Создаем dockerfile:
```
vim Dockerfile
```

Файл:
```
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
```

Собираем и загружаем образ в локальный Registry:
```
docker build -t localhost:5000/web:1.0 .
docker push localhost:5000/web:1.0
curl http://localhost:5000/v2/_catalog
```

Запускаем:
```
docker run -d --name web -p 80:80 --restart=always localhost:5000/web:1.0
docker ps
```

### ЕСЛИ НЕ ЗАПУСКАЕТСЯ:

Проблема может быть в том что какой то другой процесс использует 80 порт, поэтому необходимо узнать это:
```
netstat -tuln | grep :80
lsof -i :80
```

Останавливаем процесс:
```
systemctl stop название процесса
systemctl disable название процесса
```

Если не помогло можно попробовать перенаправить контейнер с 8080 порта на 80:
```
docker run -d --name web -p 8080:80 --restart=always localhost:5000/web:1.0
```

### Если используем порт 8080 то в браузере вбиваем http://192.168.33.253:8080

### Если 80, то http://192.168.33.253

В случае успеха страница будет выглядеть так:

![image](https://github.com/user-attachments/assets/21efa5a5-fcfd-4c22-94b1-a0f0f33a7054)


</details>

## Настройка системы централизованного мониторинга
<details>
    <summary>НАЖМИ</summary>
  
### Не работает 

Устанавливаем пакеты:
```
apt-get update
apt-get install postgresql17 postgresql17-contrib apache2 zabbix-server-pgsql postgresql17-server postgresql17-server-devel zabbix-agent
```

Инициализируем БД:
```
sudo -u postgres initdb -D /var/lib/pgsql/data
```

Запускаем Postgresql:
```
systemctl start postgresql
systemctl enable postgresql
```

Подключаемся к Postgresql:
```
sudo -u postgres psql
```

Создаем БД и юзера:
```
CREATE DATABASE zabbix;
CREATE USER zabbix WITH PASSWORD 'zabbixpwd';
GRANT ALL PRIVILEGES ON DATABASE zabbix TO zabbix;
\q
```

Импортируем схему Zabbix в бд:
```
rpm -q zabbix-server-pgsql
sudo wget https://cdn.zabbix.com/zabbix/sources/stable/7.0/zabbix-7.0.9.tar.gz
sudo tar -xvzf zabbix-7.0.9.tar.gz
sudo cd zabbix-7.0.9/database/postgresql/
cat schema.sql | sudo -u zabbix psql zabbix
cat images.sql | sudo -u zabbix psql zabbix
cat data.sql | sudo -u zabbix psql zabbix
zcat /usr/share/doc/zabbix-server-pgsql/create.sql.gz | sudo -u zabbix psql zabbix
```

Проверяем БД и юзера:
```
sudo -u postgres psql -c "\l"
sudo -u postgres psql -d zabbix -U zabbix -W
```

Редактируем конфиг Zabbix Server - /etc/zabbix/zabbix_server.conf:
```
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=zabbixpwd
```

Создаем конфиг Zabbix:
```
cd /etc/httpd2/conf/addon.d
vim A.zabbix.conf
```

конфиг:
```
<VirtualHost *:80>
    DocumentRoot /usr/share/zabbix
    <Directory /usr/share/zabbix>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>
</VirtualHost>
```

Настраиваем часовой пояс:
```
cd /etc/php/8.2/apache2-mod_php
sudo pluma php.ini
```

Редактируем:
```
date.timezone = Europe/Moscow
```

Запускаем Zabbix:
```
sudo systemctl start zabbix_pgsql
sudo systemctl enable zabbix_pgsql
```

</details>


## Настройка веб-сервера nginx как обратного прокси-сервера на SRV1-DT
<details>
    <summary>НАЖМИ</summary>

Устанавливаем nginx:
```
sudo apt update
sudo apt install nginx
```

Создаем конфиг для www.au.team:
```
sudo vim /etc/nginx/sites-available/www.au.team
```

Конфиг:
```
server {
    listen 80;
    server_name www.au.team;

    location / {
        proxy_pass http://192.168.33.253:8080;  
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Включаем:
```
sudo ln -s /etc/nginx/sites-available.d/www.au.team /etc/nginx/sites-enabled.d/
```

Проверяем конфиг nginx:
```
sudo nginx -t
```

Перезапускаем:
```
sudo systemctl restart nginx
```

На dns сервере необходимо создать запись:
```
sudo samba-tool dns add 127.0.0.1 au.team www A 192.168.33.253 -U administrator
```

### Работает через 8080 порт 

![image](https://github.com/user-attachments/assets/e8e65473-022f-44e6-9670-421ae9e3a969)


</details>
