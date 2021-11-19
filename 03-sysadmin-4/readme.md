# **Домашняя работа к занятию «3.4. Операционные системы, лекция 2»**
## _Задача №1_
**На лекции мы познакомились с node_exporter. В демонстрации его исполняемый файл запускался в background. Этого достаточно для демо, но не для настоящей production-системы, где процессы должны находиться под внешним управлением. Используя знания из лекции по systemd, создайте самостоятельно простой unit-файл для node_exporter:**

- поместите его в автозагрузку,
- предусмотрите возможность добавления опций к запускаемому процессу через внешний файл (посмотрите, например, на `systemctl cat cron`),
- удостоверьтесь, что с помощью systemctl процесс корректно стартует, завершается, а после перезагрузки автоматически поднимается.



_Выполнено_

## _Задача №2_
**Ознакомьтесь с опциями node_exporter и выводом `/metrics` по-умолчанию. Приведите несколько опций, которые вы бы выбрали для базового мониторинга хоста по CPU, памяти, диску и сети.**



## _Задача №3_
**Установите в свою виртуальную машину Netdata. Воспользуйтесь готовыми пакетами для установки (`sudo apt install -y netdata`). После успешной установки:**

- в конфигурационном файле `/etc/netdata/netdata.conf` в секции [web] замените значение с localhost на `bind to = 0.0.0.0`,
- добавьте в Vagrantfile проброс порта Netdata на свой локальный компьютер и сделайте `vagrant reload`:

`config.vm.network "forwarded_port", guest: 19999, host: 19999`



## _Задача №4_
**Можно ли по выводу `dmesg` понять, осознает ли ОС, что загружена не на настоящем оборудовании, а на системе виртуализации?**

Да, по выводу `dmesg` понятно, что ОС известно о своей загрузке на системе виртуализации:
```
vagrant@vagrant:~$ dmesg | grep virtual
[    0.001708] CPU MTRRs all blank - virtualized system.
[    0.100828] Booting paravirtualized kernel on KVM
[    2.419867] systemd[1]: Detected virtualization oracle.
```

## _Задача №5_
**Как настроен sysctl `fs.nr_open` на системе по-умолчанию? Узнайте, что означает этот параметр. Какой другой существующий лимит не позволит достичь такого числа (`ulimit --help`)?**

`fs.nr_open` определяет, сколько максимально может быть открыто файловых дескрипторов в системе:
```
vagrant@vagrant:~$ sudo sysctl -a | grep fs.nr_open
fs.nr_open = 1048576
```
- `ulimit -Sn` или `ulimit -n` - «мягкий» лимит для пользователя (но его можно изменить):
```
vagrant@vagrant:~$ ulimit -Sn
1024
```
- `ulimit -Hn` - «жёсткий» лимит для пользователя, не может превышать системный
```
vagrant@vagrant:~$ ulimit -Hn
1048576
```

## _Задача №6_
**Запустите любой долгоживущий процесс (не `ls`, который отработает мгновенно, а, например, `sleep 1h`) в отдельном неймспейсе процессов; покажите, что ваш процесс работает под PID 1 через `nsenter`. Для простоты работайте в данном задании под root (`sudo -i`). Под обычным пользователем требуются дополнительные опции (`--map-root-user`) и т.д.**

```
root@vagrant:/home/vagrant# unshare -f --pid --mount-proc sleep 1h

```
```
root@vagrant:/home/vagrant# lsns
        NS TYPE   NPROCS   PID USER            COMMAND
4026531835 cgroup    120     1 root            /sbin/init
4026531836 pid       119     1 root            /sbin/init
4026531837 user      120     1 root            /sbin/init
4026531838 uts       116     1 root            /sbin/init
4026531839 ipc       120     1 root            /sbin/init
4026531840 mnt       109     1 root            /sbin/init
4026531860 mnt         1    21 root            kdevtmpfs
4026531992 net       120     1 root            /sbin/init
4026532162 mnt         3   388 root            /lib/systemd/systemd-udevd
4026532163 uts         3   388 root            /lib/systemd/systemd-udevd
4026532164 mnt         1   404 systemd-network /lib/systemd/systemd-networkd
4026532183 mnt         1   555 systemd-resolve /lib/systemd/systemd-resolved
4026532184 mnt         2   932 root            unshare -f --pid --mount-proc sleep 1h
4026532185 pid         1   933 root            sleep 1h
4026532249 mnt         1   597 root            /usr/sbin/irqbalance --foreground
4026532250 mnt         1   611 root            /lib/systemd/systemd-logind
4026532251 uts         1   611 root            /lib/systemd/systemd-logind
4026532252 mnt         1   793 root            /usr/libexec/fwupd/fwupd
root@vagrant:/home/vagrant# nsenter --target 933 --pid --mount
root@vagrant:/# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   8076   596 pts/0    S+   20:29   0:00 sleep 1h
root           2  0.0  0.3   9836  3972 pts/1    S    20:31   0:00 -bash
root          11  0.0  0.3  11492  3348 pts/1    R+   20:31   0:00 ps aux
```

## _Задача №7_
**Найдите информацию о том, что такое `:(){ :|:& };:`. Запустите эту команду в своей виртуальной машине Vagrant с Ubuntu 20.04 _(это важно, поведение в других ОС не проверялось)_. Некоторое время все будет "плохо", после чего (минуты) – ОС должна стабилизироваться. Вызов `dmesg` расскажет, какой механизм помог автоматической стабилизации. Как настроен этот механизм по-умолчанию, и как изменить число процессов, которое можно создать в сессии?**

`:(){ :|:& };:`- это последовательность команд, так называемая «fork-бомба», в которой функция вызывает себя дважды, каждая из вызванных функций вызывает себя дважды и так далее в геометрической прогрессии пока не кончатся ресурсы.

`[ 2901.442847] cgroup: fork rejected by pids controller in /user.slice/user-1000.slice/session-1.scope`

Помогло стабилизировать систему ограничение на максимальное число процессов в слайсе текущего пользователя с id 1000:

```
vagrant@vagrant:~$ cat /sys/fs/cgroup/pids/user.slice/user-1000.slice/pids.max
2356
```
Изменить это значение можно так:
```
vagrant@vagrant:~$ sudo systemctl set-property user-1000.slice TasksMax=2356
```


