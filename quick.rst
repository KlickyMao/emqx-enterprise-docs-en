
.. _quicksetup:

===========
Quick Setup
===========

Suppose a EMQ X Cluster with two Linux nodes deployed on a cloud VPC network or a private network:

+---------------------+---------------------+
| Node name           |    IP               |
+---------------------+---------------------+
| emqx1@192.168.0.10  | 192.168.0.10        |
+---------------------+---------------------+
| emqx@192.168.0.20   | 192.168.0.20        |
+---------------------+---------------------+

-----------------
System Parameters
-----------------

Deployed under Linux, EMQ X allows 100k concurrent connections by default. To achieve this, the system Kernel, Networking, the Erlang VM and EMQ X itself must be tuned.

System-Wide File Handles
------------------------

Maximun file handles:

.. code-block:: console

    # 2 millions system-wide
    sysctl -w fs.file-max=262144
    sysctl -w fs.nr_open=262144
    echo 262144 > /proc/sys/fs/nr_open

Maximum of file handles for current session:

.. code-block:: console

    ulimit -n 262144

/etc/sysctl.conf
----------------

Add 'fs.file-max' to '/etc/sysctl.conf' and make the changes permanent:

.. code-block:: console

    fs.file-max = 262144

/etc/security/limits.conf
-------------------------

Persist the maximum number of opened file handles for users in /etc/security/limits.conf::

    emqx      soft   nofile      262144
    emqx      hard   nofile      262144

Note: Under Ubuntu, '/etc/systemd/system.conf' is to be modified:

.. code-block:: properties

    DefaultLimitNOFILE=262144

---------------
EMQ X Node Name
---------------

Set the node name and cookies(communicating between nodes)

'/etc/emqx/emqx.conf' on emqx1::

    node.name   = emqx1@192.168.0.10
    node.cookie = secret_dist_cookie

'/etc/emqx/emqx.conf' on emqx2::

    node.name   = emqx2@192.168.0.20
    node.cookie = secret_dist_cookie

-----------------
Start EMQ X Nodes
-----------------

If EMQ X is installed using RPM or DEB::

    service emqx start

if EMQ X is installed using zip package::

    ./bin/emqx start

--------------------------
Clustering the EMQ X Nodes
--------------------------

Start the two nodes, on the emqx1@192.168.0.10 run:: 

    $ ./bin/emqx_ctl cluster join emqx2@192.168.0.20

    Join the cluster successfully.
    Cluster status: [{running_nodes,['emqx1@192.168.0.10','emqx@192.168.0.20']}]

or, on the emqx1@192.168.0.20 run::

    $ ./bin/emqx_ctl cluster join emqx1@192.168.0.10

    Join the cluster successfully.
    Cluster status: [{running_nodes,['emqx1@192.168.0.10','emqx@192.168.0.20']}]

Check the cluster status on any node::

    $ ./bin/emqx_ctl cluster status

    Cluster status: [{running_nodes,['emqx1@192.168.0.10','emqx@192.168.0.20']}]

-----------------------------
Managing utlizing Web Console
-----------------------------

'emqx-dashboard' plugin starts the web management and provides the management service on port 18083.

Web console URL: http://localhost:18083/, default user-name: admin, password: public.

Through the web console, the status of cluster nodes, statistic of MQTT message, MQTT clients, MQTT sessions and routing informations can be inquired.

.. _tcp_ports:

-------------------------
TCP Ports of MQTT Service
-------------------------

By default, EMQ X starts following services on these ports:

+-----------+-----------------------------------+
| 1883      | MQTT                              |
+-----------+-----------------------------------+
| 8883      | MQTT/SSL                          |
+-----------+-----------------------------------+
| 8083      | MQTT/WebSocket                    |
+-----------+-----------------------------------+
| 8084      | MQTT/WebSocket/SSL                |
+-----------+-----------------------------------+
| 18083     | Web Management Console            |
+-----------+-----------------------------------+

The ports can be configured in the 'Listeners' section of the file 'etc/emqx.conf':

.. code-block:: properties

    ## External TCP Listener: 1883, 127.0.0.1:1883, ::1:1883
    listener.tcp.external = 0.0.0.0:1883

    ## SSL Listener: 8883, 127.0.0.1:8883, ::1:8883
    listener.ssl.external = 8883
    
    ## HTTP and WebSocket Listener
    listener.http.external = 8083

    ## External HTTPS and WSS Listener
    listener.https.external = 8084

By commenting out or deleting the above config, the related TCP services are disabled.

-----------------------
TCP Port for Clustering
-----------------------

The firewalls must allow the nodes access each other on the following ports:

+-----------+-----------------------------------+
| 4369      | Node discovery port               |
+-----------+-----------------------------------+
| 5369      | Data channel                      |
+-----------+-----------------------------------+
| 6369      | Cluster channel                   |
+-----------+-----------------------------------+

