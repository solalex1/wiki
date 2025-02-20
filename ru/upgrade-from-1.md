# OpenIPC Wiki
[Оглавление](index.md)

Программный переход с openipc-1.0 (OpenWrt) на openipc-2.x (Buildroot) 👻
-------------------------------------------------------------------------

Заходим на устройство со старым openipc-1.0 и останавливаем любыми способами
максимум сервисов кроме dropbear. Те сервисы которые "оживают" повторно
останавливаем по примеру snmp.

```
/etc/init.d/snmpd stop; /etc/init.d/snmpd disable
```

Меняем при помощи команды `fw_setenv` переменную `bootargs`, добавляя туда в
свою очередь переменную `init=/init`. Для моей платы строка выглядит вот так,
но у вас она может быть другой:

```
fw_setenv bootargs 'console=ttyAMA 0,115200 root=/dev/mtdblock3 init=/init rootfstype=squashfs,jffs2 panic=20 mtdparts=hi_sfc:256k(boot),64k(env),2048k(kernel),5120k(rootfs),-(rootfs_data)'
```

Добавляем новую переменную soc при помощи команды `fw_setenv` указав свой
процессор:

```
fw_setenv soc hi3516ev100
```

Прошиваем командой `flashcp` файловую систему, которую предварительно скачали
с GitHub аккаунта OpenIPC. В моём случае это раздел `/dev/mtd3`, но могут быть
отличия на каких-то старых железках:

```
flashcp -v rootfs.squashfs.hi3516ev100 /dev/mtd3
```

Делаем жесткий ребут плате:

```
reboot -f
```

Загружается **недо**-openipc-2.x с получением адреса по DHCP. После этого
выполняем команду для глобального и красивого обновления:

```
sysupgrade -k -r -n
```

Профит!
