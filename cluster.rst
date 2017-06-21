
.. _cluster:

===========
Clustering
===========

.. _cluster_erlang:

----------------------
Distributed Erlang/OTP
----------------------

Erlang/OTP is a concurrent, fault-tolerant, distributed programming platform. A distributed Erlang/OTP system consists of a number of Erlang runtime systems called ‘node’. Nodes connect to each other with TCP/IP sockets and communicate by Message Passing::


    ---------         ---------
    | Node1 | --------| Node2 |
    ---------         ---------
        |     \     /    |
        |       \ /      |
        |       / \      |
        |     /     \    |
    ---------         ---------
    | Node3 | --------| Node4 |
    ---------         ---------


Node
----------

An erlang runtime system called ‘node’ is identified by a unique name like email addreass. Erlang nodes communicate with each other by the name.

Suppose we start four Erlang nodes on localhost:

.. code:: shell

    erl -name node1@127.0.0.1
    erl -name node2@127.0.0.1
    erl -name node3@127.0.0.1
    erl -name node4@127.0.0.1

Connecting node1@127.0.0.1 to the other nodes::

    (node1@127.0.0.1)1> net_kernel:connect_node('node2@127.0.0.1').
    true
    (node1@127.0.0.1)2> net_kernel:connect_node('node3@127.0.0.1').
    true
    (node1@127.0.0.1)3> net_kernel:connect_node('node4@127.0.0.1').
    true
    (node1@127.0.0.1)4> nodes().
    ['node2@127.0.0.1','node3@127.0.0.1','node4@127.0.0.1']

epmd
----

epmd(Erlang Port Mapper Daemon) is a daemon service that is responsible for mapping node names to machine addresses(TCP sockets). The daemon is started automatically on every host where an Erlang node started::

    (node1@127.0.0.1)6> net_adm:names().
    {ok,[{"node1",62740},
         {"node2",62746},
         {"node3",62877},
         {"node4",62895}]}

Cookie
-------

Erlang nodes authenticate each other by a magic cookie when communicating. 

The cookie could be configured by::

    1. $HOME/.erlang.cookie文件

    2. erl -setcookie <Cookie>

.. note: 本节内容来自: http://erlang.org/doc/reference_manual/distributed.html

.. _cluster_emqx:

-----------------
EMQ X分布集群设计
-----------------

EMQ X集群基于Erlang/OTP分布式设计，集群原理可简述为下述两条规则:

1. MQTT客户端订阅主题时，所在节点订阅成功后广播通知其他节点：某个主题(Topic)被本节点订阅。

2. MQTT客户端发布消息时，所在节点会根据消息主题(Topic)，检索订阅并路由消息到相关节点。

EMQ X同一集群的所有节点，都会复制一份主题(Topic) -> 节点(Node)映射的路由表，例如::

    topic1 -> node1, node2
    topic2 -> node3
    topic3 -> node2, node4

主题树(Topic Trie)与路由表(Route Table)
---------------------------------------

EMQ X同一集群内每个节点，都保存一份主题树(Topic Trie)和路由表。

例如下述主题订阅关系:

+----------------+-------------+----------------------------+
| 客户端         | 节点        |  订阅主题                  |
+----------------+-------------+----------------------------+
| client1        | node1       | t/+/x, t/+/y               |
+----------------+-------------+----------------------------+
| client2        | node2       | t/#                        |
+----------------+-------------+----------------------------+
| client3        | node3       | t/+/x, t/a                 |
+----------------+-------------+----------------------------+

最终会生成如下主题树(Topic Trie)和路由表(Route Table)::

    --------------------------
    |             t          |
    |            / \         |
    |           +   #        |
    |         /  \           |
    |       x      y         |
    --------------------------
    | t/+/x -> node1, node3  |
    | t/+/y -> node1         |
    | t/#   -> node2         |
    | t/a   -> node3         |
    --------------------------

订阅(Subscription)与消息派发
----------------------------

客户端的主题订阅(Subscription)关系，只保存在客户端所在节点，用于本节点内派发消息到客户端。

例如client1向主题't/a'发布消息，消息在节点间的路由与派发流程::

    title: Message Route and Deliver

    client1->node1: Publish[t/a]
    node1-->node2: Route[t/#]
    node1-->node3: Route[t/a]
    node2-->client2: Deliver[t/#]
    node3-->client3: Deliver[t/a]

.. image:: ./_static/images/route.png

-----------------
EMQ X集群配置管理
-----------------

假设部署两台服务器s1.emqtt.io, s2.emqtt.io上部署集群:

+----------------------+-----------------+---------------------+
| 节点名               | 主机名(FQDN)    |    IP地址           |
+----------------------+-----------------+---------------------+
| emqx@s1.emqtt.io 或  | s1.emqtt.io     | 192.168.0.10        |
| emqx@192.168.0.10    |                 |                     |
+----------------------+-----------------+---------------------+
| emqx@s2.emqtt.io 或  | s2.emqtt.io     | 192.168.0.20        |
| emqx@192.168.0.20    |                 |                     |
+----------------------+-----------------+---------------------+

.. WARNING:: 节点名格式: Name@Host, Host必须是IP地址或FQDN(主机名.域名)

emqx@s1.emqtt.io节点设置
------------------------

.. code-block:: properties

    node.name = emq@s1.emqtt.io

    或

    node.name = emq@192.168.0.10

也可通过环境变量::

    export EMQX_NODE_NAME=emqx@s1.emqtt.io && ./bin/emqx start

.. WARNING:: 节点启动加入集群后，节点名称不能变更。

emqx@s2.emqtt.io节点设置
------------------------

.. code-block:: properties

    node.name = emq@s2.emqtt.io

    或

    node.name = emq@192.168.0.20

节点加入集群
------------

启动两台节点后，emqx@s2.emqtt.io上执行::

    $ ./bin/emqx_ctl cluster join emqx@s1.emqtt.io

    Join the cluster successfully.
    Cluster status: [{running_nodes,['emqx@s1.emqtt.io','emqx@s2.emqtt.io']}]

或，emqx@s1.emqtt.io上执行::

    $ ./bin/emqx_ctl cluster join emqx@s2.emqtt.io

    Join the cluster successfully.
    Cluster status: [{running_nodes,['emqx@s1.emqtt.io','emqx@s2.emqtt.io']}]

任意节点上查询集群状态::

    $ ./bin/emqx_ctl cluster status

    Cluster status: [{running_nodes,['emqx@s1.emqtt.io','emqx@s2.emqtt.io']}]

节点退出集群
------------

节点退出集群，两种方式:

1. leave: 本节点退出集群

2. remove: 从集群删除其他节点

emqx@s2.emqtt.io主动退出集群::

    $ ./bin/emqx_ctl cluster leave

或emqx@s1.emqtt.io节点上，从集群删除emqx@s2.emqtt.io节点::

    $ ./bin/emqx_ctl cluster remove emqx@s2.emqtt.io

.. _cluster_session:

-------------------
跨节点会话(Session)
-------------------

EMQ X消息服务器集群模式下，MQTT连接的持久会话(Session)跨节点。

例如负载均衡的两台集群节点:node1与node2，同一MQTT客户端先连接node1，node1节点会创建持久会话；客户端断线重连到node2时，MQTT的连接在node2节点，持久会话仍在node1节点::

                                      node1
                                   -----------
                               |-->| session |
                               |   -----------
                 node2         |
              --------------   |
     client-->| connection |<--|
              --------------

.. _cluster_firewall:

----------
防火墙设置
----------

如果集群节点间存在防火墙，防火墙需要开启4369端口、5369端口和一个TCP端口段。4369由epmd端口映射服务使用，5369用于节点间数据通信，TCP端口段用于节点间集群通信。

默认节点间集群默认需要开启的端口:

+--------------+-----------------------+
| 端口         | 用途                  |
+--------------+-----------------------+
| 4369         | epmd端口映射服务      | 
+--------------+-----------------------+
| 5369         | 节点间数据通道        | 
+--------------+-----------------------+
| 6369         | 节点间集群通道        | 
+--------------+-----------------------+

防火墙设置后，emqx.conf需要配置相同的端口段:

.. code-block:: properties

    ## Distributed node port range
    node.dist_listen_min = 6369
    node.dist_listen_max = 6369

.. _cluster_netsplit:

------------------
注意事项: NetSplit
------------------

EMQ X集群需要稳定网络连接以避免发生NetSplit故障。集群设计上默认不自动处理NetSplit，如集群节点间发生NetSplit，需手工重启某个分片上的相关节点。

.. NOTE:: NetSplit是指节点运行正常但因网络断开互相认为对方宕机。EMQ 2.2版本将支持NetSplit自动恢复。

.. _cluster_hash:

---------------
一致性Hash与DHT
---------------

NoSQL数据库领域分布式设计，大多会采用一致性Hash或DHT。EMQ消息服务器集群架构可支持千万级的路由，更大级别的集群可采用一致性Hash、DHT或Shard方式切分路由表。

