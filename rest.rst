.. _rest:

========
REST API
========

--------
Data API
--------

Client::

    GET  api/v2/node/{node_name}/clients?curr_page=1&page_size=20

Session::

    GET  api/v2/node/{node_name}/sessions?curr_page=1&page_size=20

Route::
   
    GET  api/v2/node/{node_name}/routers?curr_page=1&page_size=20

Subscription::

    GET  api/v2/node/{node_name}/subscriptions?curr_page=1&page_size=20

Clients on a Node
-----------------

.. code-block:: bash

    GET api/v2/nodes/emqx@127.0.0.1/clients

Response

.. code-block:: json

    {
        "code": 0,
        "result":
        {
            "current_page": 1,
            "page_size": 20,
            "total_num": 1,
            "total_page": 1,
            "objects":
            [
                {
                    "client_id": "C_1492145414740",
                    "username": "undefined",
                    "ipaddress": "127.0.0.1",
                    "port": 49639,
                    "clean_sess": true,
                    "proto_ver": 4,
                    "keepalive": 60,
                    "connected_at": "2017-04-14 12:50:15"
                }
            ]
        }
    }




Client on a Node
----------------

.. code-block:: bash

    GET api/v2/nodes/emqx@127.0.0.1/clients/C_1492145414740

Response

.. code-block:: json


    {
        "code": 0,
        "result":
        {
            "objects":
            [
                {
                    "client_id": "C_1492145414740",
                    "username": "undefined",
                    "ipaddress": "127.0.0.1",
                    "port": 50953,
                    "clean_sess": true,
                    "proto_ver": 4,
                    "keepalive": 60,
                    "connected_at": "2017-04-14 13:35:15"
                }
            ]
        }
    }
    

Client in a cluster
-------------------

.. code-block:: bash

    GET api/v2/clients/C_1492145414740

Response

.. code-block:: json

    {
        "code": 0,
        "result":
        {
            "objects":
            [
                {
                    "client_id": "C_1492145414740",
                    "username": "undefined",
                    "ipaddress": "127.0.0.1",
                    "port": 50953,
                    "clean_sess": true,
                    "proto_ver": 4,
                    "keepalive": 60,
                    "connected_at": "2017-04-14 13:35:15"
                }
            ]
        }
    }
    
    

Sessions on a Node
------------------

.. code-block:: bash

    GET api/v2/nodes/emqx@127.0.0.1/sessions

Response

.. code-block:: json

    {
        "code": 0,
        "result":
        {
            "current_page": 1,
            "page_size": 20,
            "total_num": 1,
            "total_page": 1,
            "objects":
            [
                {
                    "client_id": "C_1492145414740",
                    "clean_sess": true,
                    "max_inflight": "undefined",
                    "inflight_queue": "undefined",
                    "message_queue": "undefined",
                    "message_dropped": "undefined",
                    "awaiting_rel": "undefined",
                    "awaiting_ack": "undefined",
                    "awaiting_comp": "undefined",
                    "created_at": "2017-04-14 13:35:15"
                }
            ]
        }
    }
    
A Session on a Node
-------------------

.. code-block:: bash

    GET api/v2/nodes/emqx@127.0.0.1/sessions/C_1492145414740

Response

.. code-block:: json

    {
        "code": 0,
        "result":
        {
            "objects":
            [
                {
                    "client_id": "C_1492145414740",
                    "clean_sess": true,
                    "max_inflight": "undefined",
                    "inflight_queue": "undefined",
                    "message_queue": "undefined",
                    "message_dropped": "undefined",
                    "awaiting_rel": "undefined",
                    "awaiting_ack": "undefined",
                    "awaiting_comp": "undefined",
                    "created_at": "2017-04-14 13:35:15"
                }
            ]
        }
    }
    

A Session in a Cluster
----------------------

.. code-block:: bash

    GET api/v2/sessions/C_1492145414740

Response

.. code-block:: json

    {
        "code": 0,
        "result":
        {
            "objects":
            [
                {
                    "client_id": "C_1492145414740",
                    "clean_sess": true,
                    "max_inflight": "undefined",
                    "inflight_queue": "undefined",
                    "message_queue": "undefined",
                    "message_dropped": "undefined",
                    "awaiting_rel": "undefined",
                    "awaiting_ack": "undefined",
                    "awaiting_comp": "undefined",
                    "created_at": "2017-04-14 13:35:15"
                }
            ]
        }
    }
    



Routes in a Cluster
--------------------

.. code-block:: bash

    GET api/v2/nodes/emqx@127.0.0.1/routers

Response

.. code-block:: json

    {
        "code": 0,
        "result":
        {
            "current_page": 1,
            "page_size": 20,
            "total_num": 1,
            "total_page": 1,
            "objects":
            [
                {
                    "topic": "$client/C_1492145414740",
                    "node": "emqx@127.0.0.1"
                }
            ]
        }
    }
    



A Route in a Cluster
--------------------

.. code-block:: bash

    GET api/v2/nodes/emqx@127.0.0.1/routers/$client/C_1492145414740

Response

.. code-block:: json

    {
        "code": 0,
        "result":
        {
            "objects":
            [
                {
                    "topic": "$client/C_1492145414740",
                    "node": "emqx@127.0.0.1"
                }
            ]
        }
    }
    



A Client of a Router

.. code-block:: bash

    GET api/v2/routers/$client/C_1492145414740

Response

.. code-block:: json

    {
        "code": 0,
        "result":
        {
            "objects":
            [
                {
                    "topic": "$client/C_1492145414740",
                    "node": "emqx@127.0.0.1"
                }
            ]
        }
    }
    




Subscriptions of Nodes
----------------------

.. code-block:: bash

    GET api/v2/nodes/emqx@127.0.0.1/subscriptions

Response

.. code-block:: json

    {
        "code": 0,
        "result":
        {
            "current_page": 1,
            "page_size": 20,
            "total_num": 1,
            "total_page": 1,
            "objects":
            [
                {
                    "client_id": "C_1492145414740",
                    "topic": "$client/C_1492145414740",
                    "qos": 1
                }
            ]
        }
    }
    



Subscriptions of a Node
-----------------------

.. code-block:: bash

    GET api/v2/nodes/emqx@127.0.0.1/subscriptions/C_1492145414740

Response

.. code-block:: json

    {
        "code": 0,
        "result":
        {
            "objects":
            [
                {
                    "client_id": "C_1492145414740",
                    "topic": "$client/C_1492145414740",
                    "qos": 1
                }
            ]
        }
    }
    


A Client's Subscriptions
------------------------

.. code-block:: bash

    GET api/v2/subscriptions/C_1492145414740

Response

.. code-block:: json

    {
        "code": 0,
        "result":
        {
            "objects":
            [
                {
                    "client_id": "C_1492145414740",
                    "topic": "$client/C_1492145414740",
                    "qos": 1
                }
            ]
        }
    }


---------------    
User Management
---------------

Role
----

Tag:
    - administrator
    - viewer

Login
-----

.. code-block:: bash
  
    POST /api/v2/auth

.. code-block:: json

    {
        "username": "admin",
        "password": "public"
    }

Response

.. code-block:: json

    {
        "code": 0,
        "result": []
    }


Add New User
------------

.. code-block:: bash

    POST /api/v2/users/

.. code-block:: json
    
    {
        "username": "root",
        "password": "root",
        "email": "admin@emqtt.io",
        "role": "administrator",
        "remark": "123"
    }
    
    
Response

.. code-block:: json

    {
        "code": 0,
        "result": []
    }
    


Query One User
--------------

.. code-block:: bash

    GET /api/v2/users/{username}

Response

.. code-block:: json

    {
        "code": 0,
        "result":
        {
            "username": "root",
            "email": "admin@emqtt.io",
            "role": "administrator",
            "remark": "123",
            "created_at": "2017-04-14 13 (tel:2017041413):51:43"
        }
    }



Query Users
-----------

.. code-block:: bash

    GET /api/v2/users/

Response

.. code-block:: json

    {
        "code": 0,
        "result":
        [
            {
                "username": "admin",
                "email": "admin@emqtt.io",
                "role": "administrator",
                "remark": "administrator",
                "created_at": "2017-04-07 10 (tel:2017040710):30:01"
            },
            {
                "username": "root",
                "email": "admin@emqtt.io",
                "role": "administrator",
                "remark": "123",
                "created_at": "2017-04-14 13 (tel:2017041413):51:43"
            }
        ]
    }
    


Update a User
-------------

.. code-block:: bash

    PUT /api/v2/users/{username}

<Submit Type:json>

.. code-block:: json

    {
        "email": "admin@emqtt.io",
        "role": "administrator",
        "remark": "123456"
    }


Response

.. code-block:: bash

    {
        "code": 0,
        "result": []
    }
    


Delete a User(Except Admin)
---------------------------

.. code-block:: bash

    DELETE /api/v2/users/{username}

Response

.. code-block:: json

    {
        "code": 0,
        "result": []
    }
    

-----------------
Plugin Management
-----------------

Query Plugins
-------------

.. code-block:: bash

    GET /api/v2/nodes/{node_name}/plugins/

.. code-block:: json

    {
        "code": 0,
        "result":
        [
            {
                "name": "emqx_auth_clientid",
                "version": "2.1.1",
                "description": "EMQ X Authentication with ClientId/Password",
                "active": false
            },
            {
                "name": "emqx_auth_eems",
                "version": "1.0",
                "description": "EMQ X Authentication/ACL with eems",
                "active": false
            },
            {
                "name": "emqx_auth_http",
                "version": "2.1.1",
                "description": "EMQ X Authentication/ACL with HTTP API",
                "active": false
            },
            {
                "name": "emqx_auth_ldap",
                "version": "2.1.1",
                "description": "EMQ X Authentication/ACL with LDAP",
                "active": false
            },
            {
                "name": "emqx_auth_mongo",
                "version": "2.1.1",
                "description": "EMQ X Authentication/ACL with MongoDB",
                "active": false
            },
            {
                "name": "emqx_auth_mysql",
                "version": "2.1.1",
                "description": "EMQ X Authentication/ACL with MySQL",
                "active": false
            },
            {
                "name": "emqx_auth_pgsql",
                "version": "2.1.1",
                "description": "EMQ X Authentication/ACL with PostgreSQL",
                "active": false
            },
            {
                "name": "emqx_auth_redis",
                "version": "2.1.1",
                "description": "EMQ X Authentication/ACL with Redis",
                "active": false
            },
            {
                "name": "emqx_auth_username",
                "version": "2.1.1",
                "description": "EMQ X Authentication with Username/Password",
                "active": false
            },
            {
                "name": "emqx_backend_cassa",
                "version": "2.1.1",
                "description": "EMQ X Cassandra Backend",
                "active": false
            },
            {
                "name": "emqx_backend_mongo",
                "version": "2.1.1",
                "description": "EMQ X Mongodb Backend",
                "active": false
            },
            {
                "name": "emqx_backend_mysql",
                "version": "2.1.0",
                "description": "EMQ X MySQL Backend",
                "active": false
            },
            {
                "name": "emqx_backend_pgsql",
                "version": "2.1.1",
                "description": "EMQ X PostgreSQL Backend",
                "active": false
            },
            {
                "name": "emqx_backend_redis",
                "version": "2.1.1",
                "description": "EMQ X Redis Backend",
                "active": false
            },
            {
                "name": "emqx_bridge_kafka",
                "version": "2.1.1",
                "description": "EMQ X Kafka Bridge",
                "active": false
            },
            {
                "name": "emqx_bridge_rabbit",
                "version": "2.1.1",
                "description": "EMQ X Bridge RabbitMQ",
                "active": false
            },
            {
                "name": "emqx_dashboard",
                "version": "2.1.1",
                "description": "EMQ X Dashboard",
                "active": true
            },
            {
                "name": "emqx_modules",
                "version": "2.1.1",
                "description": "EMQ X Modules",
                "active": true
            },
            {
                "name": "emqx_recon",
                "version": "2.1.1",
                "description": "Recon Plugin",
                "active": true
            },
            {
                "name": "emqx_reloader",
                "version": "2.1.1",
                "description": "Reloader Plugin",
                "active": false
            },
            {
                "name": "emqx_retainer",
                "version": "2.1.1",
                "description": "EMQ X Retainer",
                "active": true
            }
        ]
    }
    

Start/Stop a Plugin
-------------------

.. code-block:: bash

    PUT /api/v2/nodes/plugins/{name}

.. code-block:: json 

    {
        "active": true/false,
    }
    

Response

.. code-block:: json

    {
        "code": 0,
        "result": []
    }

---------------------------------------------------------------------------
Cluster Node Information (Clients, Sessions, Topics, Routes, Subscriptions)
---------------------------------------------------------------------------

List pagination example::

    GET /api/v2/node/{node_name}/clients?_page=1&_limit=10

Query data example::

    GET /api/v2/node/{node_name}/subscriptions?_page=1&_limit=10&clientId_like=111

Cluster Node Information
------------------------

.. code-block:: bash

    GET api/v2/monitoring/nodes

Response

.. code-block:: json

    {
        "code": 0,
        "result":
        [
            {
                "name": "emqx@127.0.0.1",
                "otp_release": "R19/8.3",
                "memory_total": "69.19M",
                "memory_used": "49.28M",
                "process_available": 262144,
                "process_used": 303,
                "max_fds": 256,
                "clients": 1,
                "node_status": "Running",
                "load1": "1.93",
                "load5": "1.93",
                "load15": "1.89"
            }
        ]
    }
    



One Node Information
--------------------

.. code-block:: bash

    GET api/v2/monitoring/nodes/emqx@127.0.0.1


Response
    
.. code-block:: json


    {
        "code": 0,
        "result":
        {
            "name": "emqx@127.0.0.1",
            "otp_release": "R19/8.3",
            "memory_total": "69.19M",
            "memory_used": "49.24M",
            "process_available": 262144,
            "process_used": 303,
            "max_fds": 256,
            "clients": 1,
            "node_status": "Running",
            "load1": "2.21",
            "load5": "2.00",
            "load15": "1.92"
        }
    }
    

All Listen Ports
----------------

.. code-block:: bash

    GET api/v2/monitoring/listeners

Response

.. code-block:: json

    {
        "code": 0,
        "result":
        {
            "emqx@127.0.0.1":
            [
                {
                    "protocol": "mqtt:tcp",
                    "listen": "127.0.0.1:11883",
                    "acceptors": 16,
                    "max_clients": 102400,
                    "current_clients": 0,
                    "shutdown_count": []
                },
                {
                    "protocol": "mqtt:tcp",
                    "listen": "0.0.0.0:1883",
                    "acceptors": 16,
                    "max_clients": 102400,
                    "current_clients": 0,
                    "shutdown_count": []
                },
                {
                    "protocol": "mqtt:ws",
                    "listen": "8083",
                    "acceptors": 4,
                    "max_clients": 64,
                    "current_clients": 1,
                    "shutdown_count": []
                },
                {
                    "protocol": "mqtt:ssl",
                    "listen": "8883",
                    "acceptors": 16,
                    "max_clients": 102400,
                    "current_clients": 0,
                    "shutdown_count": []
                },
                {
                    "protocol": "mqtt:wss",
                    "listen": "8084",
                    "acceptors": 4,
                    "max_clients": 64,
                    "current_clients": 0,
                    "shutdown_count": []
                }
            ]
        }
    }
    


Listen Ports on a Node
-----------------------

.. code-block:: bash

    GET api/v2/monitoring/listeners/emqx@127.0.0.1
    
Response

.. code-block:: json

    {
        "code": 0,
        "result":
        [
            {
                "protocol": "mqtt:wss",
                "listen": "8084",
                "acceptors": 4,
                "max_clients": 64,
                "current_clients": 0,
                "shutdown_count": []
            },
            {
                "protocol": "mqtt:ssl",
                "listen": "8883",
                "acceptors": 16,
                "max_clients": 102400,
                "current_clients": 0,
                "shutdown_count": []
            },
            {
                "protocol": "mqtt:ws",
                "listen": "8083",
                "acceptors": 4,
                "max_clients": 64,
                "current_clients": 1,
                "shutdown_count": []
            },
            {
                "protocol": "mqtt:tcp",
                "listen": "0.0.0.0:1883",
                "acceptors": 16,
                "max_clients": 102400,
                "current_clients": 0,
                "shutdown_count": []
            },
            {
                "protocol": "mqtt:tcp",
                "listen": "127.0.0.1:11883",
                "acceptors": 16,
                "max_clients": 102400,
                "current_clients": 0,
                "shutdown_count": []
            }
        ]
    }




Nodes Information for Management Panel
--------------------------------------

.. code-block:: bash

    GET api/v2/management/nodes
    
Response

.. code-block:: json
    
    {
        "code": 0,
        "result":
        [
            {
                "name": "emqx@127.0.0.1",
                "version": "2.1.1",
                "sysdescr": "EMQ X",
                "uptime": "1 hours, 17 minutes, 1 seconds",
                "datetime": "2017-04-14 14 (tel:2017041414):11:38",
                "otp_release": "R19/8.3",
                "node_status": "Running"
            }
        ]
    }
    
    
    
One Node Information for Management Panel
-----------------------------------------

.. code-block:: bash

    GET api/v2/management/nodes/emqx@127.0.0.1
    
Response

.. code-block:: json

    {
        "code": 0,
        "result":
        {
            "version": "2.1.1",
            "sysdescr": "EMQ X",
            "uptime": "1 hours, 17 minutes, 18 seconds",
            "datetime": "2017-04-14 14 (tel:2017041414):11:55",
            "otp_release": "R19/8.3",
            "node_status": "Running"
        }
    }
    
-------------       
MQTT Protocol
-------------


Publish Message
---------------

.. code-block:: bash

    POST api/v2/mqtt/publish

.. code-block:: json

    {
        "topic": "test",
        "payload": "hello",
        "qos": 1,
        "retain": false,
        "client_id": "C_1492145414740"
    }
    
- topic is mandatory, other parameters are optional
- default payload is an empty string
- default qos is 0
- default retain is false
- client_id is http string

Response

.. code-block:: json

    {
        "code": 0,
        "result": []
    }
    



Subscription by Broker
----------------------

.. code-block:: bash

    POST api/v2/mqtt/subscribe

        
.. code-block:: json

    {
        "topic": "test",
        "qos": 1,
        "client_id": "C_1492145414740"
    }


----------
Error Code
----------

+-------+-----------------------------------------+
| Code  | Comment                                 |
+=======+=========================================+
| 0     | Success                                 |
+-------+-----------------------------------------+
| 101   | badrpc                                  |
+-------+-----------------------------------------+
| 102   | Unknown error                           |
+-------+-----------------------------------------+
| 103   | Username or password error              |
+-------+-----------------------------------------+
| 104   | empty username or password              |
+-------+-----------------------------------------+
| 105   | user does not exist                     |
+-------+-----------------------------------------+
| 106   | admin can not be deleted                |
+-------+-----------------------------------------+
| 107   | missing request parameter               |
+-------+-----------------------------------------+
| 108   | request parameter type error            |
+-------+-----------------------------------------+
| 109   | request parameter is not a json         |
+-------+-----------------------------------------+
| 110   | plugin has been loaded                  |
+-------+-----------------------------------------+
| 111   | plugin has been unloaded                |
+-------+-----------------------------------------+
