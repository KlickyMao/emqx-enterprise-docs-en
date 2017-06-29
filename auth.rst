
.. _authentication:

==============
AuthN/AuthZ
==============

------------------------------
Design of MQTT Auth
------------------------------

EMQ X utilizes plugins to provide Auth. EMQ X supports username / password, ClientID and anonymous Auth. It also supports Auth integration with MySQL, PostgreSQL, Redis, MongoDB, HTTP and LDAP.

By default, anonymous Auth is enabled by the system. By means of loading multiple Auth plugins, an Auth chain can be thus built:: 

               -----------------           -----------------           ------------------
    Client --> | Username Auth | -ignore-> | ClientID Auth | -ignore-> | Anonymous Auth |
               -----------------           -----------------           ------------------
                      |                         |                                 |
                     \|/                       \|/                               \|/
                allow | deny              allow | deny                      allow | deny

-------------------------------
Config Anonymous Auth
-------------------------------

Anonymous Auth is the last measure of Auth chain. By default, it is enabled in file 'etc/emqx.conf'. We suggest disabling it in deployment:

.. code-block:: properties

    ## Allow Anonymous authentication
    mqtt.allow_anonymous = true

-------------------------
Access Control List (ACL)
-------------------------

EMQ X Server utilizes Access Control List (ACL) to realize the access control on the clients.

ACL defines::

    Allow|Deny Whom Subscribe|Publish Topics

When MQTT clients subscribe to topics or publish messages, the EMQ X access control module tries to match the rules in the list till successfully matching otherwise it fallbacks to default routine::

              ---------              ---------              ---------
    Client -> | Rule1 | --nomatch--> | Rule2 | --nomatch--> | Rule3 | --> Default
              ---------              ---------              ---------
                  |                      |                      |
                match                  match                  match
                 \|/                    \|/                    \|/
            allow | deny           allow | deny           allow | deny

-------------------------------------
Default Access Control Configuration
-------------------------------------

The default access control of EMQ X server is configured by the file acl.conf:

.. code-block:: properties

    ## Default ACL File
    mqtt.acl_file = etc/acl.conf

ACL is defined in ‘etc/acl.conf’, and is loaded when EMQ X starts:

.. code-block:: erlang

    %% Allow 'dashboard' to subscribe '$SYS/#'
    {allow, {user, "dashboard"}, subscribe, ["$SYS/#"]}.

    %% Allow clients from localhost to subscribe any topics
    {allow, {ipaddr, "127.0.0.1"}, pubsub, ["$SYS/#", "#"]}.

    %% Deny clients to subscribe '$SYS#' and '#'
    {deny, all, subscribe, ["$SYS/#", {eq, "#"}]}.

    %% Allow all by default
    {allow, all}.

If ACL is modified, it can be reloaded using CLI:

.. code-block:: console

    $ ./bin/emqx_ctl acl reload

    reload acl_internal successfully

-------------------------------
List of Auth Plugins
-------------------------------

EMQ X supports integrated authentications by using ClientId, Username, HTTP, LDAP, MySQL, Redis, PostgreSQL and MongoDB. Multiple Auth plugins can be loaded simultaneously to build an authentication chain.

The config files of Auth plugins are located in '/etc/emqx/plugins/'(RPM/DEB installation) or in 'etc/plugins/'(standalone installation):

+-------------------------+---------------------------+---------------------------------------+
| Auth Plugin             | Config file               | Description                           |
+=========================+===========================+=======================================+
| emqx_auth_clientid      | emqx_auth_clientid.conf   | ClientId AuthN/AuthZ Plugin           |
+-------------------------+---------------------------+---------------------------------------+
| emqx_auth_username      | emqx_auth_username.conf   | username/password AuthN/AuthZ Plugin  |
+-------------------------+---------------------------+---------------------------------------+
| emqx_auth_ldap          | emqx_auth_ldap.conf       | LDAP AuthN/AuthZ Plugin               |
+-------------------------+---------------------------+---------------------------------------+
| emqx_auth_http          | emqx_auth_http.conf       | HTTP AuthN/AuthZ                      |
+-------------------------+---------------------------+---------------------------------------+
| emqx_auth_mysql         | emqx_auth_redis.conf      | MySQL AuthN/AuthZ                     |
+-------------------------+---------------------------+---------------------------------------+
| emqx_auth_pgsql         | emqx_auth_mysql.conf      | Postgre AuthN/AuthZ                   |
+-------------------------+---------------------------+---------------------------------------+
| emqx_auth_redis         | emqx_auth_pgsql.conf      | Redis AuthN/AuthZ                     |
+-------------------------+---------------------------+---------------------------------------+
| emqx_auth_mongo         | emqx_auth_mongo.conf      | MongoDB AuthN/AuthZ                   |
+-------------------------+---------------------------+---------------------------------------+

-------------------------------
Configure ClientID Auth Plugin
-------------------------------

Modify the 'emqx_auth_clientid.conf' to configure the ClientID / Password list:

.. code-block:: properties

    ## auth.client.${id}.clientid = ${clientid}
    ## auth.client.${id}.password = ${password}

    ## Examples
    auth.client.1.clientid = id
    auth.client.1.password = passwd
    auth.client.2.clientid = dev:devid
    auth.client.2.password = passwd2
    auth.client.3.clientid = app:appid
    auth.client.3.password = passwd3
    auth.client.4.clientid = client~!@#$%^&*()_+
    auth.client.4.password = passwd~!@#$%^&*()_+

Load ClientId Auth plugin:

.. code-block:: console

    ./bin/emqx_ctl plugins load emqx_auth_clientid

-------------------------------
Configure Username Auth Plugin
-------------------------------

Modify the 'emqx_auth_username.conf' to configure the Username / Password list:

.. code-block:: properties

    ##auth.user.$N.username = admin
    ##auth.user.$N.password = public

    ## Examples:
    ##auth.user.1.username = admin
    ##auth.user.1.password = public
    ##auth.user.2.username = feng@emqtt.io
    ##auth.user.2.password = public
    ##auth.user.3.username = name~!@#$%^&*()_+
    ##auth.user.3.password = pwsswd~!@#$%^&*()_+

Load Username Auth plugin:

.. code-block:: console

    ./bin/emqx_ctl plugins load emqx_auth_username

After the plugin is loaded, there are two possible ways to add users:

1. Modify the 'emqx_auth_username.conf' and add user using plain text::

    auth.user.1.username = admin
    auth.user.1.password = public

2. Use the './bin/emqx_ctl' CLI tool to add users:

.. code-block:: console

   $ ./bin/emqx_ctl users add <Username> <Password>

-------------------------------
Configure LDAP Auth Plugin
-------------------------------

Modify the 'emqx_auth_ldap.conf' file and configure the LDAP Auth Plugin:

.. code-block:: properties

    auth.ldap.servers = 127.0.0.1

    auth.ldap.port = 389

    auth.ldap.timeout = 30

    auth.ldap.user_dn = uid=%u,ou=People,dc=example,dc=com

    auth.ldap.ssl = false

Load the LDAP Auth plugin:

.. code-block:: console

    ./bin/emqx_ctl plugins load emqx_auth_ldap

----------------------------
Configure HTTP Auth Plugin
----------------------------

Modify the 'emqx_auth_http.conf' and configure the HTTP Auth plugin:

.. code-block:: properties

    ## Variables: %u = username, %c = clientid, %a = ipaddress, %P = password, %t = topic

    auth.http.auth_req = http://127.0.0.1:8080/mqtt/auth
    auth.http.auth_req.method = post
    auth.http.auth_req.params = clientid=%c,username=%u,password=%P

Setup the Super User URL and arguments:

.. code-block:: properties

    auth.http.super_req = http://127.0.0.1:8080/mqtt/superuser
    auth.http.super_req.method = post
    auth.http.super_req.params = clientid=%c,username=%u

Setup the ACL URL and arguments:

.. code-block:: properties

    ## 'access' parameter: sub = 1, pub = 2
    auth.http.acl_req = http://127.0.0.1:8080/mqtt/acl
    auth.http.acl_req.method = get
    auth.http.acl_req.params = access=%A,username=%u,clientid=%c,ipaddr=%a,topic=%t

    auth.http.acl_nomatch = deny

Design of HTTP Auth and ACL server API::

    If Auth/ACL sucesses, API returns 200

    If Auth/ACL fails, API return 4xx

Load HTTP Auth plugin:

.. code-block:: console

    ./bin/emqx_ctl plugins load emqx_auth_http

----------------------------
Configure MySQL Auth Plugin
----------------------------

Modify the 'emqx_auth_mysql.conf' to configure the default MQTT user, ACL and Auth:

MQTT Auth User List
-------------------

.. code-block:: sql

    CREATE TABLE `mqtt_user` (
      `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
      `username` varchar(100) DEFAULT NULL,
      `password` varchar(100) DEFAULT NULL,
      `salt` varchar(20) DEFAULT NULL,
      `is_superuser` tinyint(1) DEFAULT 0,
      `created` datetime DEFAULT NULL,
      PRIMARY KEY (`id`),
      UNIQUE KEY `mqtt_username` (`username`)
    ) ENGINE=MyISAM DEFAULT CHARSET=utf8;

.. NOTE:: User can define the user list table and configure it in the 'authquery' statement.

MQTT Access Control List
------------------------

.. code-block:: sql

    CREATE TABLE `mqtt_acl` (
      `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
      `allow` int(1) DEFAULT NULL COMMENT '0: deny, 1: allow',
      `ipaddr` varchar(60) DEFAULT NULL COMMENT 'IpAddress',
      `username` varchar(100) DEFAULT NULL COMMENT 'Username',
      `clientid` varchar(100) DEFAULT NULL COMMENT 'ClientId',
      `access` int(2) NOT NULL COMMENT '1: subscribe, 2: publish, 3: pubsub',
      `topic` varchar(100) NOT NULL DEFAULT '' COMMENT 'Topic Filter',
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

    INSERT INTO `mqtt_acl` (`id`, `allow`, `ipaddr`, `username`, `clientid`, `access`, `topic`)
    VALUES
        (1,1,NULL,'$all',NULL,2,'#'),
        (2,0,NULL,'$all',NULL,1,'$SYS/#'),
        (3,0,NULL,'$all',NULL,1,'eq #'),
        (5,1,'127.0.0.1',NULL,NULL,2,'$SYS/#'),
        (6,1,'127.0.0.1',NULL,NULL,2,'#'),
        (7,1,NULL,'dashboard',NULL,1,'$SYS/#');

MySQL Server Address
--------------------

.. code-block:: properties

    ## Mysql Server 3306, 127.0.0.1:3306, localhost:3306
    auth.mysql.server = 127.0.0.1:3306

    ## Mysql Pool Size
    auth.mysql.pool = 8

    ## Mysql Username
    ## auth.mysql.username = 

    ## Mysql Password
    ## auth.mysql.password = 

    ## Mysql Database
    auth.mysql.database = mqtt

Configure MySQL Auth Query Statement
------------------------------------

.. code-block:: properties

    ## Variables: %u = username, %c = clientid

    ## Authentication Query: select password only
    auth.mysql.auth_query = select password from mqtt_user where username = '%u' limit 1

    ## Password hash: plain, md5, sha, sha256, pbkdf2
    auth.mysql.password_hash = sha256

    ## sha256 with salt prefix
    ## auth.mysql.password_hash = salt sha256

    ## sha256 with salt suffix
    ## auth.mysql.password_hash = sha256 salt

    ## %% Superuser Query
    auth.mysql.super_query = select is_superuser from mqtt_user where username = '%u' limit 1

Configure MySQL ACL Query Statement
------------------------------------

.. code-block:: properties

    ## ACL Query Command
    auth.mysql.acl_query = select allow, ipaddr, username, clientid, access, topic from mqtt_acl where ipaddr = '%a' or username = '%u' or username = '$all' or clientid = '%c'

    ## ACL nomatch
    auth.mysql.acl_nomatch = deny

Load MySQL Auth Plugin
-----------------------

.. code-block:: console

    ./bin/emqx_ctl plugins load emqx_auth_mysql

--------------------------------
Configure PostgreSQL Auth Plugin
--------------------------------

Modify the 'emqx_auth_pgsql.conf' file to configure the MQTT user, ACL and Auth:

Postgre MQTT User List
----------------------

.. code-block:: sql

    CREATE TABLE mqtt_user (
      id SERIAL primary key,
      is_superuser boolean,
      username character varying(100),
      password character varying(100),
      salt character varying(40)
    );

.. NOTE:: User can define the user list table and configure it in the 'authquery' statement.

Postgre MQTT Access Control List
--------------------------------

.. code-block:: sql

    CREATE TABLE mqtt_acl (
      id SERIAL primary key,
      allow integer,
      ipaddr character varying(60),
      username character varying(100),
      clientid character varying(100),
      access  integer,
      topic character varying(100)
    );

    INSERT INTO mqtt_acl (id, allow, ipaddr, username, clientid, access, topic)
    VALUES
        (1,1,NULL,'$all',NULL,2,'#'),
        (2,0,NULL,'$all',NULL,1,'$SYS/#'),
        (3,0,NULL,'$all',NULL,1,'eq #'),
        (5,1,'127.0.0.1',NULL,NULL,2,'$SYS/#'),
        (6,1,'127.0.0.1',NULL,NULL,2,'#'),
        (7,1,NULL,'dashboard',NULL,1,'$SYS/#');

Postgre Server Address
----------------------

.. code-block:: properties

    ## Postgre Server
    auth.pgsql.server = 127.0.0.1:5432

    auth.pgsql.pool = 8

    auth.pgsql.username = root

    #auth.pgsql.password = 

    auth.pgsql.database = mqtt

    auth.pgsql.encoding = utf8

    auth.pgsql.ssl = false

Configure PostgreSQL Auth Query Statement
----------------------------------------------

.. code-block:: properties

    ## Variables: %u = username, %c = clientid, %a = ipaddress

    ## Authentication Query: select password only
    auth.pgsql.auth_query = select password from mqtt_user where username = '%u' limit 1

    ## Password hash: plain, md5, sha, sha256, pbkdf2
    auth.pgsql.password_hash = sha256

    ## sha256 with salt prefix
    ## auth.pgsql.password_hash = salt sha256

    ## sha256 with salt suffix
    ## auth.pgsql.password_hash = sha256 salt

    ## Superuser Query
    auth.pgsql.super_query = select is_superuser from mqtt_user where username = '%u' limit 1

Configure PostgreSQL ACL Query Statement
------------------------------------------

.. code-block:: properties

    ## ACL Query. Comment this query, the acl will be disabled.
    auth.pgsql.acl_query = select allow, ipaddr, username, clientid, access, topic from mqtt_acl where ipaddr = '%a' or username = '%u' or username = '$all' or clientid = '%c'

    ## If no rules matched, return...
    auth.pgsql.acl_nomatch = deny

Load Postgre Auth Plugin
-------------------------

.. code-block:: bash

    ./bin/emqx_ctl plugins load emqx_auth_pgsql

----------------------------
Configure Redis Auth Plugin
----------------------------

Config file 'emqx_auth_redis.conf':

Redis Server Address 
---------------------

.. code-block:: properties

    ## Redis Server
    auth.redis.server = 127.0.0.1:6379

    ## Redis Pool Size
    auth.redis.pool = 8

    ## Redis Database
    auth.redis.database = 0

    ## Redis Password
    ## auth.redis.password =

Configure Auth Query Command
-----------------------------

.. code-block:: properties

    ## Variables: %u = username, %c = clientid

    ## Authentication Query Command
    auth.redis.auth_cmd = HGET mqtt_user:%u password

    ## Password hash: plain, md5, sha, sha256, pbkdf2
    auth.redis.passwd.hash = sha256

    ## Superuser Query Command
    auth.redis.super_cmd = HGET mqtt_user:%u is_superuser

Configure ACL Query Command
----------------------------

.. code-block:: properties

    ## ACL Query Command
    auth.redis.acl_cmd = HGETALL mqtt_acl:%u

    ## ACL nomatch
    auth.redis.acl_nomatch = deny

Redis Authed Users Hash
------------------------

By default, Hash is used to store Authed users::

    HSET mqtt_user:<username> is_superuser 1
    HSET mqtt_user:<username> password "passwd"

Redis ACL Rules Hash
---------------------

By default, Hash is used to store ACL rules::

    HSET mqtt_acl:<username> topic1 1
    HSET mqtt_acl:<username> topic2 2
    HSET mqtt_acl:<username> topic3 3

.. NOTE:: 1: subscribe, 2: publish, 3: pubsub

Load Redis Auth Plugin
-----------------------

.. code-block:: bash

    ./bin/emqx_ctl plugins load emqx_auth_redis

-----------------------------
Configure MongoDB Auth Plugin
-----------------------------

Modify the 'emqx_auth_mongo.conf' file to configure MongoDB, MQTT users and ACL Collection:

MongoDB Server
-----------------

.. code-block:: properties

    ## Mongo Server
    auth.mongo.server = 127.0.0.1:27017

    ## Mongo Pool Size
    auth.mongo.pool = 8

    ## Mongo User
    ## auth.mongo.user = 

    ## Mongo Password
    ## auth.mongo.password = 

    ## Mongo Database
    auth.mongo.database = mqtt

Configure Auth Query Collection
--------------------------------

.. code-block:: properties

    ## authquery
    auth.mongo.authquery.collection = mqtt_user

    auth.mongo.authquery.password_field = password

    auth.mongo.authquery.password_hash = sha256

    auth.mongo.authquery.selector = username=%u

    ## superquery
    auth.mongo.superquery.collection = mqtt_user

    auth.mongo.superquery.super_field = is_superuser

    auth.mongo.superquery.selector = username=%u

    ## acl_query
    auth.mongo.acl_query.collection = mqtt_user

    auth.mongo.acl_query.selector = username=%u

    ## acl_nomatch
    auth.mongo.acl_nomatch = deny

Configure ACL Query Collection
------------------------------

.. code-block:: properties

    ## aclquery
    auth.mongo.aclquery.collection = mqtt_acl

    auth.mongo.aclquery.selector = username=%u

    ## acl_nomatch
    auth.mongo.acl_nomatch = deny

MongoDB Database
----------------

.. code-block:: console

    use mqtt
    db.createCollection("mqtt_user")
    db.createCollection("mqtt_acl")
    db.mqtt_user.ensureIndex({"username":1})

.. NOTE:: The DB name and Collection name are free of choice

Example of a MongoDB User Collection 
------------------------------------

.. code-block:: javascript

    {
        username: "user",
        password: "password hash",
        is_superuser: boolean (true, false),
        created: "datetime"
    }

    db.mqtt_user.insert({username: "test", password: "password hash", is_superuser: false})
    db.mqtt_user:insert({username: "root", is_superuser: true})

Example of a MongoDB ACL Collection
------------------------------------

.. code-block:: javascript

    {
        username: "username",
        clientid: "clientid",
        publish: ["topic1", "topic2", ...],
        subscribe: ["subtop1", "subtop2", ...],
        pubsub: ["topic/#", "topic1", ...]
    }

    db.mqtt_acl.insert({username: "test", publish: ["t/1", "t/2"], subscribe: ["user/%u", "client/%c"]})
    db.mqtt_acl.insert({username: "admin", pubsub: ["#"]})

Load Mognodb Auth Plugin
-------------------------

.. code-block:: bash

    ./bin/emqx_ctl plugins load emqx_auth_mongo

.. _recon: http://ferd.github.io/recon/


