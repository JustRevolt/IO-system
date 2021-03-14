# Лабораторная работа 2

**Название:** "Разработка драйверов блочных устройств"

**Цель работы:** получить знания и навыки разработки драйверов блочных устройств 
                 для операционной системы Linux.

## Описание функциональности драйвера

1. Драйвер создает блочное устройство *myblock*.
2. Драйвер производит логирование всех действий с устройством и ошибок в кольцевой 
   буфер ядра.
3. Драйвер создает логический жесткий диск размером 50 Мбайт.
4. Драйвер разбивает логический жесткий диск на
    1. первичный раздел размером 30 Мбайт (mydisk1)
    2. расширенный раздел размером 20 Мбайт разбитый на
        1. логический раздел размером 10 Мбайт (mydisk5)
        2. логический раздел размером 10 Мбайт (mydisk6)
5. С созданным логическим диском можно производить все действия, что и с реальным
   жестким диском.

## Инструкция по сборке

1. Для сборки установочных файлов драйвера необходимо выполнить команду 
   из директории `/lab2`
```
# make Makefile
```
2. Для установки драйвера после сборки необходимо выполнить команду
```
# sudo insmod my_blk_dev.ko
```
3. Для удаления драйвера и самого блочного устройства необходимо выполнить команду
```
# sudo rmmod my_blk_dev
```
4. Для удаления всех установочных файлов необходимо выполнить команду
```
# make Makefile clean
```

## Инструкция пользователя

Для взаимодействие с разделом диска через файловую систему Linux необходимо выполнить 
следующие действия, авторизировавшись под пользователем с ***root-правами***.

1. Создать в раделе файловую систему необходимого типа с помощью команды **mkfs**
```
# mkfs -t [fstype] /dev/mydisk1
```
*`fstype` - тип файловой системы.*

2. Создать директорию для доступа к файловой системе раздела 
   с помощью команды **mkdir**
```
# mkdir /mnt/mydisk1
```
3. Смонтировать раздел диска в созданную директории с помощью команды **mount**
```
# mount /dev/mydisk1 /mnt/mydisk1
```

---

Для удаления раздела из файловой системы Linux необходимо выполнить 
следующие действия, авторизировавшись под пользователем с ***root-правами***.

1. Размонтировать раздел диска из директорий с помощью команды **umount**
```
# umount /mnt/mydisk1
# umount /mnt/mydisk5
# umount /mnt/mydisk6
```
2. Отформатировать раздел с помощью команды **mkfs**
```
# mkfs /dev/mydisk1
```
3. Удалить директорию, которая использовалась для доступа к файловой системе раздела 
   с помощью команды **rmdir**
```
# rmdir /mnt/mydisk1
```

---

Для просмотра структуры диска необходимо использовать утилиты для просмотра информации 
о дисках. Одной из таких утилит является утилита **fdisk**
```
# sudo fdisk -l /dev/mydisk
```

---

Для просмотра кольцевого буфера ядра, в котором происходит логирование операций
чтения/запись в блочное устройство *mydisk*, необходимо использовать команду 
```
dmesg
```

---

Для тестирования скорости передачи данных между разделами любых дисков неоюходимо
использовать скрипт **speed_benchmark**, авторизировавшись под пользователем с ***root-правами***.
```
./speed_benchmark *disk1 disk2 disk3 ...*
```
*`disk[N]` - имена дисков, между которыми необходимо протестировать скорости передачи данных*

## Примеры использования
**Установка драйвера**
```
root# sudo insmod my_blk_dev.ko
root# dmesg 
[  209.668376] Major Number is : 252
[  209.670368] THIS IS DEVICE SIZE 102400
[  209.671897] mydiskdrive : open 
[  209.671933] my disk: Start Sector: 0, Sector Offset: 0;		Buffer: 000000002c671a74; Length: 8 sectors
[  209.671946] my disk: Start Sector: 61440, Sector Offset: 0;		Buffer: 00000000d455ee7c; Length: 8 sectors
[  209.671951] my disk: Start Sector: 81920, Sector Offset: 0;		Buffer: 000000001a309dcb; Length: 8 sectors
[  209.671954]  mydisk: mydisk1 mydisk2 < mydisk5 mydisk6 >
[  209.673215] mydisk: p6 size 20480 extends beyond EOD, truncated
[  209.673251] mydiskdrive : closed 
[  209.673726] mydiskdrive : open 
[  209.674478] mydiskdrive : closed 
[  209.685529] mydiskdrive : open 
[  209.686255] mydiskdrive : closed 
[  209.688847] mydiskdrive : open 
[  209.689565] mydiskdrive : closed 
[  209.690112] mydiskdrive : open 
[  209.690815] mydiskdrive : closed 
[  209.693062] mydiskdrive : open 
[  209.699778] mydiskdrive : closed 
```
**Просмотр информации о диске**
```
root# sudo fdisk -l /dev/mydisk
Disk /dev/mydisk: 50 MiB, 52428800 bytes, 102400 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x36e5756d

Device       Boot Start    End Sectors Size Id Type
/dev/mydisk1          1  61440   61440  30M 83 Linux
/dev/mydisk2      61441 102400   40960  20M  5 Extended
/dev/mydisk5      61442  81921   20480  10M 83 Linux
/dev/mydisk6      81923 102402   20480  10M 83 Linux

root# dmesg 
[ 3332.351087] mydiskdrive : open 
[ 3332.351139] my disk: Start Sector: 0, Sector Offset: 0;		Buffer: 000000005f352385; Length: 8 sectors
[ 3332.351160] my disk: Start Sector: 61440, Sector Offset: 0;		Buffer: 00000000e751b6c2; Length: 8 sectors
[ 3332.351174] my disk: Start Sector: 81920, Sector Offset: 0;		Buffer: 00000000b35b8287; Length: 8 sectors
[ 3332.352038] mydiskdrive : closed 
```
**Форматирование раздела**
```
root# mkfs -t vfat /dev/mydisk1
mkfs.fat 4.1 (2017-01-24)
root# dmesg 
[ 3400.506396] mydiskdrive : open 
[ 3400.506734] my disk: Start Sector: 1, Sector Offset: 0;		Buffer: 00000000e3d2ed16; Length: 8 sectors
[ 3400.506858] my disk: Start Sector: 9, Sector Offset: 0;		Buffer: 00000000732d37ff; Length: 8 sectors
[ 3400.506877] my disk: Start Sector: 17, Sector Offset: 0;		Buffer: 000000001af633b3; Length: 8 sectors
[ 3400.506894] my disk: Start Sector: 25, Sector Offset: 0;		Buffer: 00000000606b95dc; Length: 8 sectors
[ 3400.506910] my disk: Start Sector: 33, Sector Offset: 0;		Buffer: 000000006096062d; Length: 8 sectors
[ 3400.506926] my disk: Start Sector: 41, Sector Offset: 0;		Buffer: 00000000857a9102; Length: 8 sectors
[ 3400.506942] my disk: Start Sector: 49, Sector Offset: 0;		Buffer: 0000000086e488ff; Length: 8 sectors
[ 3400.506958] my disk: Start Sector: 57, Sector Offset: 0;		Buffer: 000000003d7259dd; Length: 8 sectors
[ 3400.506975] my disk: Start Sector: 65, Sector Offset: 0;		Buffer: 00000000e6a14cfd; Length: 8 sectors
[ 3400.506991] my disk: Start Sector: 73, Sector Offset: 0;		Buffer: 00000000998e146a; Length: 8 sectors
[ 3400.507008] my disk: Start Sector: 81, Sector Offset: 0;		Buffer: 000000004afbbec0; Length: 8 sectors
[ 3400.507025] my disk: Start Sector: 89, Sector Offset: 0;		Buffer: 000000005333fdb5; Length: 8 sectors
[ 3400.507063] my disk: Start Sector: 97, Sector Offset: 0;		Buffer: 000000002029526b; Length: 8 sectors
[ 3400.507084] my disk: Start Sector: 105, Sector Offset: 0;		Buffer: 000000002a853cc9; Length: 8 sectors
[ 3400.507100] my disk: Start Sector: 113, Sector Offset: 0;		Buffer: 0000000047f7312f; Length: 8 sectors
[ 3400.507116] my disk: Start Sector: 121, Sector Offset: 0;		Buffer: 0000000021bc0317; Length: 8 sectors
[ 3400.507133] my disk: Start Sector: 153, Sector Offset: 0;		Buffer: 00000000b677e03a; Length: 8 sectors
[ 3400.507228] my disk: Start Sector: 1, Sector Offset: 0;		Buffer: 00000000e3d2ed16; Length: 8 sectors
[ 3400.507229] my disk: Start Sector: 1, Sector Offset: 8;		Buffer: 00000000732d37ff; Length: 8 sectors
[ 3400.507230] my disk: Start Sector: 1, Sector Offset: 16;		Buffer: 000000001af633b3; Length: 8 sectors
[ 3400.507232] my disk: Start Sector: 1, Sector Offset: 24;		Buffer: 00000000606b95dc; Length: 8 sectors
[ 3400.507233] my disk: Start Sector: 1, Sector Offset: 32;		Buffer: 000000006096062d; Length: 8 sectors
[ 3400.507234] my disk: Start Sector: 1, Sector Offset: 40;		Buffer: 00000000857a9102; Length: 8 sectors
[ 3400.507235] my disk: Start Sector: 1, Sector Offset: 48;		Buffer: 0000000086e488ff; Length: 8 sectors
[ 3400.507237] my disk: Start Sector: 1, Sector Offset: 56;		Buffer: 000000003d7259dd; Length: 8 sectors
[ 3400.507238] my disk: Start Sector: 1, Sector Offset: 64;		Buffer: 00000000e6a14cfd; Length: 8 sectors
[ 3400.507239] my disk: Start Sector: 1, Sector Offset: 72;		Buffer: 00000000998e146a; Length: 8 sectors
[ 3400.507240] my disk: Start Sector: 1, Sector Offset: 80;		Buffer: 000000004afbbec0; Length: 8 sectors
[ 3400.507242] my disk: Start Sector: 1, Sector Offset: 88;		Buffer: 000000005333fdb5; Length: 8 sectors
[ 3400.507243] my disk: Start Sector: 1, Sector Offset: 96;		Buffer: 000000002029526b; Length: 8 sectors
[ 3400.507244] my disk: Start Sector: 1, Sector Offset: 104;		Buffer: 000000002a853cc9; Length: 8 sectors
[ 3400.507245] my disk: Start Sector: 1, Sector Offset: 112;		Buffer: 0000000047f7312f; Length: 8 sectors
[ 3400.507246] my disk: Start Sector: 1, Sector Offset: 120;		Buffer: 0000000021bc0317; Length: 8 sectors
[ 3400.507248] my disk: Start Sector: 1, Sector Offset: 128;		Buffer: 000000004fe3186b; Length: 8 sectors
[ 3400.507249] my disk: Start Sector: 1, Sector Offset: 136;		Buffer: 000000004623b112; Length: 8 sectors
[ 3400.507250] my disk: Start Sector: 1, Sector Offset: 144;		Buffer: 00000000857553ee; Length: 8 sectors
[ 3400.507252] my disk: Start Sector: 1, Sector Offset: 152;		Buffer: 00000000b677e03a; Length: 8 sectors
[ 3400.507281] mydiskdrive : closed 
```
**Тестирование скорости передачи данных между разделами логического диска**
```
root# ./speed_benchmark mydisk5 mydisk6
/dev/mydisk5->/dev/mydisk5 Time:0.0279000700 s, Speed:42.14 MB/s
/dev/mydisk5->/dev/mydisk6 Time:0.0253471800 s, Speed:46.29 MB/s
/dev/mydisk6->/dev/mydisk5 Time:0.0272595400 s, Speed:42.08 MB/s
/dev/mydisk6->/dev/mydisk6 Time:0.0305916900 s, Speed:37.94 MB/s
```
**Тестирование скорости передачи данных между разделами логического и реального диска**
```
root# ./speed_benchmark sda1 mydisk1
/dev/sda1->/dev/sda1 Time:0.0449291900 s, Speed:27.40 MB/s
/dev/sda1->/dev/mydisk1 Time:0.0177806400 s, Speed:61.51 MB/s
/dev/mydisk1->/dev/sda1 Time:0.0731654800 s, Speed:15.17 MB/s
/dev/mydisk1->/dev/mydisk1 Time:0.0284069100 s, Speed:45.80 MB/s
```
