---
title: 'Mengkonfigurasi Replikasi MariaDB'
date: '2024-11-20T20:57:00+07:00'
categories: ['Database']
tags: ['MariaDB', 'Database', 'Database Replication']
draft: True
---

[mariadb]
log-bin
server_id=1
log-basename=master1
binlog-format=mixed

CREATE USER 'replication_user'@'%' IDENTIFIED BY 'bigs3cret';
GRANT REPLICATION SLAVE ON *.* TO 'replication_user'@'%';

[mysqld]
log-bin
server_id=1

FLUSH TABLES WITH READ LOCK;

SHOW MASTER STATUS;

UNLOCK TABLES;

CHANGE MASTER TO
  MASTER_HOST='master.domain.com',
  MASTER_USER='replication_user',
  MASTER_PASSWORD='bigs3cret',
  MASTER_PORT=3306,
  MASTER_LOG_FILE='master1-bin.000096',
  MASTER_LOG_POS=568,
  MASTER_CONNECT_RETRY=10;

CHANGE MASTER TO MASTER_USE_GTID = slave_pos

START SLAVE;

SHOW SLAVE STATUS \G

Slave_IO_Running: Yes
Slave_SQL_Running: Yes