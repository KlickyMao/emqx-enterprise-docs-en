
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

    1. $HOME/.erlang.cookie

    2. erl -setcookie <Cookie>

.. NOTE:: Some content in this section is from: http://erlang.org/doc/reference_manual/distributed.html

.. _cluster_emqx:

------------------------
Cluster Design of EMQ X
------------------------

The cluster architecture of EMQ X is based on distrubuted Erlang/OTP and Mnesia database.

The cluster design could be summarized by the following two rules::

    When a MQTT client SUBSCRIBE a Topic on a node, the node will tell all the other nodes in the cluster: I subscribed a Topic.
    When a MQTT Client PUBLISH a message to a node, the node will lookup the Topic table and forward the message to nodes that subscribed the Topic.

Finally there will be a global route table(Topic -> Node) that replicated to all nodes in the cluster::

    topic1 -> node1, node2
    topic2 -> node3
    topic3 -> node2, node4

Topic Trie and Route Table
---------------------------------------

Every node in the cluster will store a topic trie and route table in mnesia database.

Suppose that we create subscriptions:

+----------------+-------------+----------------------------+
| Client         | Node        |  Topics                    |
+----------------+-------------+----------------------------+
| client1        | node1       | t/+/x, t/+/y               |
+----------------+-------------+----------------------------+
| client2        | node2       | t/#                        |
+----------------+-------------+----------------------------+
| client3        | node3       | t/+/x, t/a                 |
+----------------+-------------+----------------------------+

Finally the topic trie and route table in the cluster::

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

Message Route and Deliver
----------------------------

The brokers in the cluster route messages by topic trie and route table, deliver messages to MQTT clients by subscriptions. Subscriptions are mapping from topic to subscribers, are stored only in the local node, will not be replicated to other nodes.

Suppose client1 PUBLISH a message to the topic ‘t/a’, the message Route and Deliver process::

    title: Message Route and Deliver

    client1->node1: Publish[t/a]
    node1-->node2: Route[t/#]
    node1-->node3: Route[t/a]
    node2-->client2: Deliver[t/#]
    node3-->client3: Deliver[t/a]

.. image:: ./_static/images/route.png

--------------------
EMQ X Cluster Setup 
--------------------

Suppose we deploy two nodes cluster on s1.emqtt.io, s2.emqtt.io:

+----------------------+-----------------+---------------------+
| Node                 | Host (FQDN)     |    IP               |
+----------------------+-----------------+---------------------+
| emqx@s1.emqtt.io or  | s1.emqtt.io     | 192.168.0.10        |
| emqx@192.168.0.10    |                 |                     |
+----------------------+-----------------+---------------------+
| emqx@s2.emqtt.io or  | s2.emqtt.io     | 192.168.0.20        |
| emqx@192.168.0.20    |                 |                     |
+----------------------+-----------------+---------------------+

.. WARNING:: The node name is Name@Host, where Host is IP address or the fully qualified host name.

emqx@s1.emqtt.io Config
------------------------

.. code-block:: properties

    node.name = emq@s1.emqtt.io

    or

    node.name = emq@192.168.0.10

Or using the environment variable:: 

    export EMQX_NODE_NAME=emqx@s1.emqtt.io && ./bin/emqx start

.. WARNING:: The name cannot be changed after node joined the cluster.

emqx@s2.emqtt.io Config
------------------------

.. code-block:: properties

    node.name = emq@s2.emqtt.io

    or

    node.name = emq@192.168.0.20

Join the Cluter
----------------

Start the two broker nodes, and execute ‘cluster join‘ on emqttd@s2.emqtt.io::

    $ ./bin/emqx_ctl cluster join emqx@s1.emqtt.io

    Join the cluster successfully.
    Cluster status: [{running_nodes,['emqx@s1.emqtt.io','emqx@s2.emqtt.io']}]

or, execute 'cluster join' on emqx@s1.emqtt.io::

    $ ./bin/emqx_ctl cluster join emqx@s2.emqtt.io

    Join the cluster successfully.
    Cluster status: [{running_nodes,['emqx@s1.emqtt.io','emqx@s2.emqtt.io']}]

Query the cluster status::

    $ ./bin/emqx_ctl cluster status

    Cluster status: [{running_nodes,['emqx@s1.emqtt.io','emqx@s2.emqtt.io']}]

Leave the Cluster
-------------------

Two ways to leave the cluster:

1.  leave: this node leaves the cluster

2.  remove: remove other nodes from the cluster

emqx@s2.emqtt.io tries to leave the cluster::

    $ ./bin/emqx_ctl cluster leave

 Or remove emqttd@s2.emqtt.io node from the cluster on emqttd@s1.emqtt.io::

    $ ./bin/emqx_ctl cluster remove emqx@s2.emqtt.io

.. _cluster_session:

--------------------
Session across Nodes
--------------------

The persistent MQTT sessions (clean session = false) are across nodes in the EMQ X cluster.


.. 例如负载均衡的两台集群节点:node1与node2，同一MQTT客户端先连接node1，node1节点会创建持久会话；客户端断线重连到node2时，MQTT的连接在node2节点，持久会话仍在node1节点::

Consider two load-balanced nodes in a cluster: node1 and node2. A MQTT client connects to node1 at the first place, node1 creates persistent session for the client, and then disconnects from node1. Later when this client tries connect to node2, the connection is then created on node2, but the persistent session will be still on where is was (in this case node1)::

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
Firewalls
----------

If there are firewalls between the nodes, the 4369 port, 5369 port and a TCP port range shall be made available. The 4369 is for epmd port mapping and the 5369 is used for nodes' data communication and the tcp port range is for nodes' clustering communication. 

Ports shall be made available on firewall:

+--------------+----------------------------------+
| Port         | Usage                            |
+--------------+----------------------------------+
| 4369         | epmd port mapping                | 
+--------------+----------------------------------+
| 5369         | Nodes' data communication        | 
+--------------+----------------------------------+
| 6369         | Nodes's clustering communication | 
+--------------+----------------------------------+

Modify the 'emqx.conf' in line with the firewall configuration:

.. code-block:: properties

    ## Distributed node port range
    node.dist_listen_min = 6369
    node.dist_listen_max = 6369

.. _cluster_netsplit:

------------------
Network Partitions
------------------

EMQ X cluster requires reliable network to avoid network partition. The cluster will not recover from a network partition automatically. If network partition occurs, manual intervention is expected.

.. NOTE:: Network partition means the nodes works fine but they can't reach each other (due to network failure) and thus consider the communication partner is down. EMQ X 2.2 will support Network partition automatic recovery.

.. _cluster_hash:

-----------------------
Consistent Hash and DHT
-----------------------

Consistent Hash and DHT are popular in the design of NoSQL databases. Cluster of emqttd broker could support 10 million size of global routing table now. We could use the Consistent Hash or DHT to partition the routing table, and evolve the cluster to larger size.

