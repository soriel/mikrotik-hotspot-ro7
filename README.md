# mikrotik-hotspot-ro7
Mikrotik hotspot for RouterOS 7


Представленный скрипт работает на RouterOS от 7.11.2. Последовательность действий:

1)Осуществите базовую настройку hotspot
/ip hotspot setup

2)далее скорректируйте авторизацию в профиле сервера (/ip hotspot profile) login by: HTTP CHAP, HTTP PAP, Cookie, MAC Cookie (HTTPS не работает - тк нужен не шифрованный логин). А так-же RADIUSю

Для удобства сбора параллельной информации о начале и конце использования интернета добавьте скрипты на роутер с HOTSPOT из
ROS7 Log-in script on hotspot profile
ROS7 Log-out script on hotspot profile
Для отсылки сообщений о начале и конце сессии клиента hotspot по электронной почте необходимо заранее настроить учетную запись для отправки в /tool e-mail, последняя строка-за комментарием, относится к выводу уведомления об авторизации в лог-может быть полезно для сбора данных в одном дистанционном syslog-сервере.
На устройствах с 16Mb памяти могут быть проблемы с дисковым местом

3)Настраиваем radius - замените на ip-адрес или fqdn Usermanager:
/radius
add accounting-backup=yes address=127.0.0.1 secret="PASSWORD" service=\
    hotspot src-address=127.0.0.1
/radius incoming
set accept=yes

4)Настроим User-Manager.
Установите пакет usermanager из extrapackedges под свою архитектуру и версию ROS

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
#заменить 127.0.0.1 - на ip роутера с hotspot
/user-manager router
add address=127.0.0.1 name="ROUTER-NAME" shared-secret="PASSWORD"

5)Добавляем скрипт на авторизацию (ROS7 script on userman server) и ставим его в исполнение в (скорость выполнения и частота зависит от вашей платформы)
/system script add name=userman
/system scheduler add disabled=yes interval=30s name=hotspot on-event=\ "/system script run userman" policy=ftp,read,write,policy,test,password,sniff,sensitive start-date=2018-03-14 start-time=13:59:23
6) В скрипте userman заменяем строку генерации sms для вашего sms провайдера (можно сделать через USB модемы)
7) Замените папку на hotspot на папку из репозитария

8)(РЕКОМЕНДУЕТСЯ)Сделать правило ежедневного копирования БД UM (Мы ведь обязаны хранить данные 6 месяцев) предварительно настроив /tool email
ROS7-script UM backup вам поможет с копированием только UM.
ROS7-script all backup делает копии: Полную копию системы, скриптовую копию и копию базы UM.(копировать конфиги можно реже)

За адаптацию под ROS спасибо Калугину Артёму из компании ООО "Къюсар"
