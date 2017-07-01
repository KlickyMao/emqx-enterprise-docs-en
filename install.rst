
.. _installation:

============
Installation
============

-------------------
System Requirements
-------------------

Operating System
----------------

EMQ X depends on the Erlang/OTP platform, runs on following OSes: Linux, FreeBSD, MAC OS X and Windows Server.

Recommend a 64-bit Linux-based cloud host or server for the production deployment.

CPU/MEM
-------

In the test scenario, EMQ X with 1G memory is able to sustain 80K TCP connections or 15K SSL connections.

In production environment, it is recommended to deploy at least 2 nodes in a cluster. Evaluate CPU and Memory capacity on concurrent connections and the message throughput.

-------------------------------
Naming Rule of Software Package
-------------------------------

For each EMQ X release, it is distributed as software packages for Ubuntu, CentOs, FreeBSD, Mac OS X and windows. Besides, an image for Docker is also available.

Please contact us for the software packages: http://emqtt.com/about#contacts

The package name consists of the platform name and the version number. E.g. emqx-enterprise-centos7-v2.1.0.zip

.. _install_via_rpm:

---------------
Install via RPM
---------------

RPM is recommended for CentOS and RedHat. After installation, EMQ X service is managed by the OS.

Install the package:

.. code-block:: console

    rpm -ivh --force emqx-centos6.8-v2.1.0-1.el6.x86_64.rpm

.. NOTE:: Erlang/OTP R19 depends on lksctp-tools

.. code-block:: console

    yum install lksctp-tools

Config, Data and Log Files:

+---------------------------+------------------------------------------+
| File                      | Description                              |
+===========================+==========================================+
| /etc/emqx/emqx.conf       | EMQ X Config File                        |
+---------------------------+------------------------------------------+
| /etc/emqx/plugins/\*.conf | Plugins's config files                   |
+---------------------------+------------------------------------------+
| /var/log/emqx             | Log Files                                |
+---------------------------+------------------------------------------+
| /var/lib/emqx/            | Data Files                               |
+---------------------------+------------------------------------------+

Start/Stop the broker:

.. code-block:: console

    service emqx start|stop|restart

.. _install_via_deb:

---------------
Install via DEB
---------------

DEB is recommended for Debian and Ubuntu. After installation, EMQ X service is managed by the OS.

Install the package:

.. code-block:: console

    sudo dpkg -i emqx-ubuntu16.04_v2.1.0_amd64.deb

.. NOTE:: Erlang/OTP R19 depends on 'lksctp-tools' lib

.. code-block:: console

    apt-get install lksctp-tools

Config, Data and Log Files:

+---------------------------+------------------------------------------+
| File                      | Description                              |
+===========================+==========================================+
| /etc/emqx/emqx.conf       | EMQ X Config File                        |
+---------------------------+------------------------------------------+
| /etc/emqx/plugins/\*.conf | Plugins's config files                   |
+---------------------------+------------------------------------------+
| /var/log/emqx             | Log Files                                |
+---------------------------+------------------------------------------+
| /var/lib/emqx/            | Data Files                               |
+---------------------------+------------------------------------------+

Start/Stop the broker:

.. code-block:: console

    service emqx start|stop|restart

.. _install_on_linux:

--------------------------
General Packages for Linux
--------------------------

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

If EMQ X is started successfully, status check shall return as following:

.. code-block:: bash

    $ ./bin/emqx_ctl status
    Node 'emqx@127.0.0.1' is started
    emqx 2.1.0 is running

the status of EMQ X server can also be monitored on the following URL:

    http://localhost:8083/status

Stop the server::

    ./bin/emqx stop

.. _install_on_freebsd:

------------------
Install on FreeBSD
------------------

Please contact us for the software package: http://emqtt.com/about#contacts

Installation on FreeBSD is the same as on Linux.

.. _install_on_mac:

--------------------
Install on Mac OS X
--------------------

Same procedure as Linux.

When developing MQTT applications on Mac, modify the 'etc/emqx.conf' file as following to check the MQTT massages on the console: 

.. code-block:: properties

    ## Console log. Enum: off, file, console, both
    log.console = both

    ## Console log level. Enum: debug, info, notice, warning, error, critical, alert, emergency
    log.console.level = debug

    ## Console log file
    log.console.file = log/console.log

.. _install_via_docker:

------------------------
Install via Docker Image
------------------------

Please contact us to get the docker image: http://emqtt.com/about#contacts

Unzip the emqx-enterprise-docker package::

    unzip emqx-enterprise-docker-v2.1.0.zip

Load the Image::

    docker load < emqx-enterprise-docker-v2.1.0

Run the container::

    docker run -itd --net='host' --name emqx20 emqx-enterprise-docker-v2.1.0

Stop the broker::

    docker stop emqx20

Start the broker::

    docker start emqx20

Enter the running container::

    docker exec -it emqx20 /bin/bash

.. _qingcloud:  https://qingcloud.com
.. _AWS:        https://aws.amazon.com
.. _aliyun:     https://www.aliyun.com
.. _HAProxy:    https://www.haproxy.org
.. _NGINX:      https://www.nginx.com 

