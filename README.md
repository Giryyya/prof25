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
## 1. Базовая настройка
### R-HQ R-DT
# Настройка адаптеров
  <details>
    <summary>КЛИК</summary>

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
  
# Созданаие пользователя
  <details>
    <summary>КЛИК</summary>
    
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
