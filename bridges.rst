
.. _bridges:

=======
Bridges
=======

EMQ X can bridge and forward messages to Kafka, RabbitMQ or other EMQ X nodes. Meanwhile, mosquitto and rsm can be bridged to EMQ X using common MQTT connection.

.. _kafka_bridge:

-------------
Kafka Bridge
-------------

EMQ X bridges and forwards MQTT messages to Kafka cluster::

                  ---------             ---------
    Publisher --> | EMQ X | --Bridge--> | Kafka | --> Subscriber
                  ---------             ---------

Config file for Kafka bridge plugin: etc/plugins/emqx_bridge_kafka.conf

Config Kafka Cluster
---------------------

.. code-block:: properties

    ## Kafka Server
    bridge.kafka.pool1.server = 127.0.0.1:9092

    ## Kafka Pool Size 
    bridge.kafka.pool1.pool_size = 8
    
    ## Kafka Parition Strategy
    bridge.kafka.parition_strategy = random

Config Kafka Bridge Hooks
-------------------------

.. code-block:: properties
    
    ## Client Connected Record Hook
    bridge.kafka.hook.client.connected.1 = {"action": "on_client_connected", "pool": "pool1", "topic": "client_connected"}

    ## Client Disconnected Record Hook
    bridge.kafka.hook.client.disconnected.1 = {"action": "on_client_disconnected", "pool": "pool1", "topic": "client_disconnected"}

    ## Session Subscribed Record Hook
    bridge.kafka.hook.session.subscribed.1 = {"action": "on_session_subscribed", "filter": "#", "pool": "pool1", "topic": "session_subscribed"}

    ## Session Unsubscribed Record Hook
    bridge.kafka.hook.session.unsubscribed.1 = {"action": "on_session_unsubscribed", "filter": "#", "pool": "pool1", "topic": "session_unsubscribed"}

    ## Message Publish Record Hook
    bridge.kafka.hook.message.publish.1 = {"action": "on_message_publish", "filter": "#", "pool": "pool1", "topic": "message_publish"}

    ## Message Delivered Record Hook
    bridge.kafka.hook.message.delivered.1 = {"action": "on_message_delivered", "filter": "#", "pool": "pool1", "topic": "message_delivered"}

    ## Message Acked Record Hook
    bridge.kafka.hook.message.acked.1 = {"action": "on_message_acked", "filter": "#", "pool": "pool1", "topic": "message_acked"}

Description of Kafka Bridge Hooks
---------------------------------

+------------------------+----------------------------------+
| action                 | Description                      |
+========================+==================================+
| on_client_connected    | Client connected                 |
+------------------------+----------------------------------+
| on_client_disconnected | Client disconnected              |
+------------------------+----------------------------------+
| on_session_subscribed  | Topics subscribed                |
+------------------------+----------------------------------+
| on_session_unsubscribed| Topics unsubscribed              |
+------------------------+----------------------------------+
| on_message_publish     | Messages published               |
+------------------------+----------------------------------+
| on_message_delivered   | Messages delivered               |
+------------------------+----------------------------------+
| on_message_acked       | Messages acknowledged            |
+------------------------+----------------------------------+

Forwarding Client Connected / Disconnected Events to Kafka
-----------------------------------------------------------

Client goes online, EMQ X forwards 'client_connected' event message to Kafka:

.. code-block:: javascript
    
    topic = "client_connected",
    value = {
             "client_id": ${clientid}, 
             "node": ${node}, 
             "ts": ${ts}
            }

Client goes offline, EMQ X forwards 'client_disconnected' event message to Kafka:

.. code-block:: javascript

    topic = "client_disconnected",
    value = {
            "client_id": ${clientid},
            "reason": ${reason},
            "node": ${node},
            "ts": ${ts}
            }

Forwarding Subscription Event to Kafka
---------------------------------------

.. code-block:: javascript
    
    topic = session_subscribed

    value = {
             "client_id": ${clientid},
             "topic": ${topic},
             "qos": ${qos},
             "node": ${node},
             "ts": ${timestamp}
            }

Forwarding Unsubscription Event to Kafka
----------------------------------------

.. code-block:: javascript
    
    topic = session_unsubscribed

    value = {
             "client_id": ${clientid},
             "topic": ${topic},
             "qos": ${qos},
             "node": ${node},
             "ts": ${timestamp}
            }

Forwarding MQTT Messages to Kafka
---------------------------------

.. code-block:: javascript

    topic = message_publish

    value = {
             "client_id": ${clientid},
             "username": ${username},
             "topic": ${topic},
             "payload": ${payload},
             "qos": ${qos},
             "node": ${node}, 
             "ts": ${timestamp}
            }

Forwarding MQTT Message Deliver Event to Kafka
-----------------------------------------------

.. code-block:: javascript
    
    topic = message_delivered

    value = {"client_id": ${clientid},
             "username": ${username},
             "from": ${fromClientId},
             "topic": ${topic},
             "payload": ${payload},
             "qos": ${qos},
             "node": ${node},
             "ts": ${timestamp}
            }

Forwarding MQTT Message Ack Event to Kafka
-------------------------------------------

.. code-block:: javascript
    
    topic = message_acked

    value = {
             "client_id": ${clientid},
             "username": ${username},
             "from": ${fromClientId},
             "topic": ${topic},
             "payload": ${payload},
             "qos": ${qos},
             "node": ${node},
             "ts": ${timestamp}
            }

Examples of Kafka Message Consumption
--------------------------------------

Kafka consumes MQTT clients connected / disconnected event messages::

    sh kafka-console-consumer.sh --zookeeper localhost:2181 --topic client_connected --from-beginning

    sh kafka-console-consumer.sh --zookeeper localhost:2181 --topic client_disconnected --from-beginning

Kafka consumes MQTT subscription messages::

    sh kafka-console-consumer.sh --zookeeper localhost:2181 --topic session_subscribed --from-beginning

    sh kafka-console-consumer.sh --zookeeper localhost:2181 --topic session_unsubscribed --from-beginning

Kafka consumes MQTT published messages::

    sh kafka-console-consumer.sh --zookeeper localhost:2181 --topic message_publish --from-beginning
    
Kafka consumes MQTT message Deliver and Ack event messages::

    sh kafka-console-consumer.sh --zookeeper localhost:2181 --topic message_delivered --from-beginning
    
    sh kafka-console-consumer.sh --zookeeper localhost:2181 --topic message_acked --from-beginning
    
.. NOTE:: the payload is base64 encoded 

Enable Kafka Bridge
-------------------

.. code-block:: bash

    ./bin/emqx_ctl plugins load emqx_bridge_kafka

.. _rabbit_bridge:

---------------
RabbitMQ Bridge
---------------

EMQ X bridges and forwards MQTT messages to RabbitMQ cluster::

                  ----------             ------------ 
    Publisher --> | EMQ X  | --Bridge--> | RabbitMQ |  --> Subscriber
                  ----------             ------------ 

Config file of RabbitMQ bridge plugin: etc/plugins/emqx_bridge_rabbit.conf

Config RabbitMQ Cluster
-----------------------

.. code-block:: properties

    ## Rabbit Brokers Server
    bridge.rabbit.1.server = 127.0.0.1:5672

    ## Rabbit Brokers pool_size
    bridge.rabbit.1.pool_size = 4

    ## Rabbit Brokers username
    bridge.rabbit.1.username = guest

    ## Rabbit Brokers password
    bridge.rabbit.1.password = guest

    ## Rabbit Brokers virtual_host
    bridge.rabbit.1.virtual_host = /

    ## Rabbit Brokers heartbeat
    bridge.rabbit.1.heartbeat = 0

    # bridge.rabbit.2.server = 127.0.0.1:5672

    # bridge.rabbit.2.pool_size = 8

    # bridge.rabbit.1.username = guest

    # bridge.rabbit.1.password = guest

    # bridge.rabbit.1.virtual_host = /

    # bridge.rabbit.1.heartbeat = 0

Config RabbitMQ Bridge Hooks
----------------------------

.. code-block:: properties

    ## Bridge Hooks
    bridge.rabbit.hook.client.subscribe.1 = {"action": "on_client_subscribe", "rabbit": 1, "exchange": "direct:emq.subscription"}

    bridge.rabbit.hook.client.unsubscribe.1 = {"action": "on_client_unsubscribe", "rabbit": 1, "exchange": "direct:emq.unsubscription"}

    bridge.rabbit.hook.message.publish.1 = {"topic": "$SYS/#", "action": "on_message_publish", "rabbit": 1, "exchange": "topic:emq.$sys"}

    bridge.rabbit.hook.message.publish.2 = {"topic": "#", "action": "on_message_publish", "rabbit": 1, "exchange": "topic:emq.pub"}

    bridge.rabbit.hook.message.acked.1 = {"action": "on_message_acked", "rabbit": 1, "exchange": "topic:emq.acked"}

Forwarding Subscription Event to RabbitMQ
-----------------------------------------

.. code-block:: javascript

    routing_key = subscribe
    exchange = emq.subscription
    headers = [{<<"x-emq-client-id">>, binary, ClientId}]
    payload = jsx:encode([{Topic, proplists:get_value(qos, Opts)} || {Topic, Opts} <- TopicTable])

Forwarding Unsubscription Event to RabbitMQ
-------------------------------------------

.. code-block:: javascript

    routing_key = unsubscribe
    exchange = emq.unsubscription
    headers = [{<<"x-emq-client-id">>, binary, ClientId}]
    payload = jsx:encode([Topic || {Topic, _Opts} <- TopicTable]),

Forwarding MQTT Messages to RabbitMQ
-------------------------------------

.. code-block:: javascript

    routing_key = binary:replace(binary:replace(Topic, <<"/">>, <<".">>, [global]),<<"+">>, <<"*">>, [global])
    exchange = emq.$sys | emq.pub
    headers = [{<<"x-emq-publish-qos">>, byte, Qos},
               {<<"x-emq-client-id">>, binary, pub_from(From)},
               {<<"x-emq-publish-msgid">>, binary, emqx_base62:encode(Id)}]
    payload = Payload

Forwarding MQTT Message Ack Event to RabbitMQ
---------------------------------------------

.. code-block:: javascript

    routing_key = puback
    exchange = emq.acked
    headers = [{<<"x-emq-msg-acked">>, binary, ClientId}],
    payload = emqx_base62:encode(Id)

Example of RabbitMQ Subscription Message Consumption
----------------------------------------------------

Sample code of Rabbit message Consumption in Python:

.. code-block:: javascript

    #!/usr/bin/env python
    import pika
    import sys

    connection = pika.BlockingConnection(pika.ConnectionParameters(host='localhost'))
    channel = connection.channel()

    channel.exchange_declare(exchange='direct:emq.subscription', exchange_type='direct')

    result = channel.queue_declare(exclusive=True)
    queue_name = result.method.queue

    channel.queue_bind(exchange='direct:emq.subscription', queue=queue_name, routing_key= 'subscribe')

    def callback(ch, method, properties, body):
        print(" [x] %r:%r" % (method.routing_key, body))

    channel.basic_consume(callback, queue=queue_name, no_ack=True)

    channel.start_consuming()

Sample of RabbitMQ client coding in other programming languages::

    https://github.com/rabbitmq/rabbitmq-tutorials
    
Enable RabbitMQ Bridge
----------------------

.. code-block:: bash

    ./bin/emqx_ctl plugins load emqx_bridge_rabbit

.. _emqx_bridge:

--------------------
Bridging EMQ X Nodes
--------------------

EMQ X supports bridging between multiple nodes::

                  ---------             ---------
    Publisher --> | EMQ X | --Bridge--> | EMQ X | --> Subscriber
                  ---------             --------- 

Given EMQ nodes emqx1 and emqx2:

+---------+--------------------+
| Name    | Node               |
+---------+--------------------+
| emqx1   | emqx1@192.168.1.10 |
+---------+--------------------+
| emqx2   | emqx2@192.168.1.20 |
+---------+--------------------+

Start nodes emqx1 and emqx2, bridge emqx1 to emqx2, forward all message with topic 'sensor/#' to emqx2:

.. code-block:: bash

    $ ./bin/emqx_ctl bridges start emqx2@192.168.1.20 sensor/#

    bridge is started.

    $ ./bin/emqx_ctl bridges list

    bridge: emqx1@127.0.0.1--sensor/#-->emqx2@127.0.0.1

Test the bridge: emqx1--sensor/#-->emqx2:

.. code-block:: bash

    #on node emqx2

    mosquitto_sub -t sensor/# -p 2883 -d

    #on node emqx1

    mosquitto_pub -t sensor/1/temperature -m "37.5" -d

Delete the bridge:

.. code-block:: bash

    ./bin/emqx_ctl bridges stop emqx2@127.0.0.1 sensor/#

.. _mosquitto_bridge:

----------------
mosquitto Bridge
----------------

Mosquitto can be bridged to EMQ X cluster using common MQTT connection::

                 -------------             -----------------
    Sensor ----> | mosquitto | --Bridge--> |               |
                 -------------             |     EMQ X     |
                 -------------             |    Cluster    |
    Sensor ----> | mosquitto | --Bridge--> |               |
                 -------------             -----------------

An example of mosquitto bridge plugin config file: mosquitto.conf::

    connection emqx
    address 192.168.0.10:1883
    topic sensor/# out 2

    # Set the version of the MQTT protocol to use with for this bridge. Can be one
    # of mqttv31 or mqttv311. Defaults to mqttv31.
    bridge_protocol_version mqttv311

.. _rsmb_bridge:

------------
rsmb Bridge
------------

Rsmb can be bridged to EMQ X cluster using common MQTT connection.

An example of rsmb bridge config file: broker.cfg::

    connection emqx
    addresses 127.0.0.1:2883
    topic sensor/#

