# OpenIPC Wiki
[Оглавление](index.md)

Switcam HS-303
--------------

Компания-производитель выпускала IP камеры Switcam HS-303 в трёх версиях,
значительно отличающихся аппаратно между собой. На данный момент проект
OpenIPC поддерживает все три типа камер, однако установка альтернативного ПО
без разборки устройства возможна только на v1 и v2, причем на первый тип камеры
устанавливается более старая версия на базе OpenWrt.

Мы работаем над созданием новой универсальной прошивки для всех трёх типов
видеокамеры Switcam HS-303. Обсуждение проекта и возможностей прошивок (на
русском языке) возможно в открытой группе [Telegram](https://t.me/openipc_modding).

Так-же в ближайшее время все информационные наработки по видеокамерам в виде 
Q&A будут перенесены в данный документ общей Wiki.


## HS303(v1)

### Наиболее актуальные вопросы и ответы

#### Где можно взять прошивку для устройства Ростелеком Switcam HS303(v2)?

Обсуждение работы этих камер доступно по специальной подписке на
[OpenIPC paywall](https://paywall.pw/openipc).

#### Какой путь до SD карты необходимо указывать в `majestic.yaml` ?

`/mnt/mmc/%Y/%m/%d/%H.mp4`



## HS303(v2)

### Краткая инструкция установки OpenIPC v2.2

- отформатируйте SD/MMC карту как FAT (ограничение 2GB)
- распакуйте содержимое архива в корень SD/MMC карты
- вставьте карту памяти и подайте питание на камеру
- через 60 секунд (обновится загрузчик) сделайте выкл/вкл питания
- через 90 секунд камера щелкнет ИК фильтром два раза
- вытащите SD/MMC карту, камера готова к эксплуатации

По-умолчанию камера пытается соединиться с точкой доступа с именем OpenIPC_NFS
и паролем сети project2021. Для изменения имени сети и пароля перед прошивкой
устройства, откройте файл autoconfig/etc/network/interfaces на SD/MMC карте в
редакторе Notepad++ (для Windows), выбрав при этом кодировку UTF-8 и впишите
свои значения.

После прошивки устройства не забудьте удалить все файлы autoupdate* так как
при каждой подаче питания камера будет пытаться обновиться, что в итоге может
привести к неисправности.

### Наиболее актуальные вопросы и ответы

#### Где можно взять прошивку для устройства Ростелеком Switcam HS303(v2)?

Обсуждение работы этих камер доступно по специальной подписке на
[OpenIPC paywall](https://paywall.pw/openipc).

#### Как узнать какой IP адрес у прошитой камеры и ка зайти на неё ?

Если внесены корректные данные по настройке WiFi интерфейса в конфигурационный
фаил (SSID и ключ), то вы можете найти IP адрес камеры на своём роутере в списке
подключенных устройств с пометкой "OpenIPC".
Интерфейс управления камерой доступен в браузере на порту 85, а доступ по SSH
возможен на стандартном порту 22 с использованием логина root, без пароля при
первом подключении.

### Специализированные настройки для Switcam HS303(v2)

#### Модифицированный блок из файла /etc/network/interface

```
auto wlan0
iface wlan0 inet dhcp
    pre-up echo 54 > /sys/class/gpio/export
    pre-up echo out > /sys/class/gpio/gpio54/direction
    pre-up echo 1 > /sys/class/gpio/gpio54/value
    pre-up modprobe r8188eu
    pre-up wpa_passphrase "OpenIPC" "openipc2021" >/tmp/wpa_supplicant.conf
    pre-up sed -i '2i \\tscan_ssid=1' /tmp/wpa_supplicant.conf
    pre-up ifconfig wlan0 up
    pre-up wpa_supplicant -B -Dwext -iwlan0 -c/tmp/wpa_supplicant.conf
    post-down killall -q wpa_supplicant
```

#### Модифицированный блок из файла /etc/majestic.yaml

```
nightMode:
  enabled: true
  irSensorPin: 62
  irSensorPinInvert: true
  irCutPin1: 2
  pinSwitchDelayUs: 150
  backlightPin: 56
  nightAPI: true
```
