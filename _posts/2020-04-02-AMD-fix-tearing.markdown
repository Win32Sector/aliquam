---
layout: post
title: Как убрать tearing на видеокартах AMD
category: article
comments: true
description: Как убрать tearing на видеокартах AMD
tags:
    - Железо
    - Ubuntu 
    - Склерозник
---

## Как убрать tearing на видеокарте AMD в Ubuntu

Нужно добавить в каталог `/etc/X11/Xsession.d/` файл, например `20-radeon.conf` со следующим содержимым:

```
Section "Device"
    Identifier "AMD"
    Driver "amdgpu"
    Option "TearFree" "true"
EndSection
```

Про остальные видеокарты можно тут узнать https://tehnojam.pro/category/software/kak-ubrat-razryvy-jekrana-tiring-v-linux.html