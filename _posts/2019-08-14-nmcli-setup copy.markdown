---
layout: post
title: Настройка сетевого соединения со статическим адресом в CentOS c помощью nmcliы
category: article
comments: true
description: Настройка сетевого соединения со статическим адресом в CentOS c помощью nmcli
tags:
    - Linux
    - Склерозник
---

## Настройка сетевого соединения со статическим адресом в CentOS c помощью nmcli

Смотрим имеющиеся соединения
```
nmcli con show
```
Удаляем существующее соединение, так как вместо него мы создадим новое

```
nmcli con add type ethernet con-name eno14082019 ifname enp0s3 ip4 192.168.1.50/24 gw4 192.168.1.1
```

ВАЖНО! вместо enp0s3 вставляем то имя интерфейса, которое мы видели в талице выводе команды nmcli con show в последнем столбце

Добавляем DNS-ы

```
nmcli con mod eno14082019 ipv4.dns "8.8.8.8 8.8.4.4"
```

DNS-ы выставляем наши

Статья с дополнительными возможностями 

https://www.tecmint.com/configure-network-connections-using-nmcli-tool-in-linux/