
.. _deploy:

===========
Deployment
===========
..  部署架构

EMQ X Cluster can be deployed as the IoT Hub on private cloud of an enterprise or on public cloud like QingCloud, AliYun or AWS. 

.. EMQ X集群作为物联网接入服务(IoT Hub)，部署在青云、AWS、阿里等公有云或企业私有云平台。

Typical deployment:

.. 典型部署结构:

.. image:: _static/images/emqx_deploy.png

-------------------
Load Balancer (LB)
-------------------
.. LB(负载均衡)

The Load Balancer (LB) distributes MQTT connections and traffic across the device and the EMQ X cluster. LB enhances the HA of the clustern, balances the loads among the cluster and makes the dynamic expansion possible.

.. LB(负载均衡器)负责分发设备的MQTT连接与消息到EMQ X集群，LB提高EMQ X集群可用性、实现负载平衡以及动态扩容。

Also, we recommend that the SSL connection terminates at the LB. The links between the devices and the LB are secured by SSL, while the LB and the EMQ X is connected per plain TCP. By this setup, a single EMQ X cluster can serve a million devices.

.. 部署架构推荐在LB终结SSL连接。设备与LB之间SSL安全连接，LB与EMQ X之间TCP连接。这种部署模式下EMQ X单集群可轻松支持100万设备。

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
| `UCloud`_     | N/A             | https://ucloud.cn/site/product/ulb.html            |
+---------------+-----------------+----------------------------------------------------+
| `QCloud`_     | N/A             | https://www.qcloud.com/product/clb                 |
+---------------+-----------------+----------------------------------------------------+


.. 公有云厂商LB产品:
.. 
  +---------------+-----------------+----------------------------------------------------+
  | 云计算厂商    | 是否支持SSL终结 | LB产品介绍                                         |
  +===============+=================+====================================================+
  | `青云`_       | 是              | https://docs.qingcloud.com/guide/loadbalancer.html |
  +---------------+-----------------+----------------------------------------------------+
  | `AWS`_        | 是              | https://aws.amazon.com/cn/elasticloadbalancing/    |
  +---------------+-----------------+----------------------------------------------------+
  | `阿里云`_     | 否              | https://www.aliyun.com/product/slb                 |
  +---------------+-----------------+----------------------------------------------------+
  | `UCloud`_     | 未知            | https://ucloud.cn/site/product/ulb.html            |
  +---------------+-----------------+----------------------------------------------------+
  | `QCloud`_     | 未知            | https://www.qcloud.com/product/clb                 |
  +---------------+-----------------+----------------------------------------------------+
  
LBs for Private Cloud:

.. 私有部署LB服务器:

+----------------+-----------------+------------------------------------------------------+
| Open-Scource LB| SSL Termination | DOC/URL                                              |
+================+=================+======================================================+
| `HAProxy`_     | Y               | https://www.haproxy.com/solutions/load-balancing.html|
+----------------+-----------------+------------------------------------------------------+
| `NGINX`_       | PLUS Edtion     | https://www.nginx.com/solutions/load-balancing/      |
+----------------+-----------------+------------------------------------------------------+

..
  +---------------+-----------------+------------------------------------------------------+
  | 开源LB        | 是否支持SSL终结 | 方案介绍                                             |
  +===============+=================+======================================================+
  | `HAProxy`_    | 是              | https://www.haproxy.com/solutions/load-balancing.html|
  +---------------+-----------------+------------------------------------------------------+
  | `NGINX`_      | PLUS产品支持    | https://www.nginx.com/solutions/load-balancing/      |
  +---------------+-----------------+------------------------------------------------------+

We recommend AWS with LB for public cloud and HAProxy as LB for priavte cloud deployment.  

.. 国内公有云部署推荐青云(EMQ合作伙伴)，国外部署推荐AWS。私有部署推荐使用HAPRoxy作为LB。

--------------
EMQ X Cluster
--------------

.. EMQ X集群

EMQ X cluster nodes are deployed behind LB. It is suggested that the nodes are deployed on VPCs or on a private network. Cloud provider -- like QingCloud, AWS, UCloud and QCloud -- usually provides VPC network.

.. EMQ X节点集群部署在LB之后，建议部署在VPC或私有网络内。公有云厂商青云、AWS、UCloud、QCloud均支持VPC网络。

EMQ X Provides the MQTT service on the following TCP ports by default:

.. EMQ X默认开启的MQTT服务TCP端口:

+-----------+-----------------------------------+
| 1883      | MQTT                              |
+-----------+-----------------------------------+
| 8883      | MQTT/SSL                          |
+-----------+-----------------------------------+
| 8083      | MQTT/WebSocket                    |
+-----------+-----------------------------------+
| 8084      | MQTT/WebSocket(SSL)               |
+-----------+-----------------------------------+

..
  +-----------+-----------------------------------+
  | 1883      | MQTT协议端口                      |
  +-----------+-----------------------------------+
  | 8883      | MQTT/SSL端口                      |
  +-----------+-----------------------------------+
  | 8083      | MQTT/WebSocket端口                |
  +-----------+-----------------------------------+
  | 8084      | MQTT/WebSocket(SSL)端口           |
  +-----------+-----------------------------------+
  
According to the chosen protocol and ports, the firewall should make the relevant ports accessible. 

.. 防火墙根据使用的MQTT接入方式，开启上述端口的访问权限。

For the clustering, the following ports of EMQ X node are used:

.. EMQ X节点集群使用的TCP端口:

+-----------+-----------------------------------+
| 4369      | Node discovery port               |
+-----------+-----------------------------------+
| 5369      | Data channel                      |
+-----------+-----------------------------------+
| 6369      | Control channel                   |
+-----------+-----------------------------------+

..
  +-----------+-----------------------------------+
  | 4369      | 集群节点发现端口                  |
  +-----------+-----------------------------------+
  | 5369      | 集群节点数据通道                  |
  +-----------+-----------------------------------+
  | 6369      | 集群节点控制通道                  |
  +-----------+-----------------------------------+
  
When firewalls are deployed between nodes, the firewalls should be configured that the above ports are inter-accessible between the nodes.

.. 集群节点间如有防护墙，需开启上述TCP端口互访权限。

-----------------------
Deploying on QingCloud
-----------------------

.. 青云(QingCloud)部署

1. Create VPC network.

.. 1. 创建VPC网络。

2. Create a 'private network' for EMQ X cluster inside the VPC network, e.g. 192.168.0.0/24

.. 2. VPC网络内创建EMQ X集群'私有网络'，例如: 192.168.0.0/24

3. Create 2 EMQ X hosts inside the private network, like:

.. 3. 私有网络内创建两台EMQ X主机，例如:

+-------+-------------+
| emqx1 | 192.168.0.2 |
+-------+-------------+
| emqx2 | 192.168.0.3 |
+-------+-------------+

4. Install and cluster EMQ X on these two hosts. Please refer to the sections of cluster installation for details.
    
.. 4. 安装并集群EMQ X主机，具体配置请参考安装集群章节。

5. Create LB and assign the public IP address.

.. 5. 创建LB(负载均衡器)并指定公网IP地址。

6. Create MQTT TCP listener::


                  -----
                  |   |
                  | L | --TCP 1883--> EMQ X
    --TCP 1883--> |   |
                  | B | --TCP 1883--> EMQ X
                  |   |
                  -----
 
.. 6. 在LB上创建MQTT TCP监听器::

Or create SSL listener and terminate the SSL links on LB::

                  -----
                  |   |
                  | L | --TCP 1883--> EMQ X
    --SSL 8883--> |   |
                  | B | --TCP 1883--> EMQ X
                  |   |
                  -----
  
..   或创建SSL监听器，并终结SSL在LB::

7. Connect the MQTT clients to the LB using the public IP address and test the deployment.

.. 7. MQTT客户端连接LB公网地址测试。

-----------------
Deploying on AWS
-----------------

.. 亚马逊(AWS)部署

1. Create VPC network.

.. 1. 创建VPC网络。

2. Create a 'private network' for EMQ X cluster inside the VPC network, e.g. 192.168.0.0/24

.. 2. VPC网络内创建EMQ X集群'私有网络'，例如: 192.168.0.0/24

3. Create 2 EMQ X hosts inside the private network, like:

.. 3. 私有网络内创建两台EMQ X主机，指定上面创建的VPC网络,例如:

+-------+-------------+
| emqx1 | 192.168.0.2 |
+-------+-------------+
| emqx2 | 192.168.0.3 |
+-------+-------------+

4. Open the TCP ports for MQTT services (e.g. 1883,8883) on the security group. 

.. 4. 在安全组中，开放MQTT服务的TCP端口，比如1883, 8883。

5. Install and cluster EMQ X on these two hosts. Please refer to the sections of cluster installation for details.

.. 5. 安装并集群EMQ X主机，具体配置请参考安装集群章节。

6. Create ELB (Classic Load Balancer), assign the VPC network, and assign the public IP address.

.. 6. 创建ELB(Classic负载均衡器)，指定VPC网络，并指定公网IP地址。

.. 7. 在ELB上创建MQTT TCP监听器::

7. Create MQTT TCP listener on the ELB::

                 -----
                 |   |
                 | E | --TCP 1883--> EMQ X
    --TCP 1883-->| L |
                 | B | --TCP 1883--> EMQ X
                 |   |
                 -----

   .. 或创建SSL监听器，并终结SSL在LB::

   Or create SSL listener and terminate the SSL links on the ELB::

                 -----
                 |   |
                 | E | --TCP 1883--> EMQ X
    --SSL 8883-->| L |
                 | B | --TCP 1883--> EMQ X
                 |   |
                 -----

8. Connect the MQTT clients to the ELB using the public IP address and test the deployment.

.. 8. MQTT客户端连接LB公网地址测试。

-------------------
Deploying on AliYun
-------------------

.. 阿里云部署

.. TODO:: AliYun terminates the SSLs?

----------------------------
Deploying on private network
----------------------------

.. 私有网络部署

Direct connection of EMQ X cluster
----------------------------------

.. EMQ X集群直连

EMQ X cluster DNS-resolvable and the clients access the cluster via domain name or IP list:

.. EMQ X集群直接挂在DNS，设备通过域名或者IP地址列表访问:

1. Deploy EMQ X cluster. Please refer to the sections of 'program packet installation' and 'EMQ X nodes clustering' for details.

.. 1. 部署EMQ X集群，具体参考`程序包安装`与`集群配置`文档。

2. On the firewall enable the access to the MQTT ports (e.g. 1883, 8883).

.. 2. EMQ X节点防火墙开启外部MQTT访问端口，例如1883, 8883。

3. Client devices access the EMQ X cluster via domain name or IP list.

.. 3. 设备通过IP地址列表或域名访问EMQ X集群。

.. NOTE:: This kind of deployment is NOT recommended.

HAProxy -> EMQ X
----------------

HAProxy as LB for EMQ X cluster and terminates the SSL links:

.. HAProxy作为LB部署EMQ X集群，并终结SSL连接:

1. Create EMQ X Cluster nodes like following:

.. 1. 创建EMQ X集群节点，例如:

+-------+-------------+
| node  | IP          |
+=======+=============+
| emqx1 | 192.168.0.2 |
+-------+-------------+
| emqx2 | 192.168.0.3 |
+-------+-------------+

.. 2. 配置/etc/haproxy/haproxy.cfg，示例:

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

.. NGINX Plus产品作为EMQ X集群的LB，并终结SSL连接:

1. Install the NGINX Plus. An instruction for Ubuntu: https://cs.nginx.com/repo_setup

.. 1. 注册NGINX Plus试用版，Ubuntu下安装: https://cs.nginx.com/repo_setup

2. Create EMQ X cluster nodes like following:

.. 2. 创建EMQ X节点集群，例如: 

+-------+-------------+
| node  | IP          |
+=======+=============+
| emqx1 | 192.168.0.2 |
+-------+-------------+
| emqx2 | 192.168.0.3 |
+-------+-------------+


.. 3. 配置/etc/nginx/nginx.conf，示例:

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

.. 程序安装

-------------------
System Requirements
-------------------

.. 环境要求

Operating System
----------------

.. 操作系统

EMQ X is developed utilizing the Erlang/OTP language / platform. It runs on the following OS: Linux, FreeBSD, MAC OS X and Windows Server.

.. EMQ X采用Erlang/OTP语言平台开发，可跨平台运行在Linux、FreeBSD、Mac OS X、Windows服务器。

We recommend the 64-bit Linux-based cloud host or servr for the deployment.

.. 产品环境推荐部署在64-bit Linux云主机或服务器。

CPU/MEM
--------

.. CPU/内存

In the test scenario, EMQ X with 1G memory sustains 80K TCP links or 15K SSL links.  

.. EMQ X在测试场景下，1G内存承载80K TCP连接，15K SSL安全连接。

In production environment, it is suggested to deploy at least 2 nodes in the cluster. Planning the CPU and Momery capacity on the basic of concurrent connections and the message throughput.

.. 产品部署环境下，建议双机集群，根据并发连接与消息吞吐，规划节点CPU/内存。

---------------------------------
Naming Rule of Software Package
---------------------------------

.. 程序包命名

For every EMQ X release, it is distributed as software packages for Ubuntu, CentOs, FreeBSD, Mac OS X and windows. Besides, an image for Docker is also released. 

.. EMQ X每个版本会发布Ubuntu、CentOS、FreeBSD、Mac OS X、Windows平台程序包与Docker镜像。

Please contact us for the software package: http://emqtt.com/about#contacts

.. 联系EMQ公司获取程序包: http://emqtt.com/about#contacts

The package name consists of the platform name and the version number. E.g. emqx-enterprise-centos7-v2.1.0.zip

.. 程序包命名由平台、版本组成，例如: emqx-enterprise-centos7-v2.1.0.zip

.. _install_rpm:

-----------------
RPM Package
-----------------
.. RPM包安装

RPM is recommended for CentOS and RedHat. After installation, EMQ X service is managed by the OS. 

.. CentOS、RedHat操作系统下，推荐RPM包安装。RPM包安装后可通过操作系统，直接管理启停EMQ服务。

Installation
------------
.. RPM安装

.. code-block:: console

    rpm -ivh --force emqx-centos6.8-v2.1.0-1.el6.x86_64.rpm

.. NOTE:: Erlang/OTP R19 depends on lksctp-tools

.. code-block:: console

    yum install lksctp-tools

Config Files
------------

.. 配置文件

EMQ X config file: /etc/emqx/emqx.conf, config file for plugins: /etc/emqx/plugins/\*.conf

.. EMQ X配置文件: /etc/emqx/emqx.conf，插件配置文件: /etc/emqx/plugins/\*.conf。

Log Files
----------
.. 日志文件

Log files directory: /var/log/emqx

..  日志文件目录: /var/log/emqx

Data Files
----------

.. 数据文件

Data files derectory: /var/lib/emqx/

.. 数据文件目录：/var/lib/emqx/

Start/Stop
----------

.. 启动停止

.. code-block:: console

    service emqx start|stop|restart

.. _install_deb:

----------------
DEB package
----------------

.. DEB包安装

DEB is recommended for Debian and Ubuntu. After installation, EMQ X service is managed bu the OS.

.. Debian、Ubuntu操作系统下，推荐DEB包安装。DEB包安装后可通过操作系统，直接管理启停EMQ服务。

.. code-block:: console

    sudo dpkg -i emqx-ubuntu16.04_v2.1.0_amd64.deb

.. NOTE:: Erlang/OTP R19 depends on 'lksctp-tools' lib

.. code-block:: console

    apt-get install lksctp-tools

Config Files
------------

.. 配置文件

EMQ X config file: /etc/emqx/emqx.conf, plugins config file: /etc/emqx/plugins/\*.conf。

.. EMQ X配置文件: /etc/emqx/emqx.conf，插件配置文件: /etc/emqx/plugins/\*.conf。

Log Files
----------

.. 日志文件

Log files directory: /var/log.emqx

.. 日志文件目录: /var/log/emqx

Data Files
-----------
.. 数据文件

Data files directory: /var/lib/emqx/

.. 数据文件目录：/var/lib/emqx/

Start/Stop
----------

.. 启动停止

.. code-block:: console

    service emqx start|stop|restart

.. _install_on_linux:

---------------------------
EMQ X Packages for Linux
---------------------------

EMQ X Linus Packages:

.. EMQ X Linux通用程序包:

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

.. CentOS平台为例，下载安装过程:

.. code-block:: bash

    unzip emqx-enterprise-centos7-v2.1.0.zip

Use the console mode to check if EMQ X starts normal:

.. 控制台调试模式启动，检查EMQ X是否可正常启动:

.. code-block:: bash

    cd emqx && ./bin/emqx console

If EMQ X start normal, the output of console shall looks like:

.. 如启动正常，控制台输出:

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

.. CTRL+c关闭控制台。守护进程模式启动:

.. code-block:: bash

    ./bin/emqx start

Log files can be find under the log/ directory.

.. 启动错误日志将输出在log/目录。

Check the EMQ X service's status:

.. EMQ X服务进程状态查询:

.. code-block:: bash

    ./bin/emqx_ctl status

If EMQ X starts normally and runs correctly, status check shall return as following:

.. 正常运行状态，查询命令返回:

.. code-block:: bash

    $ ./bin/emqx_ctl status
    Node 'emqx@127.0.0.1' is started
    emqx 2.1.0 is running

.. EMQ X服务器提供了状态监控URL:

the status of EMQ X server can also be monitored on the following URL:

    http://localhost:8083/status

.. 停止服务器:

Stop the server::

    ./bin/emqx stop

.. _install_on_freebsd:

---------------------
Installing on FreeBSD
---------------------

.. FreeBSD服务器安装

Please contact us for the software package: http://emqtt.com/about#contacts

.. 联系EMQ公司获取程序包: http://emqtt.com/about#contacts

Installingon FreeBSD is the same as which on Linux.

.. FreeBSD平台安装过程与Linux相同。

.. _install_on_mac:

----------------------
Installing on Mac OS X
----------------------

.. Mac OS X系统安装

The to install and start EMQ X on Mac OS X is the same as which of on Linux.

.. EMQ X Mac平台下安装启动过程与Linux相同。

When developing MQTT applications on Mac, modify the 'etc/emqx.conf' file as following to check the MQTT massages on the console: 

.. Mac下开发调试MQTT应用，配置文件'etc/emqx.conf' log段落打开debug日志，控制台可以查看收发MQTT报文详细:

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

.. Docker镜像安装

Get EMQ X Docker Image:

.. EMQ X Docker镜像获取:

.. 解压emqx-enterprise-docker镜像包:

Unzip the emsq-enterprise-docker package::

    unzip emqx-enterprise-docker-v2.1.0.zip

.. 加载镜像:

Load the Image::

    docker load < emqplus-enterprise-docker-v2.1.0

.. 启动容器:

Run the container::

    docker run -itd --net='host' --name emqx20 emqx-enterprise-docker-v2.1.0

.. 停止容器:

Stop the brocker::

    docker stop emqx20

.. 开启容器:

Start the brocker::

    docker start emqx20

.. 进入Docker控制台:

Enter the running container:

    docker exec -it emqx20 /bin/bash

===========
Quick Setup
===========

.. 快速启动

Assuming a EMQ X Cluster with two Linux nodes deplyed on cloud VPC network or private network:

.. 假设部署两台EMQ X Linux节点集群，在云厂商VPC或私有网络内:

+---------------------+---------------------+
| Node name           |    IP               |
+---------------------+---------------------+
| emqx1@192.168.0.10  | 192.168.0.10        |
+---------------------+---------------------+
| emqx@192.168.0.20   | 192.168.0.20        |
+---------------------+---------------------+

..
  +---------------------+---------------------+
  | 节点名              |    IP地址           |
  +---------------------+---------------------+
  | emqx1@192.168.0.10  | 192.168.0.10        |
  +---------------------+---------------------+
  | emqx@192.168.0.20   | 192.168.0.20        |
  +---------------------+---------------------+
  
-----------------
System Parameters
-----------------

.. 操作系统参数

Deloyed under Linux, EMQ X sustains 100 concurrent connections. To achieve this, the system Kernel, Networking, the Erlang VM and EMQ X itself must be tuned.

.. EMQ X 在Linux环境下独立部署，支持10万线并发连接，需设置内核参数、TCP协议栈参数。

System-Wide File Handles
------------------------

.. 系统全局文件句柄

Maximun file handels:

.. 系统全局允许分配的最大文件句柄数256K:

.. code-block:: console

    # 2 millions system-wide
    sysctl -w fs.file-max=262144
    sysctl -w fs.nr_open=262144
    echo 262144 > /proc/sys/fs/nr_open

Maximum of file handels for current session:

.. 允许当前会话/进程打开文件句柄数:

.. code-block:: console

    ulimit -n 262144

/etc/sysctl.conf
----------------

.. 持久化'fs.file-max'设置到/etc/sysctl.conf文件:

Add 'fs.file-max' to '/etc/sysctl.conf' and make the changes permanent::

.. code-block:: console

    fs.file-max = 262144

/etc/security/limits.conf
-------------------------

.. /etc/security/limits.conf持久化设置允许用户/进程打开文件句柄数:

Persist the maximum number of opened file handles for users in /etc/security/limits.conf::

    emqx      soft   nofile      262144
    emqx      hard   nofile      262144

Note: Under Ubuntu, '/etc/systemd/system.conf' is to be modified:

.. 注: Ubuntu下需设置/etc/systemd/system.conf:

.. code-block:: properties

    DefaultLimitNOFILE=262144

---------------
EMQ X Node Name
---------------

.. EMQ X 节点名称

Set the node name and cookies(communicating between nodes)

.. 设置节点名称与Cookie(集群节点间通信认证)。

.. emqx1节点/etc/emqx/emqx.conf文件:

'/etc/emqx/emqx.conf' on emqx1::

    node.name   = emqx1@192.168.0.10
    node.cookie = secret_dist_cookie

.. emqx2节点/etc/emqx/emqx.conf文件::

'/etc/emqx/emqx.conf' on emqx2::

    node.name   = emqx2@192.168.0.20
    node.cookie = secret_dist_cookie

------------------
Start EMQ X Nodes
------------------

.. EMQ X 节点启动

.. 如果RPM或DEB方式安装，启动节点:

If EMQ X is installed using RPM or DEB::

    service emqx start

.. 如果独立zip包安装，启动节点:

if EMQ X is installed using zip package::

    ./bin/emqx start

----------------------------
Clustering the EMQ X Nodes
----------------------------

.. EMQ X 节点集群

.. 启动两台节点后，emqx1@192.168.0.10上执行:

Start the two nodes, on the emqx1@192.168.0.10 run:: 

    $ ./bin/emqx_ctl cluster join emqx2@192.168.0.20

    Join the cluster successfully.
    Cluster status: [{running_nodes,['emqx1@192.168.0.10','emqx@192.168.0.20']}]

.. 或，emqx2@192.168.0.20上执行:

or, on the emqx1@192.168.0.20 run::

    $ ./bin/emqx_ctl cluster join emqx1@192.168.0.10

    Join the cluster successfully.
    Cluster status: [{running_nodes,['emqx1@192.168.0.10','emqx@192.168.0.20']}]

.. 任意节点上查询集群状态:

Check the cluster status on any node::

    $ ./bin/emqx_ctl cluster status

    Cluster status: [{running_nodes,['emqx1@192.168.0.10','emqx@192.168.0.20']}]

-----------------------------
Managing utlizing Web Console
-----------------------------

.. Web 管理控制台

'emxq-dashboard' plugin starts the web management and provides the management service on port 18083.

.. 18083端口是Web管理控制占用，该端口由'emqx-dashboard'插件启用。

Web console URL: http://localhost:18083/, default user-name: admin, password: public.

.. 控制台URL: http:://localhost:18083/ ，默认登录用户名: admin, 密码: public。

Through the web console, the status of cluster nodes, statistic of MQTT message, MQTT clients, MQTT sessions and routing informations can be inquired.

.. 用户可以通过控制台，查询集群节点、MQTT报文统计、MQTT客户端、MQTT会话与路由信息。

.. _tcp_ports:

-------------------------
TCP Ports of MQTT Service
-------------------------

.. MQTT服务TCP端口

By default, EMQ X starts following service on these ports:

.. EMQ X 默认启用的外部MQTT服务端口包括:

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

.. 上述占用端口可通过etc/emqx.conf配置文件的'Listeners'段落设置:

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

.. 通过注释或删除相关段落，可禁用相关TCP服务启动。

-----------------------
TCP Port for Clustering
-----------------------

.. 节点集群TCP端口

The firewalls must allow the nodes access each other on the following ports:

.. EMQ X节点间防火墙必须开放下述端口:

+-----------+-----------------------------------+
| 4369      | Node discovery port               |
+-----------+-----------------------------------+
| 5369      | Data channel                      |
+-----------+-----------------------------------+
| 6369      | Control channel                   |
+-----------+-----------------------------------+

.. _青云:       https://qingcloud.com
.. _qingcloud:  https://qingcloud.com
.. _AWS:        https://aws.amazon.com
.. _阿里云:     https://www.aliyun.com
.. _aliyun:     https://www.aliyun.com
.. _UCloud:     https://ucloud.cn
.. _QCloud:     https://www.qcloud.com
.. _HAProxy:    https://www.haproxy.org
.. _NGINX:      https://www.nginx.com 

