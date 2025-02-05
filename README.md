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

![image](https://github.com/user-attachments/assets/6d2519d3-fd48-406a-a24c-d1d5f00049db)

Копируем дефолты:
```
cp /etc/bind/zone/{localhost,au.db}
cp /etc/bind/zone/127.in-addr.arpa /etc/bind/zone/192.168.11.in-addr.arpa.db
cp /etc/bind/zone/127.in-addr.arpa /etc/bind/zone/192.168.33.in-addr.arpa.db
```
Назначаем права:
```
chown root:named /etc/bind/zone/au.db
chown root:named /etc/bind/zone/192.168.11.in-addr.arpa.db
chown root:named /etc/bind/zone/192.168.33.in-addr.arpa.db
```
Настраиваем зону прямого просмотра /etc/bind/zone/au.db:

![image](https://github.com/user-attachments/assets/31341ea1-0c81-4e8d-82d9-3536aee8518e)

Настраиваем зону обратного просмотра /etc/bind/zone/192.168.11.in-addr.arpa.db:

![image](https://github.com/user-attachments/assets/b29f9656-58d6-43d6-aeba-63565b22b234)

Настраиваем зону обратного просмотра /etc/bind/zone/192.168.33.in-addr.arpa.db:

![image](https://github.com/user-attachments/assets/51946b83-8d91-40aa-96cf-27adb060f8ec)


</details>
