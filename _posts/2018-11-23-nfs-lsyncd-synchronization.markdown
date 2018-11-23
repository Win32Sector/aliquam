---
layout: post
title: Настройка синхронизации файлов между несколькими серверами с помощью lsyncd
category: article
comments: true
description: В данной статье я рассмотрю решение проблемы синхронизации файлов между несколькими серверами с помощью lsyncd, NFS и тюнинга ядра.
tags:
    - DevOps
    - NFS
    - lsyncd

---

Начальная ситуация: есть несколько нод веб-сервера, картинки проекта подключены симлинком и физически лежат на хранилище, подключенном по NFS. Отсюда имеем одну точку отказа - лежит NFS - нет картинок, ну и скорость чтения так себе учитывая накладные расходы на сетевую передачу.

Что будем реализовывать: несколько веб-серверов, картинки лежат на них локально на отдельно подключенном быстром диске и синхронизируют картинки через отдельное NFS хранилище. Таким образом - чтение становится быстрее за счет нахождения файлов локально + в случае падения NFS хранилища, проект продолжает работать, а так как картинки в основном читаются, а пишутся редко - 1-2 раза в день, можно в спокойном режиме переподнять NFS хранилище для обмена, чтобы растащить новые картинки по остальным нодам.

![alt text](http://devopspath.ru/resources/images/nfs.png "DevOps")

Для реализации замысла был выбран lsyncd, так как другие решения чем-то не устроили:

- DRBD - идеально для синхронизации между двумя серверами, но в более сложных конфигурациях нужно сильно велосипедить;
- Ceph - слишком сложно для данного кейса;
- GlusterFS - оказалось медленнее NFS в моем случае;

Итак, реализация.

### Выбираем/создаем каталог для синхронизации и создаем его на обоих серверах.

### Создаем ключи для пользователя, от имени которого будет происходить синхронизация

```
ssh-keygen
```

И добавляем ключи на сервер. Чтобы это сделать, копируем содержимое созданного файла `~/.ssh/id_rsa.pub` с сервера на котором создали ключ для подключения, в файл `~/.ssh/authorized_keys` на сервере, к которому будет подключаться.

Пробуем подключиться к целевому серверу:

```
ssh username@server
```
Подключение должно пройти успешно.
Повторяем 1 шаг на втором сервере и добавляем ключ пользователя со второго сервера, на первый.

### Устанавливаем lsyncd на обоих серверах

```
sudo yum install lsyncd -y
```

### Редактируем conf-файл

Файл `/etc/lsyncd.conf` пишется в формате lua. Самое простое сделать один конфиг со всеми нодами и скопировать его на сервера, просто закомментировав ненужные куски конфига.

У меня файл получился примерно таким для сервера nfs-exchange, через который будет происходить обмен:

```
settings {
  logfile    = "/var/log/lsyncd/lsyncd.log", --[[log-файл с событиями синхронизации и ошибками]]
  statusFile = "/var/log/lsyncd/lsyncd.status", --[[файл, где указано какие каталоги отслеживает inotify и какие каталоги будут синхронизированы]]
  statusInterval = 5, --[[интервал обновления файла lsyncd.status в секундах]]
}

--[[
sync {
  default.rsyncssh, --[[выбор протокола синхронизации]]
  source = "/nfssync/pics", --[[каталог-источник для синхронизации]]
  host = "nfs-exchange", --[[целевой хост, куда каталог будет синхронизироваться]]
  targetdir = "/nfssync/pics", --[[целевой каталог на сервере, заданной в параметре host]]
  rsync = {
       _extra = { "-ausS", "--temp-dir=/tmp", "-e", "/usr/bin/ssh -l username -i /home/username/.ssh/id_rsa -o StrictHostKeyChecking=no" }
  }, 
  --[[дополнительные параметры синхронизации:
    a — режим архивации; равносильно -rlptgoD, тоесть копировать рекурсивно, вместе с симлинками и специальными файлами, сохраняя права доступа, группу, владельца.
    u — не обновлять файлы на получателе если они новее;    
    s — на случай если в имени файла попадется пробел.
    S — оптимизировать передачу данных состоящих из нулей.
    temp-dir нужна если синхронизация двухсторонняя
    далее я задал пользователя и его ключ, чтобы синхронизация происходила под пользователем, для которого мы ранее создали ключи и раскидали их по серверам.
    Опция -o StrictHostKeyChecking=no решает проблему ошибки Permission denied.
    Более подробно в man и тут:
    https://github.com/mralexjuarez/lsyncd-tutorial
            
        ]]
  delay = 3, --[[пауза между запуском синхронизации]]
}
]]

sync {
  default.rsyncssh,
  source = "/nfssync/pics",
  host = "node01",
  targetdir = "/nfssync/pics",
  rsync = {
       _extra = { "-ausS", "--temp-dir=/tmp", "-e", "/usr/bin/ssh -l username -i /home/username/.ssh/id_rsa -o StrictHostKeyChecking=no" }
  },
  delay = 3
}


sync {
  default.rsyncssh,
  source = "/nfssync/pics",
  host = "node02",
  targetdir = "/nfssync/pics",
  rsync = {
       _extra = { "-ausS", "--temp-dir=/tmp", "-e", "/usr/bin/ssh -l username -i /home/username/.ssh/id_rsa -o StrictHostKeyChecking=no" }
  },
  delay = 3,
}

sync {
  default.rsyncssh,
  source = "/nfssync/pics",
  host = "node03",
  targetdir = "/nfssync/pics",
  rsync = {
       _extra = { "-ausS", "--temp-dir=/tmp", "-e", "/usr/bin/ssh -l username -i /home/username/.ssh/id_rsa -o StrictHostKeyChecking=no" }
  },
  delay = 3,
}

```
На остальных нодах закомментированы все ноды и раскомментирован только участок конфига, касающийся хоста nfs-exchange.

```
settings {
  logfile    = "/var/log/lsyncd/lsyncd.log",
  statusFile = "/var/log/lsyncd/lsyncd.status",
  statusInterval = 5,
}


sync {
  default.rsyncssh,
  source = "/nfssync/pics",
  host = "nfs-exchange",
  targetdir = "/nfssync/pics",
  rsync = {
       _extra = { "-ausS", "--temp-dir=/tmp", "-e", "/usr/bin/ssh -l username -i /home/username/.ssh/id_rsa -o StrictHostKeyChecking=no" }
  },
  delay = 3,
}


--[[
sync {
  default.rsyncssh,
  source = "/nfssync/pics",
  host = "node01",
  targetdir = "/nfssync/pics",
  rsync = {
       _extra = { "-ausS", "--temp-dir=/tmp", "-e", "/usr/bin/ssh -l username -i /home/username/.ssh/id_rsa -o StrictHostKeyChecking=no" }
  },
  delay = 3
}


sync {
  default.rsyncssh,
  source = "/nfssync/pics",
  host = "node02",
  targetdir = "/nfssync/pics",
  rsync = {
       _extra = { "-ausS", "--temp-dir=/tmp", "-e", "/usr/bin/ssh -l username -i /home/username/.ssh/id_rsa -o StrictHostKeyChecking=no" }
  },
  delay = 3,
}

sync {
  default.rsyncssh,
  source = "/nfssync/pics",
  host = "node03",
  targetdir = "/nfssync/pics",
  rsync = {
       _extra = { "-ausS", "--temp-dir=/tmp", "-e", "/usr/bin/ssh -l username -i /home/username/.ssh/id_rsa -o StrictHostKeyChecking=no" }
  },
  delay = 3,
}
]]

```

Когда конфиги созданы, можем включать lsyncd с обоих сторон на серверах. Желательно, конечно, включить с одной стороны, посмотреть log, если все ок, включать с другой стороны.

```
systemctl start lsyncd
systemctl enable lsyncd
```

## Тюнинг ядра

Также, для того, чтобы inotify смог отслеживать большое количество каталогов, необходимо увеличить значения параметров ядра 

```
echo "
fs.inotify.max_user_watches = 16777216  
fs.inotify.max_queued_events = 65536
" >> /etc/sysctl.conf
```

Если все сделано правильно, файлы должны синхронизироваться.

Далее, можно и нужно настроить мониторинг службы lsyncd с помощью monit, например, поэтому мануалу https://itldc.com/blog/monit-monitoring или какому-то другому.
