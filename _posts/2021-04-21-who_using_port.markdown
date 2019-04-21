---
layout: post
title: Три способа посмотреть кто занял порт в linux
category: article
comments: true
description: Три способа посмотреть кто занял порт в linux
tags:
    - Linux
    - Склерозник
---

## Три способа посмотреть кто занял порт в linux

netstat -tulpen | grep port

ss -tuna | grep port

lsof -i :port

Анализ удаленных сервисов
