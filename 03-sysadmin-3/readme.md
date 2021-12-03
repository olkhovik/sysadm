# **Домашняя работа к занятию «3.3. Операционные системы, лекция 1»**
## _Задача №1_
**Какой системный вызов делает команда `cd`? В прошлом ДЗ мы выяснили, что `cd` не является самостоятельной программой, это `shell builtin`, поэтому запустить `strace` непосредственно на `cd` не получится. Тем не менее, вы можете запустить `strace` на `/bin/bash -c 'cd /tmp'`. В этом случае вы увидите полный список системных вызовов, которые делает сам `bash` при старте. Вам нужно найти тот единственный, который относится именно к `cd`.**

```
vagrant@vagrant:/etc$ strace /bin/bash -c 'cd /tmp' 2>&1 | grep tmp
execve("/bin/bash", ["/bin/bash", "-c", "cd /tmp"], 0x7ffe0b6a7750 /* 25 vars */) = 0
stat("/tmp", {st_mode=S_IFDIR|S_ISVTX|0777, st_size=4096, ...}) = 0
chdir("/tmp")
```
команда `cd` делает такой системный вызов: **chdir("/tmp")**

## _Задача №2_
**Попробуйте использовать команду `file` на объекты разных типов на файловой системе. Например:**

```
vagrant@netology1:~$ file /dev/tty
/dev/tty: character special (5/0)
vagrant@netology1:~$ file /dev/sda
/dev/sda: block special (8/0)
vagrant@netology1:~$ file /bin/bash
/bin/bash: ELF 64-bit LSB shared object, x86-64
```
**Используя `strace` выясните, где находится база данных `file` на основании которой она делает свои догадки.**

 `strace file /dev/tty 2>&1 | grep openat`

- `strace` указывает на `/usr/share/misc/magic.mgc` как на место расположения базы данных для команды `file`; На самом деле это символьная ссылка на файл `/usr/lib/file/magic.mgc`

## _Задача №3_
**Предположим, приложение пишет лог в текстовый файл. Этот файл оказался удален (deleted в lsof), однако возможности сигналом сказать приложению переоткрыть файлы или просто перезапустить приложение – нет. Так как приложение продолжает писать в удаленный файл, место на диске постепенно заканчивается. Основываясь на знаниях о перенаправлении потоков предложите способ обнуления открытого удаленного файла (чтобы освободить место на файловой системе).**
```
vagrant@vagrant:~$ lsof -p 2994 | grep deleted_file
nano 2994 vagrant    4r   REG  253,0       25 563765 /home/vagrant/deleted_file (deleted)
vagrant@vagrant:~$ echo '' > /proc/2994/fd/4
vagrant@vagrant:~$
```


## _Задача №4_
**Занимают ли зомби-процессы какие-то ресурсы в ОС (CPU, RAM, IO)?**

Зомби-процессы освобождают ресурсы ОС, но не освобождают запись в таблице процессов.



## _Задача №5_
**В iovisor BCC есть утилита opensnoop:**

```
root@vagrant:~# dpkg -L bpfcc-tools | grep sbin/opensnoop
/usr/sbin/opensnoop-bpfcc
```
**На какие файлы вы увидели вызовы группы `open` за первую секунду работы утилиты? Воспользуйтесь пакетом `bpfcc-tools` для Ubuntu 20.04.**

Вызовы группы `open` за первую секунду работы:
```
vagrant@vagrant:~$ sudo opensnoop-bpfcc
PID    COMM               FD ERR PATH
785    vminfo              6   0 /var/run/utmp
580    dbus-daemon        -1   2 /usr/local/share/dbus-1/system-services
580    dbus-daemon        18   0 /usr/share/dbus-1/system-services
580    dbus-daemon        -1   2 /lib/dbus-1/system-services
580    dbus-daemon        18   0 /var/lib/snapd/dbus-1/system-services/
```


## _Задача №6_
**Какой системный вызов использует `uname -a`? Приведите цитату из man по этому системному вызову, где описывается альтернативное местоположение в `/proc`, где можно узнать версию ядра и релиз ОС.**

`uname({sysname="Linux", nodename="vagrant", ...}) = 0`

`uname -a` использует системный вызов `uname()`

Описание, где можно в `/proc` узнать версию ядра и релиз ОС (строка 50 man):

Part of the utsname information is also accessible via /proc/sys/kernel/{ostype, hostname, osrelease, version, domainname}.

## _Задача №7_
**Чем отличается последовательность команд через ; и через && в bash? Например:**

```
root@netology1:~# test -d /tmp/some_dir; echo Hi
Hi
root@netology1:~# test -d /tmp/some_dir && echo Hi
root@netology1:~#
```
**Есть ли смысл использовать в bash `&&`, если применить `set -e`?**

- `;` - разделитель последовательных команд; все команды выполняются независимо от статуса возврата предыдущей команды;
- `&&` - условный оператор; следующая команда выполняется только при успешном выполнении предыдущей.
- `set -e` - Exit  immediately  if  a pipeline (which may consist of a single simple command), a list, or a compound command, exits with a non-zero status.
Поэтому использовать `&&` при применении `set -e` смысла нет, т.к. при параметре `-e` команда завершит работу после первой ошибки в составном списке команд.

## _Задача №8_
**Из каких опций состоит режим bash `set -euxo pipefail` и почему его хорошо было бы использовать в сценариях?**

- `-e` - прерывает выполнение при ошибке любой команды кроме последней в последовательности
- `-u` - неустановленные переменные трактуются как ошибки
- `-x` - в стандартный поток ошибок пишется трассировка по каждой команде до её выполнения (с какими аргументами вызвана) и в процессе
- `-o pipefail` - если в ряде команд в `pipe` будет ненулевое значение, то как результат конвейера вернется этот последний ненулевой код.

Полезность использования в том, что сценарий в таком случае может не иметь обработчика ошибок (не нужно о нём думать) и не будет продолжать работу при первом появлении ошибки (так как дальнейшая его работа скорее всего будет некорректной). А если вывод сценария перенаправить в файл, то будет необходимая информация для анализа и устранения ошибки (не нужно писать дополнительный вывод в протокол).

## _Задача №9_
**Используя `-o stat` для `ps`, определите, какой наиболее часто встречающийся статус у процессов в системе. В `man ps` ознакомьтесь (`/PROCESS STATE CODES`) что значат дополнительные к основной заглавной букве статусы процессов. Его можно не учитывать при расчете (считать S, Ss или Ssl равнозначными).**

Наиболее часто встречающиеся статусы: `S` – прерывистый сон, ожидает завершения события, `R` - работает или запускается (в очереди выполнения)

- `<` - с высоким приоритетом
- `N` - с низким приоритетом
- `L` - имеет страницы, заблокированные в памяти (для реального времени и пользовательского ввода-вывода)
- `s` - лидер сессии
- `l` - многопоточный
- `+` - находится в группе процессов переднего плана