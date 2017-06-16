
.. _deploy:

===========
Deployment
===========

EMQ X Cluster can be deployed as the IoT Hub on private cloud of an enterprise or on public cloud like QingCloud, AliYun or AWS. 

Typical deployment:

.. image:: _static/images/emqx_deploy.png

-------------------
Load Balancer (LB)
-------------------

The Load Balancer (LB) distributes MQTT connections and traffic across the device and the EMQ X cluster. LB enhances the HA of the clustern, balances the loads among the cluster and makes the dynamic expansion possible.

Also, we recommend that the SSL connection terminates at the LB. The links between the devices and the LB are secured by SSL, while the LB and the EMQ X is connected per plain TCP. By this setup, a single EMQ X cluster can serve a million devices.

LB products by public cloud providers:

+---------------+-----------------+----------------------------------------------------+
| Cloud provider| SSL Termination | LB Product DOC/URL                                 |
+===============+=================+====================================================+
| `QingCloud`_  | Y               | https://docs.qingcloud.com/guide/loadbalancer.html |
+---------------+-----------------+----------------------------------------------------+
| `AWS`_        | Y               | https://aws.amazon.com/cn/elasticloadbalancing/    |
+---------------+-----------------+----------------------------------------------------+
| `aliyun`_     | N               | https://www.aliyun.com/product/slb                 |
+---------------+-----------------+----------------------------------------------------+
| `QCloud`_     | N/A             | https://www.qcloud.com/product/clb                 |
+---------------+-----------------+----------------------------------------------------+


LBs for Private Cloud:

+----------------+-----------------+------------------------------------------------------+
| Open-Scource LB| SSL Termination | DOC/URL                                              |
+================+=================+======================================================+
| `HAProxy`_     | Y               | https://www.haproxy.com/solutions/load-balancing.html|
+----------------+-----------------+------------------------------------------------------+
| `NGINX`_       | PLUS Edtion     | https://www.nginx.com/solutions/load-balancing/      |
+----------------+-----------------+------------------------------------------------------+

We recommend AWS with LB for public cloud and HAProxy as LB for priavte cloud deployment.  

--------------
EMQ X Cluster
--------------

EMQ X cluster nodes are deployed behind LB. It is suggested that the nodes are deployed on VPCs or on a private network. Cloud provider -- like QingCloud, AWS, UCloud and QCloud -- usually provides VPC network.

EMQ X Provides the MQTT service on the following TCP ports by default:

+-----------+-----------------------------------+
| 1883      | MQTT                              |
+-----------+-----------------------------------+
| 8883      | MQTT/SSL                          |
+-----------+-----------------------------------+
| 8083      | MQTT/WebSocket                    |
+-----------+-----------------------------------+
| 8084      | MQTT/WebSocket/SSL                |
+-----------+-----------------------------------+

According to the chosen protocol and ports, the firewall should make the relevant ports accessible. 

For the clustering, the following ports of EMQ X node are used:

+-----------+-----------------------------------+
| 4369      | Node discovery port               |
+-----------+-----------------------------------+
| 5369      | Data channel                      |
+-----------+-----------------------------------+
| 6369      | Control channel                   |
+-----------+-----------------------------------+

When firewalls are deployed between nodes, the firewalls should be configured that the above ports are inter-accessible between the nodes.

-----------------------
Deploying on QingCloud
-----------------------

1. Create VPC network.

2. Create a 'private network' for EMQ X cluster inside the VPC network, e.g. 192.168.0.0/24

3. Create 2 EMQ X hosts inside the private network, like:

+-------+-------------+
| emqx1 | 192.168.0.2 |
+-------+-------------+
| emqx2 | 192.168.0.3 |
+-------+-------------+

4. Install and cluster EMQ X on these two hosts. Please refer to the sections of cluster installation for details.
    
5. Create LB and assign the public IP address.

6. Create MQTT TCP listener::


                  -----
                  |   |
                  | L | --TCP 1883--> EMQ X
    --TCP 1883--> |   |
                  | B | --TCP 1883--> EMQ X
                  |   |
                  -----
 
Or create SSL listener and terminate the SSL links on LB::

                  -----
                  |   |
                  | L | --TCP 1883--> EMQ X
    --SSL 8883--> |   |
                  | B | --TCP 1883--> EMQ X
                  |   |
                  -----
  
7. Connect the MQTT clients to the LB using the public IP address and test the deployment.

-----------------
Deploying on AWS
-----------------

1. Create VPC network.

2. Create a 'private network' for EMQ X cluster inside the VPC network, e.g. 192.168.0.0/24

3. Create 2 EMQ X hosts inside the private network, like:

+-------+-------------+
| emqx1 | 192.168.0.2 |
+-------+-------------+
| emqx2 | 192.168.0.3 |
+-------+-------------+

4. Open the TCP ports for MQTT services (e.g. 1883,8883) on the security group. 

5. Install and cluster EMQ X on these two hosts. Please refer to the sections of cluster installation for details.

6. Create ELB (Classic Load Balancer), assign the VPC network, and assign the public IP address.

7. Create MQTT TCP listener on the ELB::

                 -----
                 |   |
                 | E | --TCP 1883--> EMQ X
    --TCP 1883-->| L |
                 | B | --TCP 1883--> EMQ X
                 |   |
                 -----

   Or create SSL listener and terminate the SSL links on the ELB::

                 -----
                 |   |
                 | E | --TCP 1883--> EMQ X
    --SSL 8883-->| L |
                 | B | --TCP 1883--> EMQ X
                 |   |
                 -----

8. Connect the MQTT clients to the ELB using the public IP address and test the deployment.

----------------------------
Deploying on private network
----------------------------

Direct connection of EMQ X cluster
----------------------------------

EMQ X cluster DNS-resolvable and the clients access the cluster via domain name or IP list:

1. Deploy EMQ X cluster. Please refer to the sections of 'program packet installation' and 'EMQ X nodes clustering' for details.

2. On the firewall enable the access to the MQTT ports (e.g. 1883, 8883).

3. Client devices access the EMQ X cluster via domain name or IP list.

.. NOTE:: This kind of deployment is NOT recommended.

HAProxy -> EMQ X
----------------

HAProxy as LB for EMQ X cluster and terminates the SSL connections:

1. Create EMQ X Cluster nodes like following:

+-------+-------------+
| node  | IP          |
+=======+=============+
| emqx1 | 192.168.0.2 |
+-------+-------------+
| emqx2 | 192.168.0.3 |
+-------+-------------+

2. Modify the /etc/haproxy/haproxy.cfg accordingly. 
   An example::

    listen mqtt-ssl
        bind *:8883 ssl crt /etc/ssl/emqx/emqx.pem no-sslv3
        mode tcp
        maxconn 50000
        timeout client 600s
        default_backend emqx_nodes

    backend emqx_nodes
        mode tcp
        balance source
        timeout server 50s
        timeout check 5000
        server emqx1 192.168.0.2:1883 check inter 10000 fall 2 rise 5 weight 1
        server emqx2 192.168.0.3:1883 check inter 10000 fall 2 rise 5 weight 1
        source 0.0.0.0 usesrc clientip

NGINX Plus -> EMQ X
-------------------

NGINX Plus as LB for EMQ X cluster and terminates the SSL links:

1. Install the NGINX Plus. An instruction for Ubuntu: https://cs.nginx.com/repo_setup

2. Create EMQ X cluster nodes like following:

+-------+-------------+
| node  | IP          |
+=======+=============+
| emqx1 | 192.168.0.2 |
+-------+-------------+
| emqx2 | 192.168.0.3 |
+-------+-------------+

3. Modify the /etc/nginx/nginx.conf.
   An example::

    stream {
        # Example configuration for TCP load balancing

        upstream stream_backend {
            zone tcp_servers 64k;
            hash $remote_addr;
            server 192.168.0.2:1883 max_fails=2 fail_timeout=30s;
            server 192.168.0.3:1883 max_fails=2 fail_timeout=30s;
        }

        server {
            listen 8883 ssl;
            status_zone tcp_server;
            proxy_pass stream_backend;
            proxy_buffer_size 4k;
            ssl_handshake_timeout 15s;
            ssl_certificate     /etc/emqx/certs/cert.pem;
            ssl_certificate_key /etc/emqx/certs/key.pem;
        }
    }

=====================
Installation
=====================

-------------------
System Requirements
-------------------

Operating System
----------------

EMQ X is developed utilizing the Erlang/OTP language / platform. It runs on the following OS: Linux, FreeBSD, MAC OS X and Windows Server.

We recommend the 64-bit Linux-based cloud host or servr for the deployment.

CPU/MEM
--------

In the test scenario, EMQ X with 1G memory sustains 80K TCP links or 15K SSL links.  

In production environment, it is suggested to deploy at least 2 nodes in the cluster. Planning the CPU and Momery capacity on the basic of concurrent connections and the message throughput.

---------------------------------
Naming Rule of Software Package
---------------------------------

For every EMQ X release, it is distributed as software packages for Ubuntu, CentOs, FreeBSD, Mac OS X and windows. Besides, an image for Docker is also released. 

Please contact us for the software package: http://emqtt.com/about#contacts

The package name consists of the platform name and the version number. E.g. emqx-enterprise-centos7-v2.1.0.zip

.. _install_rpm:

-----------------
RPM Package
-----------------

RPM is recommended for CentOS and RedHat. After installation, EMQ X service is managed by the OS. 

Installation
------------

.. code-block:: console

    rpm -ivh --force emqx-centos6.8-v2.1.0-1.el6.x86_64.rpm

.. NOTE:: Erlang/OTP R19 depends on lksctp-tools

.. code-block:: console

    yum install lksctp-tools

Config Files
------------

EMQ X config file: /etc/emqx/emqx.conf, config file for plugins: /etc/emqx/plugins/\*.conf

Log Files
----------

Log files directory: /var/log/emqx

Data Files
----------

Data files derectory: /var/lib/emqx/

Start/Stop
----------

.. code-block:: console

    service emqx start|stop|restart

.. _install_deb:

----------------
DEB package
----------------

DEB is recommended for Debian and Ubuntu. After installation, EMQ X service is managed bu the OS.

.. code-block:: console

    sudo dpkg -i emqx-ubuntu16.04_v2.1.0_amd64.deb

.. NOTE:: Erlang/OTP R19 depends on 'lksctp-tools' lib

.. code-block:: console

    apt-get install lksctp-tools

Config Files
------------

EMQ X config file: /etc/emqx/emqx.conf, plugins config file: /etc/emqx/plugins/\*.conf。

Log Files
----------

Log files directory: /var/log.emqx

Data Files
-----------

Data files directory: /var/lib/emqx/

Start/Stop
----------

.. code-block:: console

    service emqx start|stop|restart

.. _install_on_linux:

---------------------------
EMQ X Packages for Linux
---------------------------

EMQ X Linux General Packages:

+---------------------+------------------------------------------+
|  OS                 |           Software Package               |
+=====================+==========================================+
| CentOS6(64-bit)     | emqx-enterprise-centos6.8-v2.1.0.zip     |
+---------------------+------------------------------------------+
| CentOS7(64-bit)     | emqx-enterprise-centos7-v2.1.0.zip       |
+---------------------+------------------------------------------+
| Ubuntu16.04(64-bit) | emqx-enterprise-ubuntu16.04-v2.1.0.zip   |
+---------------------+------------------------------------------+
| Ubuntu14.04(64-bit) | emqx-enterprise-ubuntu14.04-v2.1.0.zip   |
+---------------------+------------------------------------------+
| Ubuntu12.04(64-bit) | emqx-enterprise-ubuntu12.04-v2.1.0.zip   |
+---------------------+------------------------------------------+
| Debian7(64-bit)     | emqx-enterprise-debian7-v2.1.0.zip       |
+---------------------+------------------------------------------+
| Debian8(64-bit)     | emqx-enterprise-debian8-v2.1.0.zip       |
+---------------------+------------------------------------------+

Following is a demonstration of installing EMQ X on CentOS: 

.. code-block:: bash

    unzip emqx-enterprise-centos7-v2.1.0.zip

Use the console mode to check if EMQ X starts normal:

.. code-block:: bash

    cd emqx && ./bin/emqx console

If EMQ X start normal, the output of console shall looks like:

.. code-block:: bash

    Starting emqx on node emqx@127.0.0.1
    Load emqx_mod_presence module successfully.
    Load emqx_mod_subscription module successfully.
    dashboard:http listen on 0.0.0.0:18083 with 2 acceptors.
    mqtt:tcp listen on 127.0.0.1:11883 with 4 acceptors.
    mqtt:tcp listen on 0.0.0.0:1883 with 8 acceptors.
    mqtt:ws listen on 0.0.0.0:8083 with 4 acceptors.
    mqtt:ssl listen on 0.0.0.0:8883 with 4 acceptors.
    mqtt:wss listen on 0.0.0.0:8084 with 4 acceptors.
    emqx 2.1.0 is running now!

CTRL+C to close console, start EMQ X as daemon:

.. code-block:: bash

    ./bin/emqx start

Log files can be find under the log/ directory.

Check the EMQ X service's status:

.. code-block:: bash

    ./bin/emqx_ctl status

If EMQ X starts normally and runs correctly, status check shall return as following:

.. code-block:: bash

    $ ./bin/emqx_ctl status
    Node 'emqx@127.0.0.1' is started
    emqx 2.1.0 is running

the status of EMQ X server can also be monitored on the following URL:

    http://localhost:8083/status

Stop the server::

    ./bin/emqx stop

.. _install_on_freebsd:

---------------------
Installing on FreeBSD
---------------------

Please contact us for the software package: http://emqtt.com/about#contacts

Installingon FreeBSD is the same as which on Linux.

.. _install_on_mac:

----------------------
Installing on Mac OS X
----------------------

The to install and start EMQ X on Mac OS X is the same as which of on Linux.

When developing MQTT applications on Mac, modify the 'etc/emqx.conf' file as following to check the MQTT massages on the console: 

.. code-block:: properties

    ## Console log. Enum: off, file, console, both
    log.console = both

    ## Console log level. Enum: debug, info, notice, warning, error, critical, alert, emergency
    log.console.level = debug

    ## Console log file
    log.console.file = log/console.log

.. _install_docker:

---------------------------
Installing the Docker Image
---------------------------

Please contact us to get the docker image: http://emqtt.com/about#contacts

Unzip the emqx-enterprise-docker package::

    unzip emqx-enterprise-docker-v2.1.0.zip

Load the Image::

    docker load < emqx-enterprise-docker-v2.1.0

Run the container::

    docker run -itd --net='host' --name emqx20 emqx-enterprise-docker-v2.1.0

Stop the brocker::

    docker stop emqx20

Start the brocker::

    docker start emqx20

Enter the running container:

    docker exec -it emqx20 /bin/bash

===========
Quick Setup
===========

Assuming a EMQ X Cluster with two Linux nodes deplyed on cloud VPC network or private network:

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

Deloyed under Linux, EMQ X sustains 100 concurrent connections. To achieve this, the system Kernel, Networking, the Erlang VM and EMQ X itself must be tuned.

System-Wide File Handles
------------------------

Maximun file handels:

.. code-block:: console

    # 2 millions system-wide
    sysctl -w fs.file-max=262144
    sysctl -w fs.nr_open=262144
    echo 262144 > /proc/sys/fs/nr_open

Maximum of file handels for current session:

.. code-block:: console

    ulimit -n 262144

/etc/sysctl.conf
----------------

Add 'fs.file-max' to '/etc/sysctl.conf' and make the changes permanent::

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

------------------
Start EMQ X Nodes
------------------

If EMQ X is installed using RPM or DEB::

    service emqx start

if EMQ X is installed using zip package::

    ./bin/emqx start

----------------------------
Clustering the EMQ X Nodes
----------------------------

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

'emxq-dashboard' plugin starts the web management and provides the management service on port 18083.

Web console URL: http://localhost:18083/, default user-name: admin, password: public.

Through the web console, the status of cluster nodes, statistic of MQTT message, MQTT clients, MQTT sessions and routing informations can be inquired.

.. _tcp_ports:

-------------------------
TCP Ports of MQTT Service
-------------------------

By default, EMQ X starts following service on these ports:

+-----------+-----------------------------------+
| 1883      | MQTT                              |
+-----------+-----------------------------------+
| 8883      | MQTT/SSL                          |
+-----------+-----------------------------------+
| 8083      | MQTT/WebSocket                    |
+-----------+-----------------------------------+
| 8084      | MQTT/WebSocket(SSL)               |
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

By Commenting out or deleting the above config, the related TCP services are disabled.

-----------------------
TCP Port for Clustering
-----------------------

The firewalls must allow the nodes access each other on the following ports:

+-----------+-----------------------------------+
| 4369      | Node discovery port               |
+-----------+-----------------------------------+
| 5369      | Data channel                      |
+-----------+-----------------------------------+
| 6369      | Control channel                   |
+-----------+-----------------------------------+

.. _qingcloud:  https://qingcloud.com
.. _AWS:        https://aws.amazon.com
.. _aliyun:     https://www.aliyun.com
.. _UCloud:     https://ucloud.cn
.. _QCloud:     https://www.qcloud.com
.. _HAProxy:    https://www.haproxy.org
.. _NGINX:      https://www.nginx.com 

