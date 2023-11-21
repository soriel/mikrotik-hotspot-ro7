Представленный скрипт работает на RouterOS от 7.11.2. Последовательность действий:
1)Осуществите базовую настройку hotspot
/ip hotspot setup

2)далее скорректируйте авторизацию в профиле сервера (/ip hotspot profile) login by: HTTP CHAP, HTTP PAP, Cookie, MAC Cookie. А так-же RADIUSю
для удобства сбора параллельной информации о начале и конце использования интернета добавьте скрипты из
ROS7 Log-in script on hotspot profile
ROS7 Log-out script on hotspot profile
Для отсылки сообщений о начале и конце сессии клиента hotspot по электронной почте необходимо заранее настроить учетную запись для отправки в /tool e-mail, последняя строка-за комментарием, относится к выводу уведомления об авторизации в лог-может быть полезно для сбора данных в одном дистанционном syslog-сервере.
3)Настраиваем radius:
/radius
add accounting-backup=yes address=127.0.0.1 secret="PASSWORD" service=\
    hotspot src-address=127.0.0.1
/radius incoming
set accept=yes

4)Настроим User-Manager.

/user-manager attribute
add name=Phone type-id=31 value-type=string vendor-id=Mikrotik
/user-manager limitation
add name=guest rate-limit-burst-rx=20000000B rate-limit-burst-threshold-rx=9000000B rate-limit-burst-threshold-tx=9000000B rate-limit-burst-time-rx=20m rate-limit-burst-time-tx=20m rate-limit-burst-tx=20000000B rate-limit-rx=10000000B rate-limit-tx=10000000B reset-counters-interval=daily
/user-manager profile
add name=guest-prof name-for-users=guest override-shared-users=5 starts-when=first-auth
/user-manager
set certificate=*0 enabled=yes
/user-manager profile-limitation
add limitation=guest profile=guest-prof
/user-manager router
add address=127.0.0.1 name="ROUTER-NAME" shared-secret="PASSWORD"
5)Добавляем скрипт на авторизацию (ROS7 script on userman server) и ставим его в исполнение в /system scheduler:

add disabled=yes interval=30s name=hotspot on-event=\
    "/system script run userman" policy=\
    ftp,read,write,policy,test,password,sniff,sensitive start-date=2018-03-14 \
    start-time=13:59:23

6)(РЕКОМЕНДУЕТСЯ)Сделать правило ежедневного копирования БД UM (Мы ведь обязаны хранить данные 6 месяцев)
ROS7-script UM backup вам поможет с копированием только UM.
ROS7-script all backup делает копии: Полную копию системы, скриптовую копию и копию базы UM.(копировать конфиги можно реже)