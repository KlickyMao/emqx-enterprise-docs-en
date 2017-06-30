
.. _cli:

============
CLI Commands
============

EMQ X provides './bin/emqx_ctl' as a CLI management tool.

----------
status
----------

Check EMQ X status::

    $ ./bin/emqx_ctl status

    Node 'emqx@127.0.0.1' is started
    emqx 2.1.0 is running

----------
broker
----------

Query basic information, statistics and performance metrics of the broker.

+----------------+----------------------------------------------------+
| broker         | Show version, description and up-time              |
+----------------+----------------------------------------------------+
| broker pubsub  | Show status of core pubsub process                 |
+----------------+----------------------------------------------------+
| broker stats   | Show statistics of client, session, topic,         |
|                | subscription and route                             |
+----------------+----------------------------------------------------+
| broker metrics | Show metrics of MQTT bytes, packets and messages   |
+----------------+----------------------------------------------------+

Query EMQ X version, description and up-time::

    $ ./bin/emqx_ctl broker

    sysdescr  : EMQ X
    version   : 2.1.0
    uptime    : 3 hours, 56 minutes, 48 seconds
    datetime  : 2017-01-12 23:22:05

broker stats
------------

Query statistics of MQTT Client, Session, Topic, Subscription and Route::

    $ ./bin/emqx_ctl broker stats

    clients/count       : 0
    clients/max         : 0
    retained/count      : 3
    retained/max        : 3
    routes/count        : 0
    routes/max          : 0
    sessions/count      : 0
    sessions/max        : 0
    subscribers/count   : 0
    subscribers/max     : 0
    subscriptions/count : 0
    subscriptions/max   : 0
    topics/count        : 0
    topics/max          : 0

broker metrics
--------------

Query metrics of Bytes, MQTT Packets and Messages(sent/received):: 

    $ ./bin/emqx_ctl broker metrics

    bytes/received          : 383
    bytes/sent              : 108
    messages/dropped        : 3
    messages/qos0/received  : 0
    messages/qos0/sent      : 3
    messages/qos1/received  : 0
    messages/qos1/sent      : 0
    messages/qos2/dropped   : 0
    messages/qos2/received  : 6
    messages/qos2/sent      : 0
    messages/received       : 6
    messages/retained       : 3
    messages/sent           : 3
    packets/connack         : 7
    packets/connect         : 7
    packets/disconnect      : 6
    packets/pingreq         : 0
    packets/pingresp        : 0
    packets/puback/missed   : 0
    packets/puback/received : 0
    packets/puback/sent     : 0
    packets/pubcomp/missed  : 0
    packets/pubcomp/received: 0
    packets/pubcomp/sent    : 6
    packets/publish/received: 6
    packets/publish/sent    : 3
    packets/pubrec/missed   : 0
    packets/pubrec/received : 0
    packets/pubrec/sent     : 6
    packets/pubrel/missed   : 0
    packets/pubrel/received : 6
    packets/pubrel/sent     : 0
    packets/received        : 26
    packets/sent            : 23
    packets/suback          : 1
    packets/subscribe       : 1
    packets/unsuback        : 0
    packets/unsubscribe     : 0

-----------
cluster
-----------

Manage EMQ X nodes (processes) in a cluster:

+-----------------------+-----------------------------+
| cluster join <Node>   | Join the cluster            |
+-----------------------+-----------------------------+
| cluster leave         | Leave the cluster           |
+-----------------------+-----------------------------+
| cluster remove <Node> | Remove a node from cluster  |
+-----------------------+-----------------------------+
| cluster status        | Query cluster status & nodes|
+-----------------------+-----------------------------+

Example: Creating a cluster with two nodes on localhost:

+-----------+---------------------+-------------+
| Dir       | Node                | MQTT Port   |
+-----------+---------------------+-------------+
| emqx1     | emqx1@127.0.0.1     | 1883        |
+-----------+---------------------+-------------+
| emqx2     | emqx2@127.0.0.1     | 2883        |
+-----------+---------------------+-------------+

Start emqx1::

    cd emqx1 && ./bin/emqx start

Start emqx2::

    cd emqx2 && ./bin/emqx start

In emqx2 directory::

    $ ./bin/emqx_ctl cluster join emqx1@127.0.0.1

    Join the cluster successfully.
    Cluster status: [{running_nodes,['emqx1@127.0.0.1','emqx2@127.0.0.1']}]

Query cluster status::

    $ ./bin/emqx_ctl cluster status

    Cluster status: [{running_nodes,['emqx2@127.0.0.1','emqx1@127.0.0.1']}]

Message route between nodes::

    # subscribe to topic 'x' on node 'emqx1'
    mosquitto_sub -t x -q 1 -p 1883

    # publish to topic 'x' on node 'emqx2'
    mosquitto_pub -t x -q 1 -p 2883 -m hello

emqx2 leaves the cluster::

    cd emqx2 && ./bin/emqx_ctl cluster leave

Or revmove emqx2 on emqx1::

    cd emqx1 && ./bin/emqx_ctl cluster remove emqx2@127.0.0.1

-----------
clients
-----------

Query MQTT clients connected to the broker:

+-------------------------+-----------------------------+
| clients list            | List all MQTT clients       |
+-------------------------+-----------------------------+
| clients show <ClientId> | Query client by ClientId    |
+-------------------------+-----------------------------+
| clients kick <ClientId> | Kick out by ClientId        |
+-------------------------+-----------------------------+

clients list
------------

Query all MQTT clients connected to the broker::

    $ ./bin/emqx_ctl clients list

    Client(mosqsub/43832-airlee.lo, clean_sess=true, username=test, peername=127.0.0.1:64896, connected_at=1452929113)
    Client(mosqsub/44011-airlee.lo, clean_sess=true, username=test, peername=127.0.0.1:64961, connected_at=1452929275)
    ...

Properties of client:

+--------------+-----------------------------+
| clean_sess   | Clean session flag          |
+--------------+-----------------------------+
| username     | Username of client          |
+--------------+-----------------------------+
| peername     | Peername of TCP connection  |
+--------------+-----------------------------+
| connected_at | Timestamp of connection     |
+--------------+-----------------------------+

clients show <ClientId>
-----------------------

Query client by ClientId::

    ./bin/emqx_ctl clients show "mosqsub/43832-airlee.lo"

    Client(mosqsub/43832-airlee.lo, clean_sess=true, username=test, peername=127.0.0.1:64896, connected_at=1452929113)

clients kick <ClientId>
-----------------------

Kick out client by ClientId::

    ./bin/emqx_ctl clients kick "clientid"

------------
sessions
------------

Query all MQTT sessions. The broker will create a session for each MQTT client. Session type will be persistent if clean_session flag is true, transient otherwise.

+--------------------------+-----------------------------+
| sessions list            | List all sessions           |
+--------------------------+-----------------------------+
| sessions list persistent | List all persistent sessions|
+--------------------------+-----------------------------+
| sessions list transient  | List all transient sessions |
+--------------------------+-----------------------------+
| sessions show <ClientId> | Query sessions by ClientID  |
+--------------------------+-----------------------------+

sessions list
-------------

List all sessions::

    $ ./bin/emqx_ctl sessions list

    Session(clientid, clean_sess=false, max_inflight=100, inflight_queue=0, message_queue=0, message_dropped=0, awaiting_rel=0, awaiting_ack=0, awaiting_comp=0, created_at=1452935508)
    Session(mosqsub/44101-airlee.lo, clean_sess=true, max_inflight=100, inflight_queue=0, message_queue=0, message_dropped=0, awaiting_rel=0, awaiting_ack=0, awaiting_comp=0, created_at=1452935401)

Properties of session:

+-------------------+----------------------------------------------------------------+
| clean_sess        | clean sess flag. false: persistent, true: transient            |
+-------------------+----------------------------------------------------------------+
| max_inflight      | Inflight window (Max number of messages delivering)            |
+-------------------+----------------------------------------------------------------+
| inflight_queue    | Inflight Queue Size                                            |
+-------------------+----------------------------------------------------------------+
| message_queue     | Message Queue Size                                             |
+-------------------+----------------------------------------------------------------+
| message_dropped   | Number of Messages Dropped since queue is full                   |
+-------------------+----------------------------------------------------------------+
| awaiting_rel      | The number of QoS2 messages received and waiting for PUBREL    |
+-------------------+----------------------------------------------------------------+
| awaiting_ack      | The number of QoS1/2 messages delivered and waiting for PUBACK |
+-------------------+----------------------------------------------------------------+
| awaiting_comp     | The number of QoS2 messages delivered and waiting for PUBCOMP  |
+-------------------+----------------------------------------------------------------+
| created_at        | Timestamp when the session is created                          |
+-------------------+----------------------------------------------------------------+


sessions list persistent
------------------------

Query all persistent sessions::

    $ ./bin/emqx_ctl sessions list persistent

    Session(clientid, clean_sess=false, max_inflight=100, inflight_queue=0, message_queue=0, message_dropped=0, awaiting_rel=0, awaiting_ack=0, awaiting_comp=0, created_at=1452935508)

sessions list transient
-----------------------

Query all transient sessions::

    $ ./bin/emqx_ctl sessions list transient

    Session(mosqsub/44101-airlee.lo, clean_sess=true, max_inflight=100, inflight_queue=0, message_queue=0, message_dropped=0, awaiting_rel=0, awaiting_ack=0, awaiting_comp=0, created_at=1452935401)

sessions show <ClientId>
------------------------

Query sessions by ClientId::

    $ ./bin/emqx_ctl sessions show clientid

    Session(clientid, clean_sess=false, max_inflight=100, inflight_queue=0, message_queue=0, message_dropped=0, awaiting_rel=0, awaiting_ack=0, awaiting_comp=0, created_at=1452935508)

----------
routes
----------

Show routing table of the broker.

routes list
-----------

List all routes::

    $ ./bin/emqx_ctl routes list

    t2/# -> emqx2@127.0.0.1
    t/+/x -> emqx2@127.0.0.1,emq1@127.0.0.1

routes show <Topic>
-------------------

Show the route of a topic::

    $ ./bin/emqx_ctl routes show t/+/x

    t/+/x -> emqx2@127.0.0.1,emqx1@127.0.0.1

----------
topics
----------

Query the topic table of the broker.

topics list
-----------

Qeury all topics::

    $ ./bin/emqx_ctl topics list

    $SYS/brokers/emqx@127.0.0.1/metrics/packets/subscribe: static
    $SYS/brokers/emqx@127.0.0.1/stats/subscriptions/max: static
    $SYS/brokers/emqx2@127.0.0.1/stats/subscriptions/count: static
    ...

topics show <Topic>
-------------------

Query a particular topic::

    $ ./bin/emqx_ctl topics show '$SYS/brokers'

    $SYS/brokers: static

-----------------
subscriptions
-----------------

Query the subscription table of the broker.

+--------------------------------------------+--------------------------------+
| subscriptions list                         | List all subscriptions         |
+--------------------------------------------+--------------------------------+
| subscriptions show <ClientId>              | Show subscriptions by ClientID |
+--------------------------------------------+--------------------------------+

subscriptions list
------------------

List all subscriptins::

    $ ./bin/emqx_ctl subscriptions list

    mosqsub/91042-airlee.lo -> t/y:1
    mosqsub/90475-airlee.lo -> t/+/x:2

subscriptions show <ClientId>
-----------------------------

Show subscritions by ClientID::

    $ ./bin/emqx_ctl subscriptions show 'mosqsub/90475-airlee.lo'

    mosqsub/90475-airlee.lo -> t/+/x:2

-----------
plugins
-----------

List, load or unload plugins. Plugins are placed in 'pluglins/' directory.

+---------------------------+-------------------------+
| plugins list              | List all plugins        |
+---------------------------+-------------------------+
| plugins load <Plugin>     | Load plugin             |
+---------------------------+-------------------------+
| plugins unload <Plugin>   | unload plugin           |
+---------------------------+-------------------------+

plugins list
------------

List all plugins::

    $ ./bin/emqx_ctl plugins list

    Plugin(emqx_auth_clientid, version=2.1.0, description=EMQ X Authentication with ClientId/Password, active=false)
    Plugin(emqx_auth_http, version=2.1.0, description=EMQ X Authentication/ACL with HTTP API, active=false)
    Plugin(emqx_auth_ldap, version=2.1.0, description=EMQ X Authentication/ACL with LDAP, active=false)
    Plugin(emqx_auth_mongo, version=2.1.0, description=EMQ X Authentication/ACL with MongoDB, active=false)
    Plugin(emqx_auth_mysql, version=2.1.0, description=EMQ X Authentication/ACL with MySQL, active=false)
    Plugin(emqx_auth_pgsql, version=2.1, description=EMQ X Authentication/ACL with PostgreSQL, active=false)
    Plugin(emqx_auth_redis, version=2.1.0, description=EMQ X Authentication/ACL with Redis, active=false)
    Plugin(emqx_auth_username, version=2.1.0, description=EMQ X Authentication with Username/Password, active=false)
    Plugin(emqx_backend_cassa, version=2.1.0, description=EMQ X Cassandra Backend, active=false)
    Plugin(emqx_backend_mongo, version=2.1.0, description=EMQ X Mongodb Backend, active=false)
    Plugin(emqx_backend_mysql, version=2.1, description=EMQ X MySQL Backend, active=false)
    Plugin(emqx_backend_pgsql, version=2.1.0, description=EMQ X PostgreSQL Backend, active=false)
    Plugin(emqx_backend_redis, version=2.1.0, description=EMQ X Redis Backend, active=false)
    Plugin(emqx_bridge_kafka, version=2.1.0, description=EMQ X Kafka Bridge, active=false)
    Plugin(emqx_bridge_rabbit, version=2.1.0, description=EMQ X Bridge RabbitMQ, active=false)
    Plugin(emqx_dashboard, version=2.1.0, description=EMQ X Dashboard, active=true)
    Plugin(emqx_modules, version=2.1.0, description=EMQ X Modules, active=true)
    Plugin(emqx_recon, version=2.1.0, description=Recon Plugin, active=true)
    Plugin(emqx_reloader, version=2.1, description=Reloader Plugin, active=false)
    Plugin(emqx_retainer, version=2.1, description=EMQ X Retainer, active=true)

Properties of a plugin:

+-------------+-----------------+
| version     | Version         |
+-------------+-----------------+
| description | Description     |
+-------------+-----------------+
| active      | Is loaded?      |
+-------------+-----------------+

load <Plugin>
-------------

Load a plugin::

    $ ./bin/emqx_ctl plugins load emqx_recon

    Start apps: [emqx_recon]
    Plugin emqx_recon loaded successfully.

unload <Plugin>
---------------

Unload a plugin::

    $ ./bin/emqx_ctl plugins unload emqx_recon

    Plugin emqx_recon unloaded successfully.

-----------
bridges
-----------

Bridge multiple EMQ X nodes:: 

                  ---------             ---------
    Publisher --> | node1 | --Bridge--> | node2 | --> Subscriber
                  ---------             ---------

+----------------------------------------+-------------------------------+
| bridges list                           | List all bridges              |
+----------------------------------------+-------------------------------+
| bridges options                        | Show bridge options           |
+----------------------------------------+-------------------------------+
| bridges start <Node> <Topic>           | Create a bridge               |
+----------------------------------------+-------------------------------+
| bridges start <Node> <Topic> <Options> | Create a bridge with options  |
+----------------------------------------+-------------------------------+
| bridges stop <Node> <Topic>            | Delete a bridge               |
+----------------------------------------+-------------------------------+

Create a bridge teween emqx1 and emqx2, forward 'sensor/#' to exqx2::

    $ ./bin/emqx_ctl bridges start emqx2@127.0.0.1 sensor/#

    bridge is started.

    $ ./bin/emqx_ctl bridges list

    bridge: emqx1@127.0.0.1--sensor/#-->emqx2@127.0.0.1

Test bridge 'emqx1--sensor/#-->emqx2'::

    #on emqx2

    mosquitto_sub -t sensor/# -p 2883 -d

    #on emqx1

    mosquitto_pub -t sensor/1/temperature -m "37.5" -d

bridge options
--------------

Show bridge options::

    $ ./bin/emqx_ctl bridges options

    Options:
      qos     = 0 | 1 | 2
      prefix  = string
      suffix  = string
      queue   = integer
    Example:
      qos=2,prefix=abc/,suffix=/yxz,queue=1000

bridges stop <Node> <Topic>
---------------------------

Delete the bridge 'emqx1--sensor/#-->emqx2'::

    $ ./bin/emqx_ctl bridges stop emqx2@127.0.0.1 sensor/#

    bridge is stopped.

------
vm
------

Query the load, cpu, memory, processes and IO information of the Erlang VM.

+-------------+-----------------------------------+
| vm all      | Query all                         |
+-------------+-----------------------------------+
| vm load     | Query VM Load                     |
+-------------+-----------------------------------+
| vm memory   | Query Memory Usage                |
+-------------+-----------------------------------+
| vm process  | Query Number of Erlang Processes  |
+-------------+-----------------------------------+
| vm io       | Query Max Fds of VM               |
+-------------+-----------------------------------+

vm load
-------

Query VM load::

    $ ./bin/emqx_ctl vm load

    cpu/load1               : 2.21
    cpu/load5               : 2.60
    cpu/load15              : 2.36

vm memory
---------

Query VM memory::

    $ ./bin/emqx_ctl vm memory

    memory/total            : 23967736
    memory/processes        : 3594216
    memory/processes_used   : 3593112
    memory/system           : 20373520
    memory/atom             : 512601
    memory/atom_used        : 491955
    memory/binary           : 51432
    memory/code             : 13401565
    memory/ets              : 1082848

vm process
----------

Query number of Erlang processes::

    $ ./bin/emqx_ctl vm process

    process/limit           : 8192
    process/count           : 221

vm io
-----

Query max, active file descriptions of IO::

    $ ./bin/emqx_ctl vm io

    io/max_fds              : 2560
    io/active_fds           : 1

---------
trace
---------

Trace MQTT packets, messages(sent/received) by ClientID or topic. Print logs to file.

+-----------------------------------+-----------------------------------+
| trace list                        | List all the traces               |
+-----------------------------------+-----------------------------------+
| trace client <ClientId> <LogFile> | Trace a client                    |
+-----------------------------------+-----------------------------------+
| trace client <ClientId> off       | Stop tracing the client           |
+-----------------------------------+-----------------------------------+
| trace topic <Topic> <LogFile>     | Trace a topic                     |
+-----------------------------------+-----------------------------------+
| trace topic <Topic> off           | Stop tracing the topic            |
+-----------------------------------+-----------------------------------+


trace client <ClientId> <LogFile>
---------------------------------

Start tracing a client::

    $ ./bin/emqx_ctl trace client clientid log/clientid_trace.log

    trace client clientid successfully.


trace client <ClientId> off
---------------------------

Stop tracing a client::

    $ ./bin/emqx_ctl trace client clientid off

    stop to trace client clientid successfully.

trace topic <Topic> <LogFile>
-----------------------------

Start tracing a topic::

    $ ./bin/emqx_ctl trace topic topic log/topic_trace.log

    trace topic topic successfully.

trace topic <Topic> off
-----------------------

Stop tracing a topic::

    $ ./bin/emqx_ctl trace topic topic off

    stop to trace topic topic successfully.

trace list
----------

Lista all traces::

    $ ./bin/emqx_ctl trace list

    trace client clientid -> log/clientid_trace.log
    trace topic topic -> log/topic_trace.log

---------
listeners
---------

Show all TCP listeners::

    $ ./bin/emqx_ctl listeners

    listener on mqtt:wss:8084
      acceptors       : 4
      max_clients     : 64
      current_clients : 0
      shutdown_count  : []
    listener on mqtt:ssl:8883
      acceptors       : 4
      max_clients     : 1024
      current_clients : 0
      shutdown_count  : []
    listener on mqtt:ws:8083
      acceptors       : 4
      max_clients     : 64
      current_clients : 0
      shutdown_count  : []
    listener on mqtt:tcp:0.0.0.0:1883
      acceptors       : 8
      max_clients     : 1024
      current_clients : 1
      shutdown_count  : [{closed,2}]
    listener on mqtt:tcp:127.0.0.1:11883
      acceptors       : 4
      max_clients     : 1024
      current_clients : 0
      shutdown_count  : []
    listener on dashboard:http:18083
      acceptors       : 2
      max_clients     : 512
      current_clients : 0
      shutdown_count  : []

listener Parameters:

+-----------------+--------------------------------------+
| acceptors       | TCP Acceptor Pool                    |
+-----------------+--------------------------------------+
| max_clients     | Max number of clients                |
+-----------------+--------------------------------------+
| current_clients | Count of current clients             |
+-----------------+--------------------------------------+
| shutdown_count  | Statistics of client shutdown reason |
+----------------+---------------------------------------+

----------
mnesia
----------

Show system_info of mnesia database.

----------
admins
----------

The 'admins' CLI is used to add/del admin account, it is registered by the dashboard plugin.

+------------------------------------+-----------------------------+
| admins add <Username> <Password>   | Add admin account           |
+------------------------------------+-----------------------------+
| admins passwd <Username> <Password>| Reset admin password        |
+------------------------------------+-----------------------------+
| admins del <Username>              | Delete admin account        |
+------------------------------------+-----------------------------+


admins add
----------

Create admin account::

    $ ./bin/emqx_ctl admins add root public
    ok

admins passwd
-------------

Reset admin password::

    $ ./bin/emqx_ctl admins passwd root private
    ok

admins del
----------

Delete admin account::

    $ ./bin/emqx_ctl admins del root
    ok

