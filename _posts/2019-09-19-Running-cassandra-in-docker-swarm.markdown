---
layout: post
title: Запуск кластера cassandra в docker swarm
category: article
comments: true
description: Запуск кластера cassandra в docker swarm
tags:
    - Linux
    - DevOps
---

## Запуск кластера cassandra в docker swarm

1. Подготавливаем 3 виртуальных машины

- Ставим последние обновления
- Устанавливаем docker и docker-compose
- Открываем соответствующие порты на всех нодах:

    Для swarm:
```
      - 22/tcp
      - 2376/tcp
      - 2377/tcp
      - 7946/tcp
      - 7946/udp
      - 4789/udp
```

    Для cassandra:

```
      - 7000/tcp
      - 9042/tcp
```

- Одну из нод делаем мастером, остальные джойним как воркеры
- Добавляем Labels для каждой ноды:

```
  docker node update --label-add cassandra_1 node_1
  docker node update --label-add cassandra_2 node_2
  docker node update --label-add cassandra_3 node_3
```

2. Копируем на мастер ноду следующий docker-compose.yaml:

```
version: "3"
services:
  cassandra_1:
    image: cassandra:latest
    command: bash -c 'if [ -z "$$(ls -A /var/lib/cassandra/)" ] ; then sleep 0; fi && /docker-entrypoint.sh cassandra -f''
    deploy:
      placement:
        constraints:
          - node.labels.app == cassandra_1
      restart_policy:
        condition: on-failure
        max_attempts: 3
        window: 120s
    environment:
      CASSANDRA_CLUSTER_NAME: "CassandraCluster"
      CASSANDRA_BROADCAST_ADDRESS: tasks.cassandra_1
      CASSANDRA_LISTEN_ADDRESS: tasks.cassandra_1
      #CASSANDRA_SEEDS: tasks.cassandra_1
      CASSANDRA_DC: DC
      CASSANDRA_RACK: RACK1
      CASSANDRA_ENDPOINT_SNITCH: GossipingPropertyFileSnitch
      CASSANDRA_INIT_KEYSPACE_REPLICA_FACTOR: "3"
      #Only for dev
      #MAX_HEAP_SIZE: "4G"
      #HEAP_NEWSIZE: "2G"
    volumes:
      - "/some_dir_from_your_server:/var/lib/cassandra"
    ports:
      - "7000"
      - "9042:9042/tcp"
    networks:
      - cassandra:

  cassandra_2:
    image: cassandra:latest
    command: bash -c 'if [ -z "$$(ls -A /var/lib/cassandra/)" ] ; then sleep 120; fi && /docker-entrypoint.sh cassandra -f'
    deploy:
      placement:
        constraints:
          - node.labels.app == cassandra_2
      restart_policy:
        condition: on-failure
        max_attempts: 3
        window: 120s
    environment:
      CASSANDRA_CLUSTER_NAME: "CassandraCluster"
      CASSANDRA_BROADCAST_ADDRESS: tasks.cassandra_2
      CASSANDRA_LISTEN_ADDRESS: tasks.cassandra_2
      CASSANDRA_SEEDS: tasks.cassandra_1
      CASSANDRA_DC: DC
      CASSANDRA_RACK: RACK2
      CASSANDRA_ENDPOINT_SNITCH: GossipingPropertyFileSnitch
      #Only for dev
      #MAX_HEAP_SIZE: "4G"
      #HEAP_NEWSIZE: "2G"
    volumes:
      - "/some_dir_from_your_server:/var/lib/cassandra"
    ports:
      - "7000"
    networks:
      - cassandra:

  cassandra_3:
    image: cassandra:latest
    command: bash -c 'if [ -z "$$(ls -A /var/lib/cassandra/)" ] ; then sleep 240; fi && /docker-entrypoint.sh cassandra -f'
    deploy:
      placement:
        constraints:
          - node.labels.app == cassandra_3
      restart_policy:
        condition: on-failure
        max_attempts: 3
        window: 120s
    environment:
      CASSANDRA_CLUSTER_NAME: "CassandraCluster"
      CASSANDRA_BROADCAST_ADDRESS: tasks.cassandra_3
      CASSANDRA_LISTEN_ADDRESS: tasks.cassandra_3
      CASSANDRA_SEEDS: tasks.cassandra_1
      CASSANDRA_DC: DC
      CASSANDRA_RACK: RACK3
      CASSANDRA_ENDPOINT_SNITCH: GossipingPropertyFileSnitch
      #Only for dev
      #MAX_HEAP_SIZE: "4G"
      #HEAP_NEWSIZE: "2G"
    volumes:
      - "/some_dir_from_your_server:/var/lib/cassandra"
    ports:
      - "7000"
    networks:
      - cassandra:

networks:
  cassandra:
    driver: overlay
    external:
     name: cassandra-net
```
   

