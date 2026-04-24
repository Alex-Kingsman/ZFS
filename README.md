````markdown id="8k2lq1"
# Определить алгоритм с наилучшим сжатием

## Список дисков
```bash
lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0 17.9G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0  1.8G  0 part /boot
└─sda3                      8:3    0 16.1G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0 11.2G  0 lvm  /
sdb                         8:16   0  512M  0 disk
sdc                         8:32   0  512M  0 disk
sdd                         8:48   0  512M  0 disk
sde                         8:64   0  512M  0 disk
sdf                         8:80   0  512M  0 disk
sdg                         8:96   0  512M  0 disk
sdh                         8:112  0  512M  0 disk
sdi                         8:128  0  512M  0 disk
sr0                        11:0    1 1024M  0 rom
````

## Установка пакета утилит для ZFS

```bash
sudo apt install zfsutils-linux
```

## Создаём пулы otus1,otus2,otus3, otus4 из двух дисков в режиме RAID 1

```bash
zpool create otus1 mirror /dev/sdb /dev/sdc
zpool create otus2 mirror /dev/sdd /dev/sde
zpool create otus3 mirror /dev/sdf /dev/sdg
zpool create otus4 mirror /dev/sdh /dev/sdi
```

## Проверяем

```bash
zpool list
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
otus1   480M   111K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus2   480M   114K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus3   480M   117K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus4   480M   104K   480M        -         -     0%     0%  1.00x    ONLINE  -
```

## Добавляем алгоритмы сжатия

```bash
zfs set compression=lzjb otus1
zfs set compression=lz4 otus2
zfs set compression=gzip-9 otus3
zfs set compression=zle otus4
```

## Проверяем

```bash
zfs get all | grep compression
otus1  compression           lzjb                   local
otus2  compression           lz4                    local
otus3  compression           gzip-9                 local
otus4  compression           zle                    local
```

## Скачиваем один и тот же текстовый файл во все пулы

```bash
for i in {1..4}; do wget -P /otus$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done
```

## Проверяем

```bash
ls -l /otus*

/otus1:
total 22123
-rw-r--r-- 1 root root 41227642 Apr  2 07:31 pg2600.converter.log

/otus2:
total 18019
-rw-r--r-- 1 root root 41227642 Apr  2 07:31 pg2600.converter.log

/otus3:
total 10972
-rw-r--r-- 1 root root 41227642 Apr  2 07:31 pg2600.converter.log

/otus4:
total 40290
-rw-r--r-- 1 root root 41227642 Apr  2 07:31 pg2600.converter.log
```

## Проверяем

```bash
zfs list
NAME    USED  AVAIL  REFER  MOUNTPOINT
otus1  21.8M   330M  21.6M  /otus1
otus2  17.7M   334M  17.6M  /otus2
otus3  10.9M   341M  10.7M  /otus3
otus4  39.5M   313M  39.4M  /otus4
```

## Проверяем

```bash
zfs get all | grep compressratio | grep -v ref
otus1  compressratio         1.82x                  -
otus2  compressratio         2.23x                  -
otus3  compressratio         3.66x                  -
otus4  compressratio         1.00x                  -
```

gzip-9 лучшая степень сжатия 3.65x

---

# Определить настройки пула

## Скачиваем архив в домашний каталог

```bash
wget -O archive.tar.gz --no-check-certificate 'https://drive.usercontent.google.com/download?id=1MvrcEp-WgAQe57aDEzxSRalPAwbNN1Bb&export=download'
2026-04-24 06:52:28 (7.59 MB/s) - ‘archive.tar.gz’ saved [7275140/7275140]
```

## Разархивируем

```bash
tar -xzvf archive.tar.gz
zpoolexport/
zpoolexport/filea
zpoolexport/fileb
```

## Пробуем импортировать данный каталог в пул

```bash
zpool import -d zpoolexport/
   pool: otus
     id: 6554193320433390805
  state: ONLINE
status: Some supported features are not enabled on the pool.
        (Note that they may be intentionally disabled if the
        'compatibility' property is set.)
 action: The pool can be imported using its name or numeric identifier, though
        some features will not be available without an explicit 'zpool upgrade'.
 config:

        otus                         ONLINE
          mirror-0                   ONLINE
            /root/zpoolexport/filea  ONLINE
            /root/zpoolexport/fileb  ONLINE
```

## Импорт данного пула в ОС

```bash
zpool import -d zpoolexport/ otus
root@srv:~# zpool status
  pool: otus
 state: ONLINE
status: Some supported and requested features are not enabled on the pool.
        The pool can still be used, but some features are unavailable.
action: Enable all features using 'zpool upgrade'. Once this is done,
        the pool may no longer be accessible by software that does not support
        the features. See zpool-features(7) for details.
config:

        NAME                         STATE     READ WRITE CKSUM
        otus                         ONLINE       0     0     0
          mirror-0                   ONLINE       0     0     0
            /root/zpoolexport/filea  ONLINE       0     0     0
            /root/zpoolexport/fileb  ONLINE       0     0     0

errors: No known data errors
```

```bash
zpool status
  pool: otus
 state: ONLINE
status: Some supported and requested features are not enabled on the pool.
        The pool can still be used, but some features are unavailable.
action: Enable all features using 'zpool upgrade'. Once this is done,
        the pool may no longer be accessible by software that does not support
        the features. See zpool-features(7) for details.
config:

        NAME                         STATE     READ WRITE CKSUM
        otus                         ONLINE       0     0     0
          mirror-0                   ONLINE       0     0     0
            /root/zpoolexport/filea  ONLINE       0     0     0
            /root/zpoolexport/fileb  ONLINE       0     0     0

errors: No known data errors

  pool: otus1
 state: ONLINE
config:

        NAME        STATE     READ WRITE CKSUM
        otus1       ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            sdb     ONLINE       0     0     0
            sdc     ONLINE       0     0     0

errors: No known data errors

  pool: otus2
 state: ONLINE
config:

        NAME        STATE     READ WRITE CKSUM
        otus2       ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            sdd     ONLINE       0     0     0
            sde     ONLINE       0     0     0

errors: No known data errors
```

```bash
zfs get available otus
NAME  PROPERTY   VALUE  SOURCE
otus  available  350M   -

zfs get readonly otus
NAME  PROPERTY  VALUE   SOURCE
otus  readonly  off     default

zfs get recordsize otus
NAME  PROPERTY    VALUE    SOURCE
otus  recordsize  128K     local

zfs get compression otus
NAME  PROPERTY     VALUE           SOURCE
otus  compression  zle             local

zfs get checksum otus
NAME  PROPERTY  VALUE      SOURCE
otus  checksum  sha256     local
```

---

# Работа со снапшотами

## Скачиваем файл

```bash
wget -O otus_task2.file --no-check-certificate https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI&export=download
```

## Восстановим файловую систему из снапшота

```bash
zfs receive otus/test@today < otus_task2.file
```

## Ищем файл

```bash
find /otus/test -name "secret_message"
/otus/test/task1/file_mess/secret_message
```

## Смотрим содержимое

```bash
cat /otus/test/task1/file_mess/secret_message
https://otus.ru/lessons/linux-hl/
```

```
```
