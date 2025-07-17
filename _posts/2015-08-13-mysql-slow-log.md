---
layout: post
title: "mysql slow log"
description: ""
category: 
tags: [Ops]
---

上午MySQL日志打爆，同事之前已经设置slow_query_log 5 秒，经过排查，是某表未加index，并且 log_queries_not_using_indexes is ON

故将该表加index

使用ALTER TABLE语句创建索引。
语法如下：

    alter table table_name add index index_name (column_list) ;
    alter table table_name add unique (column_list) ;
    alter table table_name add primary key (column_list) ;

另一解决办法是将 log_queries_not_using_indexes set to OFF

由于不想改各种表结构，两个都做了。。。

参考<https://dev.mysql.com/doc/refman/5.6/en/slow-query-log.html>

    slow_query_log = 1
    slow_query_log_file = /var/log/mysql/slow.log
    long_query_time = 10
    log_queries_not_using_indexes = 1

