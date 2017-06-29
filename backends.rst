.. _backends:

=================
Data Persistence
=================

--------------------------
Design of Data Persistence
--------------------------

One to one message Persistence
------------------------------

.. image:: ./_static/images/backend_queue.png

1. PUB publishes a message;

2. Backend records this message in DB;

3. SUB subscribes to a topic;

4. Backend retrieves the messages of this topic from DB;

5. Messages are sent to SUB;

6. After the SUB acknowledged / received the message, backend removes the message from DB.

One to many message Persistence 
-------------------------------

.. image:: ./_static/images/backend_pubsub.png

1. PUB publishes a message;

2. Backend records the message in DB;

3. SUB1 and SUB2 subscribe to a topic;

4. Backend retrieves the messages of this topic;

5. Messages are sent to SUB1 and SUB2; 

6. Backend records the read position of SUB1 and SUB2, next message retrieval starts from this position.

Retainment of Client Connection State
-------------------------------------

EMQ X supports retaining the client's connection state in Redis or DB.

Client Subscription by Broker
-----------------------------

EMQ X Persistence supports Subscription by broker. When a client goes online, the persistence module loads the subscriptions of the client from Redis or Databases.

List of Persistence Plugins
----------------------------

EMQ X supports storing messages in Redis, MySQL, PostgreSQL, MongoDB and Cassandra:

+-----------------------+--------------------------+-------------------------------+
| Persistence Plugins   | Config File              | Description                   |
+=======================+==========================+===============================+
| emqx_backend_redis    | emqx_backend_redis.conf  | Redis Message Persistence     |
+-----------------------+--------------------------+-------------------------------+
| emqx_backend_mysql    | emqx_backend_mysql.conf  | MySQL Message Persistence     |
+-----------------------+--------------------------+-------------------------------+
| emqx_backend_pgsql    | emqx_backend_pgsql.conf  | PostgreSQL Message Persistence|
+-----------------------+--------------------------+-------------------------------+
| emqx_backend_mongo    | emqx_backend_mongo.conf  | MongoDB Message Persistence   |
+-----------------------+--------------------------+-------------------------------+
| emqx_backend_cassa    | emqx_backend_cassa.conf  | Cassandra Message Persistence |
+-----------------------+--------------------------+-------------------------------+

.. _redis_backend:

----------------------------
Data Persistence using Redis
----------------------------

Config file: emqx_backend_redis.conf

Config the Redis Server
-----------------------

Config Connection Pool of Multiple Redis Servers:

.. code-block:: properties

    ## Redis Server
    backend.redis.pool1.server = 127.0.0.1:6379

    ## Redis Pool Size 
    backend.redis.pool1.pool_size = 8

    ## Redis database 
    backend.redis.pool1.database = 1

    ## Redis subscribe channel
    backend.redis.pool1.channel = mqtt_channel

Config Persistence Hooks
------------------------

.. code-block:: properties
    
    ## Expired after seconds, if =< 0 take the default value
    backend.redis.msg.expired_after = 3600
    
    ## Client Connected Record 
    backend.redis.hook.client.connected.1    = {"action": {"function": "on_client_connected"}, "pool": "pool1"}

    ## Subscribe Lookup Record 
    backend.redis.hook.client.connected.2    = {"action": {"function": "on_subscribe_lookup"}, "pool": "pool1"}

    ## Client DisConnected Record 
    backend.redis.hook.client.disconnected.1 = {"action": {"function": "on_client_disconnected"}, "pool": "pool1"}

    ## Lookup Unread Message for one QOS > 0
    backend.redis.hook.session.subscribed.1  = {"topic": "queue/#", "action": {"function": "on_message_fetch_for_queue"}, "pool": "pool1"}
    
    ## Lookup Unread Message for many QOS > 0
    backend.redis.hook.session.subscribed.2  = {"topic": "pubsub/#", "action": {"function": "on_message_fetch_for_pubsub"}, "pool": "pool1"}

    ## Lookup Retain Message 
    backend.redis.hook.session.subscribed.3  = {"action": {"function": "on_retain_lookup"}, "pool": "pool1"}

    ## Store Publish Message  QOS > 0
    backend.redis.hook.message.publish.1     = {"topic": "#", "action": {"function": "on_message_publish"}, "pool": "pool1"}

    ## Store Retain Message 
    backend.redis.hook.message.publish.2     = {"topic": "#", "action": {"function": "on_message_retain"}, "pool": "pool1"}

    ## Delete Retain Message 
    backend.redis.hook.message.publish.3     = {"topic": "#", "action": {"function": "on_retain_delete"}, "pool": "pool1"}

    ## Store Ack for one
    backend.redis.hook.message.acked.1       = {"topic": "queue/#", "action": {"function": "on_message_acked_for_queue"}, "pool": "pool1"}
    
    ## Store Ack for many
    backend.redis.hook.message.acked.2       = {"topic": "pubsub/#", "action": {"function": "on_message_acked_for_pubsub"}, "pool": "pool1"}

Description of Persistence Hooks
--------------------------------

+------------------------+------------------------+-----------------------------+-------------------------------------+
| hook                   | topic                  | action/function             | Description                         |
+========================+========================+=============================+=====================================+
| client.connected       |                        | on_client_connected         | Store client connected state        |
+------------------------+------------------------+-----------------------------+-------------------------------------+
| client.connected       |                        | on_subscribe_lookup         | Subscribe to topics                 |
+------------------------+------------------------+-----------------------------+-------------------------------------+
| client.disconnected    |                        | on_client_disconnected      | Store the client disconnected state |
+------------------------+------------------------+-----------------------------+-------------------------------------+
| session.subscribed     | queue/#                | on_message_fetch_for_queue  | Fetch one to one offline message    |
+------------------------+------------------------+-----------------------------+-------------------------------------+
| session.subscribed     | pubsub/#               | on_message_fetch_for_pubsub | Fetch one to many offline message   |
+------------------------+------------------------+-----------------------------+-------------------------------------+
| session.subscribed     | #                      | on_retain_lookup            | Lookup retained message             |
+------------------------+------------------------+-----------------------------+-------------------------------------+
| message.publish        | #                      | on_message_publish          | Store the published messages        |
+------------------------+------------------------+-----------------------------+-------------------------------------+
| message.publish        | #                      | on_message_retain           | Store retained messages             |
+------------------------+------------------------+-----------------------------+-------------------------------------+
| message.publish        | #                      | on_retain_delete            | Delete retained messages            |
+------------------------+------------------------+-----------------------------+-------------------------------------+
| message.acked          | queue/#                | on_message_acked_for_queue  | Process ACK of one to one messages  |
+------------------------+------------------------+-----------------------------+-------------------------------------+
| message.acked          | pubsub/#               | on_message_acked_for_pubsub | Process ACK of one to many messages |
+------------------------+------------------------+-----------------------------+-------------------------------------+

Redis Command Line Arguments
----------------------------

+----------------------+-----------------------------------------------+-------------------------------------------------+
| hook                 | Arguments                                     | Example (Fields separated exactly by one space) |
+======================+===============================================+=================================================+
| client.connected     | clientid                                      | SET conn:${clientid} clientid                   |
+----------------------+-----------------------------------------------+-------------------------------------------------+
| client.disconnected  | clientid                                      | SET disconn:${clientid} clientid                |
+----------------------+-----------------------------------------------+-------------------------------------------------+
| session.subscribed   | clientid, topic, qos                          | HSET sub:${clientid} topic qos                  |
+----------------------+-----------------------------------------------+-------------------------------------------------+
| session.unsubscribed | clientid, topic                               | SET unsub:${clientid} topic                     |
+----------------------+-----------------------------------------------+-------------------------------------------------+
| message.publish      | message, msgid, topic, payload, qos, clientid | RPUSH pub:${topic} msgid                        |
+----------------------+-----------------------------------------------+-------------------------------------------------+
| message.acked        | msgid, topic, clientid                        | HSET ack:${clientid} topic msgid                |
+----------------------+-----------------------------------------------+-------------------------------------------------+
| message.delivered    | msgid, topic, clientid                        | HSET delivered:${clientid} topic msgid          |
+----------------------+-----------------------------------------------+-------------------------------------------------+

Config 'action' utilizing Redis Command Line
---------------------------------------------

Redis backend supports using 'commands' in 'action', e.g.:

.. code-block:: properties
    
    ## After a client connected to the EMQ X server, it executes a redis command (multiple redis commands also supported)
    backend.redis.hook.client.connected.3 = {"action": {"commands": ["SET conn:${clientid} clientid"]}, "pool": "pool1"}

Using Redis Hash for Devices' Connection State
----------------------------------------------

*mqtt:client* Hash for devices' connection state::

    hmset
    key = mqtt:client:${clientid} 
    value = {state:int, online_at:timestamp, offline_at:timestamp}

    hset
    key = mqtt:node:${node}
    field = ${clientid}
    value = ${ts}

Lookup devices' connection state::

    HGETALL "mqtt:client:${clientId}"
    
E.g.: Client with ClientId 'test' goes online::
    
    HGETALL mqtt:client:test
    1) "state"
    2) "1"
    3) "online_at"
    4) "1481685802"
    5) "offline_at"
    6) "undefined"
    
Client with ClientId 'test' goes offline::
    
    HGETALL mqtt:client:test
    1) "state"
    2) "0"
    3) "online_at"
    4) "1481685802"
    5) "offline_at"
    6) "1481685924"

Using Redis Hash for Retained Messages
--------------------------------------

*mqtt:retain* Hash for retained messages::

    hmset
    key = mqtt:retain:${topic}
    value = {id: string, from: string, qos: int, topic: string, retain: int, payload: string, ts: timestamp}

Lookup retained message::

    HGETALL "mqtt:retain:${topic}"

Lookup retained messages with a topic of 'retain'::
    
    HGETALL mqtt:retain:topic
     1) "id"
     2) "6P9NLcJ65VXBbC22sYb4"
     3) "from"
     4) "test"
     5) "qos"
     6) "1"
     7) "topic"
     8) "topic"
     9) "retain"
    10) "true"
    11) "payload"
    12) "Hello world!"
    13) "ts"
    14) "1481690659"

Using Redis Hash for messages
-----------------------------

*mqtt:msg* Hash for MQTT messages::

    hmset
    key = mqtt:msg:${msgid}
    value = {id: string, from: string, qos: int, topic: string, retain: int, payload: string, ts: timestamp}

    zadd
    key = mqtt:msg:${topic}
    field = 1
    value = ${msgid}

Using Redis Set for Message Acknowledgements
--------------------------------------------

*mqtt:acked* SET stores acknowledgements from the clients::

    set
    key = mqtt:acked:${clientid}:${topic}
    value = ${msgid}

Using Redis Hash for Subscription
----------------------------------

*mqtt:sub* Hash for Subscriptions::

    hset
    key = mqtt:sub:${clientid}
    field = ${topic}
    value = ${qos}

A client subscribes to a topic::
    
    HSET mqtt:sub:${clientid} ${topic} ${qos}
    
A client with ClientId of 'test' subscribes to topic1 and topic2::

    HSET "mqtt:sub:test" "topic1" 1
    HSET "mqtt:sub:test" "topic2" 2
    
Lookup the subscribed topics of client with ClientId of 'test::
 
    HGETALL mqtt:sub:test
    1) "topic1"
    2) "1"
    3) "topic2"
    4) "2"
 
Redis SUB/UNSUB Publish
-----------------------

When a device subscribes / unsubscribes topics, EMQ X publish an event to the Redis::

    PUBLISH
    channel = "mqtt_channel"
    message = {type: string , topic: string, clientid: string, qos: int} 
    \*type: [subscribe/unsubscribe]

client with ClientID 'test' subscribe to 'topic0'::

    PUBLISH "mqtt_channel" "{\"type\": \"subscribe\", \"topic\": \"topic0\", \"clientid\": \"test\", \"qos\": \"0\"}"

Client with ClientId 'test' unsubscribes to 'test_topic0'::

    PUBLISH "mqtt_channel" "{\"type\": \"unsubscribe\", \"topic\": \"test_topic0\", \"clientid\": \"test\"}"

Enable Redis Backend
--------------------

.. code-block:: bash

    ./bin/emqx_ctl plugins load emqx_backend_redis

.. _mysql_backend:

----------------------------
Data Persistence Using MySQL
----------------------------

Config file: emqx_backend_mysql.conf

Config MySQL Server
--------------------

Connection pool of multiple MySQL servers is supported::

.. code-block:: properties

    ## Mysql Server
    backend.mysql.pool1.server = 127.0.0.1:3306

    ## Mysql Pool Size
    backend.mysql.pool1.pool_size = 8

    ## Mysql Username
    backend.mysql.pool1.user = root

    ## Mysql Password
    backend.mysql.pool1.password = public

    ## Mysql Database
    backend.mysql.pool1.database = mqtt

Config MySQL Persistence Hooks
------------------------------

.. code-block:: properties

    ## Client Connected Record 
    backend.mysql.hook.client.connected.1    = {"action": {"function": "on_client_connected"}, "pool": "pool1"}

    ## Subscribe Lookup Record 
    backend.mysql.hook.client.connected.2    = {"action": {"function": "on_subscribe_lookup"}, "pool": "pool1"}
    
    ## Client DisConnected Record 
    backend.mysql.hook.client.disconnected.1 = {"action": {"function": "on_client_disconnected"}, "pool": "pool1"}

    ## Lookup Unread Message QOS > 0
    backend.mysql.hook.session.subscribed.1  = {"topic": "#", "action": {"function": "on_message_fetch"}, "pool": "pool1"}

    ## Lookup Retain Message 
    backend.mysql.hook.session.subscribed.2  = {"topic": "#", "action": {"function": "on_retain_lookup"}, "pool": "pool1"}

    ## Store Publish Message  QOS > 0
    backend.mysql.hook.message.publish.1     = {"topic": "#", "action": {"function": "on_message_publish"}, "pool": "pool1"}

    ## Store Retain Message 
    backend.mysql.hook.message.publish.2     = {"topic": "#", "action": {"function": "on_message_retain"}, "pool": "pool1"}

    ## Delete Retain Message 
    backend.mysql.hook.message.publish.3     = {"topic": "#", "action": {"function": "on_retain_delete"}, "pool": "pool1"}

    ## Store Ack
    backend.mysql.hook.message.acked.1       = {"topic": "#", "action": {"function": "on_message_acked"}, "pool": "pool1"}

Description of MySQL Persistence Hooks
--------------------------------------

+------------------------+------------------------+-------------------------+----------------------------------+
| hook                   | topic                  | action                  | Description                      |
+========================+========================+=========================+==================================+
| client.connected       |                        | on_client_connected     | Store client connected state     |
+------------------------+------------------------+-------------------------+----------------------------------+
| client.connected       |                        | on_subscribe_lookup     | Subscribed topics                |
+------------------------+------------------------+-------------------------+----------------------------------+
| client.disconnected    |                        | on_client_disconnected  | Store client disconnected state  |
+------------------------+------------------------+-------------------------+----------------------------------+
| session.subscribed     | #                      | on_message_fetch        | Fetch offline messages           |
+------------------------+------------------------+-------------------------+----------------------------------+
| session.subscribed     | #                      | on_retain_lookup        | Lookup retained messages         |
+------------------------+------------------------+-------------------------+----------------------------------+
| message.publish        | #                      | on_message_publish      | Store published messages         |
+------------------------+------------------------+-------------------------+----------------------------------+
| message.publish        | #                      | on_message_retain       | Store retained messages          |
+------------------------+------------------------+-------------------------+----------------------------------+
| message.publish        | #                      | on_retain_delete        | Delete retained messages         |
+------------------------+------------------------+-------------------------+----------------------------------+
| message.acked          | #                      | on_message_acked        | Process ACK                      |
+------------------------+------------------------+-------------------------+----------------------------------+

SQL Arguments Description 
--------------------------

+----------------------+---------------------------------------+----------------------------------------------------------------+
| hook                 | Arguments                             | Example (${name} represents available argument)                |
+======================+=======================================+================================================================+
| client.connected     | clientid                              | insert into conn(clientid) values(${clientid})                 |
+----------------------+---------------------------------------+----------------------------------------------------------------+
| client.disconnected  | clientid                              | insert into disconn(clientid) values(${clientid})              |
+----------------------+---------------------------------------+----------------------------------------------------------------+
| session.subscribed   | clientid, topic, qos                  | insert into sub(topic, qos) values(${topic}, ${qos})           |
+----------------------+---------------------------------------+----------------------------------------------------------------+
| session.unsubscribed | clientid, topic                       | delete from sub where topic = ${topic}                         |
+----------------------+---------------------------------------+----------------------------------------------------------------+
| message.publish      | msgid, topic, payload, qos, clientid  | insert into msg(msgid, topic) values(${msgid}, ${topic})       |
+----------------------+---------------------------------------+----------------------------------------------------------------+
| message.acked        | msgid, topic, clientid                | insert into ack(msgid, topic) values(${msgid}, ${topic})       |
+----------------------+---------------------------------------+----------------------------------------------------------------+
| message.delivered    | msgid, topic, clientid                | insert into delivered(msgid, topic) values(${msgid}, ${topic}) |
+----------------------+---------------------------------------+----------------------------------------------------------------+

Config 'action' utilizing SQL
-----------------------------

MySQL backend supports using SQL in 'action':

.. code-block:: properties

    ## After a client is connected to the EMQ X server, it executes a SQL command (multiple SQL commands also supported)
    backend.mysql.hook.client.connected.3 = {"action": {"sql": ["insert into conn(clientid) values(${clientid})"]}, "pool": "pool1"}

Create MySQL DB
---------------

.. code-block:: sql

    create database mqtt;

Import MySQL DB & Table Schema
------------------------------
    
.. code-block:: bash
    
    mysql -u root -p mqtt < etc/sql/emqx_backend_mysql.sql

.. NOTE:: DB name is free of choice

MySQL Client Connection Table
-----------------------------

*mqtt_client* stores client connection states:

.. code-block:: sql

    DROP TABLE IF EXISTS `mqtt_client`;
    CREATE TABLE `mqtt_client` (
      `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
      `clientid` varchar(64) DEFAULT NULL,
      `state` varchar(3) DEFAULT NULL,
      `node` varchar(100) DEFAULT NULL,
      `online_at` datetime DEFAULT NULL,
      `offline_at` datetime DEFAULT NULL,
      `created` timestamp NULL DEFAULT CURRENT_TIMESTAMP,
      PRIMARY KEY (`id`),
      KEY `mqtt_client_idx` (`clientid`),
      UNIQUE KEY `mqtt_client_key` (`clientid`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

Inquire the client connection state:

.. code-block:: sql

    select * from mqtt_client where clientid = ${clientid};
    
If client 'test' is online:

.. code-block:: sql

    select * from mqtt_client where clientid = "test";
    
    +----+----------+-------+----------------+---------------------+---------------------+---------------------+
    | id | clientid | state | node           | online_at           | offline_at          | created             |
    +----+----------+-------+----------------+---------------------+---------------------+---------------------+
    |  1 | test     | 1     | emqx@127.0.0.1 | 2016-11-15 09:40:40 | NULL                | 2016-12-24 09:40:22 |
    +----+----------+-------+----------------+---------------------+---------------------+---------------------+
    1 rows in set (0.00 sec)

If client 'test' is offline:

.. code-block:: sql

    select * from mqtt_client where clientid = "test";
    
    +----+----------+-------+----------------+---------------------+---------------------+---------------------+
    | id | clientid | state | node           | online_at           | offline_at          | created             |
    +----+----------+-------+----------------+---------------------+---------------------+---------------------+
    |  1 | test     | 0     | emqx@127.0.0.1 | 2016-11-15 09:40:40 | 2016-11-15 09:46:10 | 2016-12-24 09:40:22 |
    +----+----------+-------+----------------+---------------------+---------------------+---------------------+
    1 rows in set (0.00 sec)

MySQL Subscription TABLE
------------------------

*mqtt_sub* stores subscriptions of clients:

.. code-block:: sql

    DROP TABLE IF EXISTS `mqtt_sub`;
    CREATE TABLE `mqtt_sub` (
      `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
      `clientid` varchar(64) DEFAULT NULL,
      `topic` varchar(256) DEFAULT NULL,
      `qos` int(3) DEFAULT NULL,
      `created` timestamp NULL DEFAULT CURRENT_TIMESTAMP,
      PRIMARY KEY (`id`),
      KEY `mqtt_sub_idx` (`clientid`,`topic`(255),`qos`),
      UNIQUE KEY `mqtt_sub_key` (`clientid`,`topic`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

E.g., client 'test' subscribes to 'test_topic1' and 'test_topic2':

.. code-block:: sql

    insert into mqtt_sub(clientid, topic, qos) values("test", "test_topic1", 1);
    insert into mqtt_sub(clientid, topic, qos) values("test", "test_topic2", 2);

Inquire subscription of a client:

.. code-block:: sql
    
    select * from mqtt_sub where clientid = ${clientid};

E.g., inquiring the Subscription of client 'test':

.. code-block:: sql
    
    select * from mqtt_sub where clientid = "test";
    
    +----+--------------+-------------+------+---------------------+
    | id | clientId     | topic       | qos  | created             |
    +----+--------------+-------------+------+---------------------+
    |  1 | test         | test_topic1 |    1 | 2016-12-24 17:09:05 |
    |  2 | test         | test_topic2 |    2 | 2016-12-24 17:12:51 |
    +----+--------------+-------------+------+---------------------+
    2 rows in set (0.00 sec)

MySQL Message Table
-------------------

*mqtt_msg* stores MQTT messages:

.. code-block:: sql
    
    DROP TABLE IF EXISTS `mqtt_msg`;
    CREATE TABLE `mqtt_msg` (
      `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
      `msgid` varchar(100) DEFAULT NULL,
      `topic` varchar(1024) NOT NULL,
      `sender` varchar(1024) DEFAULT NULL,
      `node` varchar(60) DEFAULT NULL,
      `qos` int(11) NOT NULL DEFAULT '0',
      `retain` tinyint(2) DEFAULT NULL,
      `payload` blob,
      `arrived` datetime NOT NULL,
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

Inquiring messages published by a client:

.. code-block:: sql

    select * from mqtt_msg where sender = ${clientid};

Inquiring messages published by client 'test':

.. code-block:: sql

    select * from mqtt_msg where sender = "test";
    
    +----+-------------------------------+----------+--------+------+-----+--------+---------+---------------------+
    | id | msgid                         | topic    | sender | node | qos | retain | payload | arrived             |
    +----+-------------------------------+----------+--------+------+-----+--------+---------+---------------------+
    | 1  | 53F98F80F66017005000004A60003 | hello    | test   | NULL |   1 |      0 | hello   | 2016-12-24 17:25:12 |
    | 2  | 53F98F9FE42AD7005000004A60004 | world    | test   | NULL |   1 |      0 | world   | 2016-12-24 17:25:45 |
    +----+-------------------------------+----------+--------+------+-----+--------+---------+---------------------+
    2 rows in set (0.00 sec)

MySQL Retained Message Table
----------------------------

mqtt_retain stores retained messages:

.. code-block:: sql
    
    DROP TABLE IF EXISTS `mqtt_retain`;
    CREATE TABLE `mqtt_retain` (
      `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
      `topic` varchar(200) DEFAULT NULL,
      `msgid` varchar(60) DEFAULT NULL,
      `sender` varchar(100) DEFAULT NULL,
      `node` varchar(100) DEFAULT NULL,
      `qos` int(2) DEFAULT NULL,
      `payload` blob,
      `arrived` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
      PRIMARY KEY (`id`),
      UNIQUE KEY `mqtt_retain_key` (`topic`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

Inquiring retained messages:

.. code-block:: sql

    select * from mqtt_retain where topic = ${topic};

Inquiring retained messages with topic 'retain':

.. code-block:: sql

    select * from mqtt_retain where topic = "retain";
    
    +----+----------+-------------------------------+---------+------+------+---------+---------------------+
    | id | topic    | msgid                         | sender  | node | qos  | payload | arrived             |
    +----+----------+-------------------------------+---------+------+------+---------+---------------------+
    |  1 | retain   | 53F33F7E4741E7007000004B70001 | test    | NULL |    1 | www     | 2016-12-24 16:55:18 |
    +----+----------+-------------------------------+---------+------+------+---------+---------------------+
    1 rows in set (0.00 sec)

MySQL Acknowledgement Table
----------------------------

*mqtt_acked* stores acknowledgements from the clients:

.. code-block:: sql
    
    DROP TABLE IF EXISTS `mqtt_acked`;
    CREATE TABLE `mqtt_acked` (
      `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
      `clientid` varchar(200) DEFAULT NULL,
      `topic` varchar(200) DEFAULT NULL,
      `mid` int(200) DEFAULT NULL,
      `created` timestamp NULL DEFAULT NULL,
      PRIMARY KEY (`id`),
      UNIQUE KEY `mqtt_acked_key` (`clientid`,`topic`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

Enable MySQL Backend
--------------------

.. code-block:: bash

    ./bin/emqx_ctl plugins load emqx_backend_mysql

.. _postgre_backend:

---------------------------------
Data Persistence using PostgreSQL
---------------------------------

Config file: emqx_backend_pgsql.conf

Config PostgreSQL Server
------------------------

Connection pool of multiple PostgreSQL servers is supported:

.. code-block:: properties

    ## Pgsql Server
    backend.pgsql.pool1.server = 127.0.0.1:5432

    ## Pgsql Pool Size
    backend.pgsql.pool1.pool_size = 8

    ## Pgsql Username
    backend.pgsql.pool1.username = root

    ## Pgsql Password
    backend.pgsql.pool1.password = public

    ## Pgsql Database
    backend.pgsql.pool1.database = mqtt

    ## Pgsql Ssl
    backend.pgsql.pool1.ssl = false  

Config PostgreSQL Persistence Hooks
-----------------------------------

.. code-block:: properties

    ## Client Connected Record 
    backend.pgsql.hook.client.connected.1    = {"action": {"function": "on_client_connected"}, "pool": "pool1"}

    ## Subscribe Lookup Record 
    backend.pgsql.hook.client.connected.2    = {"action": {"function": "on_subscribe_lookup"}, "pool": "pool1"}

    ## Client DisConnected Record 
    backend.pgsql.hook.client.disconnected.1 = {"action": {"function": "on_client_disconnected"}, "pool": "pool1"}

    ## Lookup Unread Message QOS > 0
    backend.pgsql.hook.session.subscribed.1  = {"topic": "#", "action": {"function": "on_message_fetch"}, "pool": "pool1"}

    ## Lookup Retain Message 
    backend.pgsql.hook.session.subscribed.2  = {"topic": "#", "action": {"function": "on_retain_lookup"}, "pool": "pool1"}

    ## Store Publish Message  QOS > 0
    backend.pgsql.hook.message.publish.1     = {"topic": "#", "action": {"function": "on_message_publish"}, "pool": "pool1"}

    ## Store Retain Message 
    backend.pgsql.hook.message.publish.2     = {"topic": "#", "action": {"function": "on_message_retain"}, "pool": "pool1"}

    ## Delete Retain Message 
    backend.pgsql.hook.message.publish.3     = {"topic": "#", "action": {"function": "on_retain_delete"}, "pool": "pool1"}

    ## Store Ack
    backend.pgsql.hook.message.acked.1       = {"topic": "#", "action": {"function": "on_message_acked"}, "pool": "pool1"}

Description of PostgreSQL Persistence Hooks
-------------------------------------------

+------------------------+------------------------+-------------------------+----------------------------------+
| hook                   | topic                  | action                  | Description                      |
+========================+========================+=========================+==================================+
| client.connected       |                        | on_client_connected     | Store client connected state     |
+------------------------+------------------------+-------------------------+----------------------------------+
| client.connected       |                        | on_subscribe_lookup     | Subscribed topics                |
+------------------------+------------------------+-------------------------+----------------------------------+
| client.disconnected    |                        | on_client_disconnected  | Store client disconnected state  |
+------------------------+------------------------+-------------------------+----------------------------------+
| session.subscribed     | #                      | on_message_fetch        | Fetch offline messages           |
+------------------------+------------------------+-------------------------+----------------------------------+
| session.subscribed     | #                      | on_retain_lookup        | Lookup retained messages         |
+------------------------+------------------------+-------------------------+----------------------------------+
| message.publish        | #                      | on_message_publish      | Store published messages         |
+------------------------+------------------------+-------------------------+----------------------------------+
| message.publish        | #                      | on_message_retain       | Store retained messages          |
+------------------------+------------------------+-------------------------+----------------------------------+
| message.publish        | #                      | on_retain_delete        | Delete retained messages         |
+------------------------+------------------------+-------------------------+----------------------------------+
| message.acked          | #                      | on_message_acked        | Process ACK                      |
+------------------------+------------------------+-------------------------+----------------------------------+

SQL Arguments Description
-------------------------

+----------------------+---------------------------------------+----------------------------------------------------------------+
| hook                 | Arguments                             | Example (${name} represents available argument)                |
+======================+=======================================+================================================================+
| client.connected     | clientid                              | insert into conn(clientid) values(${clientid})                 |
+----------------------+---------------------------------------+----------------------------------------------------------------+
| client.disconnected  | clientid                              | insert into disconn(clientid) values(${clientid})              |
+----------------------+---------------------------------------+----------------------------------------------------------------+
| session.subscribed   | clientid, topic, qos                  | insert into sub(topic, qos) values(${topic}, ${qos})           |
+----------------------+---------------------------------------+----------------------------------------------------------------+
| session.unsubscribed | clientid, topic                       | delete from sub where topic = ${topic}                         |
+----------------------+---------------------------------------+----------------------------------------------------------------+
| message.publish      | msgid, topic, payload, qos, clientid  | insert into msg(msgid, topic) values(${msgid}, ${topic})       |
+----------------------+---------------------------------------+----------------------------------------------------------------+
| message.acked        | msgid, topic, clientid                | insert into ack(msgid, topic) values(${msgid}, ${topic})       |
+----------------------+---------------------------------------+----------------------------------------------------------------+
| message.delivered    | msgid, topic, clientid                | insert into delivered(msgid, topic) values(${msgid}, ${topic}) |
+----------------------+---------------------------------------+----------------------------------------------------------------+

Config 'action' utilizing SQL
-----------------------------

PostgreSQL backend supports using SQL in 'action':

.. code-block:: properties

    ## After a client is connected to the EMQ X server, it executes a SQL command (multiple command also supported)
    backend.pgsql.hook.client.connected.3 = {"action": {"sql": ["insert into conn(clientid) values(${clientid})"]}, "pool": "pool1"}

Create PostgreSQL DB
--------------------
    
.. code-block:: bash

    createdb mqtt -E UTF8 -e

Import PostgreSQL DB & Table Schema
-----------------------------------
    
.. code-block:: bash
    
    \i etc/sql/emqx_backend_pgsql.sql

.. NOTE:: DB name is free of choice 

PostgreSQL Client Connection Table
-----------------------------------

*mqtt_client* stores client connection states::

    CREATE TABLE mqtt_client(
      id SERIAL primary key,
      clientid character varying(100),
      state integer,
      node character varying(100),
      online_at integer,
      offline_at integer,
      created timestamp without time zone,
      UNIQUE (clientid)
    );

Inquiring a client's connection state::

    select * from mqtt_client where clientid = ${clientid};

E.g., if client 'test' is online::

    select * from mqtt_client where clientid = 'test';

     id | clientid | state | node             | online_at           | offline_at        | created
    ----+----------+-------+----------------+---------------------+---------------------+---------------------
      1 | test     | 1     | emqx@127.0.0.1 | 2016-11-15 09:40:40 | NULL                | 2016-12-24 09:40:22
    (1 rows)

Client 'test" is offline::

    select * from mqtt_client where clientid = 'test';

     id | clientid | state | nod            | online_at           | offline_at          | created
    ----+----------+-------+----------------+---------------------+---------------------+---------------------
      1 | test     | 0     | emqx@127.0.0.1 | 2016-11-15 09:40:40 | 2016-11-15 09:46:10 | 2016-12-24 09:40:22
    (1 rows)

PostgreSQL Subscription Table
-----------------------------
    
*mqtt_sub* stores subscriptions of clients::

    CREATE TABLE mqtt_sub(
      id SERIAL primary key,
      clientid character varying(100),
      topic character varying(200),
      qos integer,
      created timestamp without time zone,
      UNIQUE (clientid, topic)
    );

E.g., client 'test' subscribes to topic 'test_topic1' and 'test_topic2':

.. code-block:: sql

    insert into mqtt_sub(clientid, topic, qos) values('test', 'test_topic1', 1);
    insert into mqtt_sub(clientid, topic, qos) values('test', 'test_topic2', 2);

Inquiring subscription of a client::
    
    select * from mqtt_sub where clientid = ${clientid};

Inquiring subscription of client 'test'::
    
    select * from mqtt_sub where clientid = 'test';

     id | clientId     | topic       | qos  | created             
    ----+--------------+-------------+------+---------------------
      1 | test         | test_topic1 |    1 | 2016-12-24 17:09:05 
      2 | test         | test_topic2 |    2 | 2016-12-24 17:12:51
    (2 rows) 

PostgreSQL Message Table
------------------------

*mqtt_msg* stores MQTT messages:

.. code-block:: sql

    CREATE TABLE mqtt_msg (
      id SERIAL primary key,
      msgid character varying(60),
      sender character varying(100),
      topic character varying(200),
      qos integer,
      retain integer,
      payload text,
      arrived timestamp without time zone
    );

Inquiring messages published by a client::
    
    select * from mqtt_msg where sender = ${clientid};

Inquiring messages published by client 'test'::

    select * from mqtt_msg where sender = 'test';

     id | msgid                         | topic    | sender | node | qos | retain | payload | arrived             
    ----+-------------------------------+----------+--------+------+-----+--------+---------+---------------------
     1  | 53F98F80F66017005000004A60003 | hello    | test   | NULL |   1 |      0 | hello   | 2016-12-24 17:25:12 
     2  | 53F98F9FE42AD7005000004A60004 | world    | test   | NULL |   1 |      0 | world   | 2016-12-24 17:25:45 
    (2 rows)

PostgreSQL Retained Message Table
---------------------------------

*mqtt_retain* stores retained messages:

.. code-block:: sql

    CREATE TABLE mqtt_retain(
      id SERIAL primary key,
      topic character varying(200),
      msgid character varying(60),
      sender character varying(100),
      qos integer,
      payload text,
      arrived timestamp without time zone,
      UNIQUE (topic)
    );

Inquiring retained messages::

    select * from mqtt_retain where topic = ${topic};

Inquiring retained messages with topic 'retain'::

    select * from mqtt_retain where topic = 'retain';

     id | topic    | msgid                         | sender  | node | qos  | payload | arrived             
    ----+----------+-------------------------------+---------+------+------+---------+---------------------
      1 | retain   | 53F33F7E4741E7007000004B70001 | test    | NULL |    1 | www     | 2016-12-24 16:55:18 
    (1 rows)
 
PostgreSQL Acknowledgement Table
--------------------------------

*mqtt_acked* stores acknowledgements from the clients:

.. code-block:: sql
    
    CREATE TABLE mqtt_acked (
      id SERIAL primary key,
      clientid character varying(100),
      topic character varying(100),
      mid integer,
      created timestamp without time zone,
      UNIQUE (clientid, topic)
    );

Enable PostgreSQL Backend
-------------------------

.. code-block:: bash

    ./bin/emqx_ctl plugins load emqx_backend_pgsql

.. _mongodb_backend:

------------------------------
Data Persistence using MongoDB
------------------------------

Config file: emqx_backend_mongo.conf

Config MongoDB Server
---------------------

Connection pool of multiple PostgreSQL servers is supported:

.. code-block:: properties

    ## MongoDB Server
    backend.mongo.pool1.server = 127.0.0.1:27017

    ## MongoDB Pool Size
    backend.mongo.pool1.pool_size = 8

    ## MongoDB Database
    backend.mongo.pool1.database = mqtt

Config MongoDB Persistence Hooks
--------------------------------

.. code-block:: properties

    ## Client Connected Record 
    backend.mongo.hook.client.connected.1    = {"action": {"function": "on_client_connected"}, "pool": "pool1"}

    ## Subscribe Lookup Record 
    backend.mongo.hook.client.connected.2    = {"action": {"function": "on_subscribe_lookup"}, "pool": "pool1"}
    
    ## Client DisConnected Record 
    backend.mongo.hook.client.disconnected.1 = {"action": {"function": "on_client_disconnected"}, "pool": "pool1"}

    ## Lookup Unread Message QOS > 0
    backend.mongo.hook.session.subscribed.1  = {"topic": "#", "action": {"function": "on_message_fetch"}, "pool": "pool1"}

    ## Lookup Retain Message 
    backend.mongo.hook.session.subscribed.2  = {"topic": "#", "action": {"function": "on_retain_lookup"}, "pool": "pool1"}

    ## Store Publish Message  QOS > 0
    backend.mongo.hook.message.publish.1     = {"topic": "#", "action": {"function": "on_message_publish"}, "pool": "pool1"}

    ## Store Retain Message 
    backend.mongo.hook.message.publish.2     = {"topic": "#", "action": {"function": "on_message_retain"}, "pool": "pool1"}

    ## Delete Retain Message 
    backend.mongo.hook.message.publish.3     = {"topic": "#", "action": {"function": "on_retain_delete"}, "pool": "pool1"}

    ## Store Ack
    backend.mongo.hook.message.acked.1       = {"topic": "#", "action": {"function": "on_message_acked"}, "pool": "pool1"}

Description of MongoDB Persistence Hooks
----------------------------------------

+------------------------+------------------------+-------------------------+----------------------------------+
| hook                   | topic                  | action                  | Description                      |
+========================+========================+=========================+==================================+
| client.connected       |                        | on_client_connected     | Store client connected state     |
+------------------------+------------------------+-------------------------+----------------------------------+
| client.connected       |                        | on_subscribe_lookup     | Subscribed topics                |
+------------------------+------------------------+-------------------------+----------------------------------+
| client.disconnected    |                        | on_client_disconnected  | Store client disconnected state  |
+------------------------+------------------------+-------------------------+----------------------------------+
| session.subscribed     | #                      | on_message_fetch        | Fetch offline messages           |
+------------------------+------------------------+-------------------------+----------------------------------+
| session.subscribed     | #                      | on_retain_lookup        | Lookup retained messages         |
+------------------------+------------------------+-------------------------+----------------------------------+
| message.publish        | #                      | on_message_publish      | Store published messages         |
+------------------------+------------------------+-------------------------+----------------------------------+
| message.publish        | #                      | on_message_retain       | Store retained messages          |
+------------------------+------------------------+-------------------------+----------------------------------+
| message.publish        | #                      | on_retain_delete        | Delete retained messages         |
+------------------------+------------------------+-------------------------+----------------------------------+
| message.acked          | #                      | on_message_acked        | Process ACK                      |
+------------------------+------------------------+-------------------------+----------------------------------+

Create MongoDB DB & Collections
-------------------------------

.. code-block:: javascript

    use mqtt
    db.createCollection("mqtt_client")
    db.createCollection("mqtt_sub")
    db.createCollection("mqtt_msg")
    db.createCollection("mqtt_retain")
    db.createCollection("mqtt_acked")

    db.mqtt_client.ensureIndex({clientid:1, node:2})
    db.mqtt_sub.ensureIndex({clientid:1})
    db.mqtt_msg.ensureIndex({sender:1, topic:2})
    db.mqtt_retain.ensureIndex({topic:1})

.. NOTE:: DB name is free of choice

MongoDB Client Connection Collection
------------------------------------

*mqtt_client* stores client connection states:

.. code-block:: javascript

    {
        clientid: string,
        state: 0,1, //0 disconnected 1 connected
        node: string,
        online_at: timestamp,
        offline_at: timestamp
    }

Inquiring client's connection state:

.. code-block:: javascript

    db.mqtt_client.findOne({clientid: ${clientid}})

E.g., if client 'test' is online:

.. code-block:: javascript

    db.mqtt_client.findOne({clientid: "test"})
    
    {
        "_id" : ObjectId("58646c9bdde89a9fb9f7fb73"),
        "clientid" : "test",
        "state" : 1,
        "node" : "emqx@127.0.0.1",
        "online_at" : 1482976411,
        "offline_at" : null
    }

Client 'test' is offline:

.. code-block:: javascript

    db.mqtt_client.findOne({clientid: "test"})
    
    {
        "_id" : ObjectId("58646c9bdde89a9fb9f7fb73"),
        "clientid" : "test",
        "state" : 0,
        "node" : "emq@127.0.0.1",
        "online_at" : 1482976411,
        "offline_at" : 1482976501
    }

MongoDB Subscription Collection
-------------------------------

*mqtt_sub* stores subscriptions of clients:

.. code-block:: javascript

    {
        clientid: string,
        topic: string,
        qos: 0,1,2
    }

E.g., client 'test' subscribes to topic 'test_topic1' and 'test_topic2':

.. code-block:: javascript

    db.mqtt_sub.insert({clientid: "test", topic: "test_topic1", qos: 1})
    db.mqtt_sub.insert({clientid: "test", topic: "test_topic2", qos: 2})

Inquiring subscription of client 'test':

.. code-block:: javascript
    
    db.mqtt_sub.find({clientid: "test"})
    
    { "_id" : ObjectId("58646d90c65dff6ac9668ca1"), "clientid" : "test", "topic" : "test_topic1", "qos" : 1 }
    { "_id" : ObjectId("58646d96c65dff6ac9668ca2"), "clientid" : "test", "topic" : "test_topic2", "qos" : 2 }

MongoDB Message Collection
---------------------------

*mqtt_msg* stores MQTT messages:

.. code-block:: javascript

    {
        _id: int,
        topic: string,
        msgid: string, 
        sender: string, 
        qos: 0,1,2, 
        retain: boolean (true, false),
        payload: string,
        arrived: timestamp
    }

Inquiring messages published by a client:

.. code-block:: javascript

    db.mqtt_msg.find({sender: ${clientid}})

Inquiring messages published by client 'test': 

.. code-block:: javascript
    
    db.mqtt_msg.find({sender: "test"})
    { 
        "_id" : 1, 
        "topic" : "/World", 
        "msgid" : "AAVEwm0la4RufgAABeIAAQ==", 
        "sender" : "test", 
        "qos" : 1, 
        "retain" : 1, 
        "payload" : "Hello world!", 
        "arrived" : 1482976729 
    }

MongoDB Retained Message Collection
-----------------------------------

*mqtt_retain* stores retained messages:

.. code-block:: javascript

    {
        topic: string,
        msgid: string, 
        sender: string, 
        qos: 0,1,2, 
        payload: string,
        arrived: timestamp
    }

Inquiring retained messages:

.. code-block:: javascript

    db.mqtt_retain.findOne({topic: ${topic}})

Inquiring retained messages with topic 'retain':

.. code-block:: javascript

    db.mqtt_retain.findOne({topic: "/World"})
    {
        "_id" : ObjectId("58646dd9dde89a9fb9f7fb75"),
        "topic" : "/World",
        "msgid" : "AAVEwm0la4RufgAABeIAAQ==",
        "sender" : "c1",
        "qos" : 1,
        "payload" : "Hello world!",
        "arrived" : 1482976729
    }

MongoDB Acknowledgement Collection
-------------------------------------

*mqtt_acked* stores acknowledgements from the clients:

.. code-block:: javascript

    {
        clientid: string, 
        topic: string, 
        mongo_id: int
    }

Enable MongoDB Backend
-----------------------

.. code-block:: console

    ./bin/emqx_ctl plugins load emqx_backend_mongo

.. _cassandra_backend:

--------------------------------
Data Persistence using Cassandra
--------------------------------

Config file: etc/plugins/emqx_backend_cassa.conf

Config Cassandra Cluster
-------------------------

Multi node Cassandra cluster is supported: 

.. code-block:: properties
    
    ## Cassandra Node
    backend.ecql.pool1.nodes = 127.0.0.1:9042
    
    ## Cassandra Pool Size
    backend.ecql.pool1.size = 8

    ## Cassandra auto reconnect flag
    backend.ecql.pool1.auto_reconnect = 1

    ## Cassandra Username
    backend.ecql.pool1.username = cassandra

    ## Cassandra Password
    backend.ecql.pool1.password = cassandra

    ## Cassandra Keyspace
    backend.ecql.pool1.keyspace = mqtt

    ## Cassandra Logger type
    backend.ecql.pool1.logger = info

Config Cassandra Persistence Hooks
----------------------------------

.. code-block:: properties

    ## Client Connected Record 
    backend.cassa.hook.client.connected.1    = {"action": {"function": "on_client_connected"}, "pool": "pool1"}

    ## Subscribe Lookup Record 
    backend.cassa.hook.client.connected.2    = {"action": {"function": "on_subscription_lookup"}, "pool": "pool1"}

    ## Client DisConnected Record 
    backend.cassa.hook.client.disconnected.1 = {"action": {"function": "on_client_disconnected"}, "pool": "pool1"}

    ## Lookup Unread Message QOS > 0
    backend.cassa.hook.session.subscribed.1  = {"topic": "#", "action": {"function": "on_message_fetch"}, "pool": "pool1"}

    ## Lookup Retain Message 
    backend.cassa.hook.session.subscribed.2  = {"action": {"function": "on_retain_lookup"}, "pool": "pool1"}

    ## Store Publish Message  QOS > 0
    backend.cassa.hook.message.publish.1     = {"topic": "#", "action": {"function": "on_message_publish"}, "pool": "pool1"}
    
    ## Delete Acked Record
    backend.cassa.hook.session.unsubscribed.1= {"topic": "#", action": {"cql": ["delete from acked where client_id = ${clientid} and topic = ${topic}"]}, "pool": "pool1"}

    ## Store Retain Message 
    backend.cassa.hook.message.publish.2     = {"topic": "#", "action": {"function": "on_message_retain"}, "pool": "pool1"}

    ## Delete Retain Message
    backend.cassa.hook.message.publish.3     = {"topic": "#", "action": {"function": "on_retain_delete"}, "pool": "pool1"}

    ## Store Ack
    backend.cassa.hook.message.acked.1       = {"topic": "#", "action": {"function": "on_message_acked"}, "pool": "pool1"}

Description of Cassandra Persistence Hooks
------------------------------------------

+------------------------+------------------------+-------------------------+----------------------------------+
| hook                   | topic                  | action                  | Description                      |
+========================+========================+=========================+==================================+
| client.connected       |                        | on_client_connected     | Store client connected state     |
+------------------------+------------------------+-------------------------+----------------------------------+
| client.connected       |                        | on_subscribe_lookup     | Subscribed topics                |
+------------------------+------------------------+-------------------------+----------------------------------+
| client.disconnected    |                        | on_client_disconnected  | Store client disconnected state  |
+------------------------+------------------------+-------------------------+----------------------------------+
| session.subscribed     | #                      | on_message_fetch        | Fetch offline messages           |
+------------------------+------------------------+-------------------------+----------------------------------+
| session.subscribed     | #                      | on_retain_lookup        | Lookup retained messages         |
+------------------------+------------------------+-------------------------+----------------------------------+
| message.publish        | #                      | on_message_publish      | Store published messages         |
+------------------------+------------------------+-------------------------+----------------------------------+
| message.publish        | #                      | on_message_retain       | Store retained messages          |
+------------------------+------------------------+-------------------------+----------------------------------+
| message.publish        | #                      | on_retain_delete        | Delete retained messages         |
+------------------------+------------------------+-------------------------+----------------------------------+
| message.acked          | #                      | on_message_acked        | Process ACK                      |
+------------------------+------------------------+-------------------------+----------------------------------+

CQL Arguments Description
-------------------------

Customized CQL command arguments includes:

+----------------------+---------------------------------------+----------------------------------------------------------------+		
| hook                 | Argument                              | Example (${name} in CQL represents available argument          |		
+======================+=======================================+================================================================+		
| client.connected     | clientid                              | insert into conn(clientid) values(${clientid})                 |		
+----------------------+---------------------------------------+----------------------------------------------------------------+		
| client.disconnected  | clientid                              | insert into disconn(clientid) values(${clientid})              |		
+----------------------+---------------------------------------+----------------------------------------------------------------+		
| session.subscribed   | clientid, topic, qos                  | insert into sub(topic, qos) values(${topic}, ${qos})           |		
+----------------------+---------------------------------------+----------------------------------------------------------------+		
| session.unsubscribed | clientid, topic                       | delete from sub where topic = ${topic}                         |		
+----------------------+---------------------------------------+----------------------------------------------------------------+		
| message.publish      | msgid, topic, payload, qos, clientid  | insert into msg(msgid, topic) values(${msgid}, ${topic})       |		
+----------------------+---------------------------------------+----------------------------------------------------------------+		
| message.acked        | msgid, topic, clientid                | insert into ack(msgid, topic) values(${msgid}, ${topic})       |		
+----------------------+---------------------------------------+----------------------------------------------------------------+		
| message.delivered    | msgid, topic, clientid                | insert into delivered(msgid, topic) values(${msgid}, ${topic}) |		
+----------------------+---------------------------------------+----------------------------------------------------------------+		

Config 'action' utlizing CQL
----------------------------

Cassandra backend supports using CLQ in 'action':

.. code-block:: properties

    ## After a client is connected to the EMQ X server, it executes a CQL command(multiple command also supported):
    backend.cassa.hook.client.connected.3 = {"action": {"cql": ["insert into conn(clientid) values(${clientid})"]}, "pool": "pool1"}

Initializing Cassandra 
----------------------

Create KeySpace:

.. code-block:: sql

    CREATE KEYSPACE mqtt WITH REPLICATION = { 'class' : 'SimpleStrategy', 'replication_factor' : 1 };
    USR mqtt;

Import Cassandra tables:

.. code-block:: sql

    cqlsh -e "SOURCE 'emqx_backend_cassa.cql'" 

.. NOTE:: KeySpace is free of choice

Cassandra Client Connection Table
----------------------------------

*mqtt.client* stores client connection states::

    CREATE TABLE mqtt.client (
        client_id text,
        node text,
        state int,
        connected timestamp,
        disconnected timestamp,
        PRIMARY KEY(client_id)
    );

Inquiring a client's connection state::

    select * from mqtt.client where clientid = ${clientid};
    
If client 'test' is online::

    select * from mqtt.client where clientid = 'test';
    
     client_id | connected                       | disconnected  | node          | state
    -----------+---------------------------------+---------------+---------------+-------
          test | 2017-02-14 08:27:29.872000+0000 |          null | emqx@127.0.0.1|     1

Client 'test' is offline::

    select * from mqtt.client where clientid = 'test';
    
     client_id | connected                       | disconnected                    | node          | state
    -----------+---------------------------------+---------------------------------+---------------+-------
          test | 2017-02-14 08:27:29.872000+0000 | 2017-02-14 08:27:35.872000+0000 | emqx@127.0.0.1|     0

Cassandra Subscription Table
----------------------------

*mqtt.sub* stores subscriptions of clients::

    CREATE TABLE mqtt.sub (
        client_id text,
        topic text,
        qos int,
        PRIMARY KEY(client_id, topic)
    );

Client 'test' subscribes to topic 'test_topic1' and 'test_topic2'::

    insert into mqtt.sub(client_id, topic, qos) values('test', 'test_topic1', 1);
    insert into mqtt.sub(client_id, topic, qos) values('test', 'test_topic2', 2);

Inquiring subscriptions of a client::
    
    select * from mqtt_sub where clientid = ${clientid};

Inquiring subscriptions of client 'test'::
    
    select * from mqtt_sub where clientid = 'test';

     client_id | topic       | qos
    -----------+-------------+-----
          test | test_topic1 |   1
          test | test_topic2 |   2
    
Cassandra Message Table
-----------------------

*mqtt.msg* stores MQTT messages::
    
    CREATE TABLE mqtt.msg (
        topic text,
        msgid text,
        sender text,
        qos int,
        retain int,
        payload text,
        arrived timestamp,
        PRIMARY KEY(topic, msgid)
      ) WITH CLUSTERING ORDER BY (msgid DESC);

Inquiring messages published by a client::

    select * from mqtt_msg where sender = ${clientid};

Inquiring messages published by client 'test'::

    select * from mqtt_msg where sender = 'test';
    
     topic | msgid                | arrived                         | payload      | qos | retain | sender
    -------+----------------------+---------------------------------+--------------+-----+--------+--------
     hello | 2PguFrHsrzEvIIBdctmb | 2017-02-14 09:07:13.785000+0000 | Hello world! |   1 |      0 |   test
     world | 2PguFrHsrzEvIIBdctmb | 2017-02-14 09:07:13.785000+0000 | Hello world! |   1 |      0 |   test

Cassandra Retained Message Table
--------------------------------

*mqtt.retain* stores retained messages::
    
    CREATE TABLE mqtt.retain (
        topic text,
        msgid text,
        PRIMARY KEY(topic)
    );

Inquiring retained messages::

    select * from mqtt_retain where topic = ${topic};

Inquiring retained messages with topic 'retain'::

    select * from mqtt_retain where topic = 'retain';

     topic  | msgid                
    --------+----------------------
     retain | 2PguFrHsrzEvIIBdctmb 

Cassandra Acknowledgement Table
--------------------------------

*mqtt.acked* stores acknowledgements from the clients::
    
    CREATE TABLE mqtt.acked (
        client_id text,
        topic text,
        msgid text,
        PRIMARY KEY(client_id, topic)
      );

Enable Cassandra Backend
------------------------

.. code-block:: bash

    ./bin/emqx_ctl plugins load emqx_backend_cassa

