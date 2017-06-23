
.. _overview:

=========
Oerview
=========

EMQ is one of the most deployed MQTT message broker which can sustain million level connection. It is chosen worldwide by more than 5000 enterprise user. More than 100,000 nodes are deployed and serve 30 million connections.

EMQ X extends the function and prformance of  EMQ. It is a commercial version for enterprise. It improves the system architecture of EMQ, adopts Scalable RPC mechanism, provides more reliable clustering and higher performance of message routing.

EMQ X supprts persistence MQTT messages to Redis , MySQL, PostgreSQL, MongoDB, Cassandra and other Database. It also supprts bridging and forwarding messages to enterprise middleware like Kafka and RabbitMQ.

EMQ X can be used as access platform for smart hardware, smart home, IoT, automotive networking applications that serve millions end device.

.. image:: _static/images/emqx_enterprise.png

----------------
Design Objective
----------------

EMQ (Erlang MQTT Broker) is an open source MQTT broker written in Erlang/OTP. Erlang/OTP is a concurrent, fault-tolerant, soft-realtime and distributed programming platform. MQTT is an extremely lightweight publish/subscribe messaging protocol powering IoT, M2M and Mobile applications.

The design objectives of EMQ X focus on enterprise level applications which require high reliability, ability to sustain massive IoT end devices' MQTT connections and meanwhile to keep the message latency very low.

1. Steadily sustains massive MQTT client connct, a single node handles 0.5 to 1 million connect.

2. Distributed clustering, low message routing latency. Single cluster handles 10 million level message routing.

3. Extensible server design. Ability of customizition of various  Auth and high performance data persistencei.

4. Comprehensive IoT protocol supporting: MQTT, MQTT-SN, CoAP, WebSocket and other protocols by properties.

-------------
Features
-------------

1. Scalable RPC Arcitecture: segregated cluster management channel and data channel between nodes.

2. Fastlane subscription: dedicated Fastlane message routing for IoT data collection.

3. Persistence to Redis: subscriptions, client connection status, MQTT messages, retained messages, SUB/UNSUB events.

4. Persistence to MySQL: subscriptions, client connection status, MQTT messages, retained messages.
   
5. Persistence to PostgreSQL: subscription, client connection status, MQTT messages, retained messages.
 
6. Persistence to MongoDB: subscription, client connection status, MQTT messages, retained messages.

7. Bridge to Kafka: EMQ X bridges MQTT messages, client connected/disconnected event to Kafka.

8. Bridge to RabbitMQ: EMQ X bridges MQTT messages, client connected/disconnected event to RabbitMQ.

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

------------------
Agent Subscription
------------------

EMQ X supports agent subscription. It is not necessary that a client subscribes to any topics after connect, the EQM X agent can load the subscriptions for it from the database.

In a context of low power consumption and low network bandwidth,EMQ X agent subscription saves the packets exchanged and the transport volume.

------------------------
Message Data Persistence
------------------------

EMQ X supports message data (subscription, messages, client status) persistence to Redis, MySQL, PostgreSQL, MongoDB and Cassandra database:

.. image:: _static/images/storage.png

For details please refer to the "Data Persistence" chapter.


------------------------
Message bridge & Forward 
------------------------

EMQ X supports bridging and forwarding MQTT messages to systems like RabbitMQ and Kafka. It can be deployed as IoT Hub:

.. image:: _static/images/iothub.png

