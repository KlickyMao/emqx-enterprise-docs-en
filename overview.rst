
.. _overview:

=======
Oerview
=======

EMQ is a distributed, massively scalable, highly extensible MQTT message broker which can sustain million level connections. It is chosen worldwide by more than 1000 enterprise users. More than 10,000 nodes are deployed and serve more than 30 million mobile and IoT connections.

EMQ X is the enterprise edition of the EMQ broker which extends the function and performance of EMQ. It improves the system architecture of EMQ, adopts Scalable RPC mechanism, provides more reliable clustering and higher performance of message routing.

EMQ X supports persistence MQTT messages to Redis , MySQL, PostgreSQL, MongoDB, Cassandra and other Database. It also supports bridging and forwarding messages to enterprise messaging middleware like Kafka and RabbitMQ.

EMQ X can be used as connection platform for smart hardware, smart home, IoT, automotive networking applications that serve millions of devices.

.. image:: _static/images/emqx_enterprise.png

----------------
Design Objective
----------------

EMQ (Erlang MQTT Broker) is an open source MQTT broker written in Erlang/OTP. Erlang/OTP is a concurrent, fault-tolerant, soft-realtime and distributed programming platform. MQTT is an extremely lightweight publish/subscribe messaging protocol powering IoT, M2M and Mobile applications.

The design objectives of EMQ X focus on enterprise-level applications which require high reliability, ability to sustain massive IoT end devices' MQTT connections and meanwhile to keep the message latency very low.

1. Steadily sustains massive MQTT client connections, a single node can handle 0.5 to 1 million connections.

2. Distributed clustering, low-latency message routing. Single cluster handles 10 million level subscriptions.

3. Extensible broker design. Ability of customizition of various Authentication/ACL and data persistence.

4. Comprehensive IoT protocol supporting: MQTT, MQTT-SN, CoAP, WebSocket and other protocols by properties.

---------
Features
---------

1. Scalable RPC Architecture: segregated cluster management channel and data channel between nodes.

2. Fastlane subscription: dedicated Fastlane message routing for IoT data collection.

3. Persistence to Redis: subscriptions, client connection status, MQTT messages, retained messages, SUB/UNSUB events.

4. Persistence to MySQL: subscriptions, client connection status, MQTT messages, retained messages.
   
5. Persistence to PostgreSQL: subscriptions, client connection status, MQTT messages, retained messages.
 
6. Persistence to MongoDB: subscriptions, client connection status, MQTT messages, retained messages.

7. Bridge to Kafka: EMQ X forwards MQTT messages, client connected/disconnected event to Kafka.

8. Bridge to RabbitMQ: EMQ X forwards MQTT messages, client connected/disconnected event to RabbitMQ.

.. _scalable_rpc:

-------------------------
Scalable RPC Architecture
-------------------------

EMQ X improved the communication mechanism between distributed nodes, the cluster management channel and the data channel are segregated, the message throughput and the cluster reliability are greatly improved.

.. NOTE:: the dash line indicates the cluster management and the solid line indicates the data exchange.

.. image:: _static/images/scalable_rpc.png

Scalable RPC configuration::

    ## TCP server port.
    rpc.tcp_server_port = 5369

    ## Default TCP port for outgoing connections
    rpc.tcp_client_port = 5369

    ## Client connect timeout
    rpc.connect_timeout = 5000

    ## Client and Server send timeout
    rpc.send_timeout = 5000

    ## Authentication timeout
    rpc.authentication_timeout = 5000

    ## Default receive timeout for call() functions
    rpc.call_receive_timeout = 15000

    ## Socket keepalive configuration
    rpc.socket_keepalive_idle = 7200

    ## Seconds between probes
    rpc.socket_keepalive_interval = 75

    ## Probes lost to close the connection
    rpc.socket_keepalive_count = 9

.. NOTE:: If firewalls are deployed among the nodes, the 5369 port on them must be opened.

.. _fastlane:

---------------------
Fastlane Subscription
---------------------

EMQ X supports Fastlane Subscription, it greatly enhanced the message routing efficience and is thus very suitable for big data collection of IoT applications.

.. image:: _static/images/fastlane.png

Fastlane usage: *$fastlane/* prefix + topic。

Fastlane limitations:

1. CleanSession = true
2. Qos = 0

Fastlane subscription is suitable for IoT sensor data collection::

                 -----------------
    Sensor ----> |               |
                 |     EMQ X     | --$fastlane/$queue/#--> Subscriber
                 |    Cluster    | --$fastlane/$queue/#--> Subscriber
    Sensor ----> |               |
                 -----------------

----------------------
Subscription by Broker
----------------------

EMQ X supports subscription by broker. It is not necessary that a client subscribes to any topics after connect, the EQM X broker can load the subscriptions for it from Redis or database.

In a context of low power consumption and low network bandwidth, The subscription by broker can save the packets exchanged and the transport volume.

---------------------
MQTT Data Persistence
---------------------

EMQ X supports MQTT data (subscription, messages, client status) persistence to Redis, MySQL, PostgreSQL, MongoDB and Cassandra database:

.. image:: _static/images/storage.png

For details please refer to the "Data Persistence" chapter.

------------------------
Message Bridge & Forward 
------------------------

EMQ X supports bridging and forwarding MQTT messages to systems like RabbitMQ and Kafka. It can be deployed as IoT Hub:

.. image:: _static/images/iothub.png

