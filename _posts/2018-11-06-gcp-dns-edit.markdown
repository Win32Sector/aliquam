---
layout: post
title: Как отредактировать DNS-записи в CloudDNS в Google Cloud Platform c помощью gcloud
category: article
comments: true
description: Как отредактировать DNS-записи в CloudDNS в Google Cloud Platform c помощью gcloud
tags:
    - DevOps

---

## Понадобилось в скрипте менять записи Google CloudDNS.

Процесс выглядит следующим образом:

```
gcloud dns --project=<project_name> record-sets transaction start --zone=<zone_name>

gcloud dns --project=<project_name> record-sets transaction remove <OLD_IP> --name=<dns_name>. --ttl=300 --type=A --zone=<zone_name>

gcloud dns --project=<project_name> record-sets transaction add <NEW_IP> --name=<dns_name>. --ttl=300 --type=A --zone=<zone_name>

gcloud dns --project=<project_name> record-sets transaction execute --zone=<zone_name>
```
