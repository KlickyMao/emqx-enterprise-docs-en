
.. _design:

======
Design
======

.. _intro:

------------
Intro
------------

Upgraded from EMQ R1, EMQ X R 2.0 separated the Message Flow Plane and Monitor/Control Plane. This makes possible of sustaining million-level MQTT connections. Separating of Flow Plane and Monitor/Control Plane also makes the EMQ cluster more reliable and provides higher performance. The Architecture of EMQ X R2 is like following::

              Control Plane
           --------------------
              |            |
  Frontend -> | Flow Plane | -> Backend
              |            |
            Session      Router
           ---------------------
               Monitor Plane


EMQ X supports MQTT message persistence to Redis, MySQL, PostgreSQL, MongoDB and Cassandra. It can bridge and forward MQTT messages to enterprise middle-ware like Kafka and RabbitMQ.

Full Asynchronous Architecture
------------------------------

The full asynchronous architecture of EMQ message broker is based on Erlang/OTP platform: asynchronous TCP connection processing, asynchronous topic subscription, asynchronous message publishing.

A MQTT message from Publisher to Subscriber, it is asynchronously processed in the emqx broker by a series of Erlang process::

                      ----------          -----------          ----------
    Publisher --Msg-->| Client | --Msg--> | Session | --Msg--> | Client | --Msg--> Subscriber
                      ----------          -----------          ----------

Message Persistence
-------------------

EMQ X separates Message routing and message persistence. It supports message persistence to Redis, MySQL, PostgreSQl, MongoDB, Canssandra and middleware like Kafka.

.. _architecture:

------------
Architecture
------------

Concept 
--------

The EMQ X broker is more like a network Switch or Router, not a traditional enterprise message queue. Compared to a network router that routes packets based on IP or MPLS label, the EMQ broker routes MQTT messages based on topic trie.

.. image:: ./_static/images/concept.png

Design Philosophy
-----------------

1. Focus on handling millions of MQTT connections and routing MQTT messages between clustered nodes.

2. Embrace Erlang/OTP, The Soft-Realtime, Low-Latency, Concurrent and Fault-Tolerant Platform.

3. Layered Design: Connection, Session, PubSub and Router Layers.

4. Separate the Message Flow Plane and the Control/Management Plane.

5. Stream MQTT messages to various backends including MQ or databases.

System Layers
--------------

1. Connection Layer: Handle TCP and WebSocket connections, encode/decode MQTT packets.

2. Session Layer: Process MQTT PUBLISH/SUBSCRIBE Packets received from client, and deliver MQTT messages to client.

3. Route Layer: Dispatch MQTT messages to subscribers in a node.

4. Distributed Layer: Route MQTT messages in the distributed nodes.

5. Authentication and Access Control: The connection layer supports extensible Auth and ACL modules.

6. Hooks and Plugins: Extensible hooks are supported at every system layer. It makes the Broker extensible by means of plugins.

.. _connection_layer:

-----------------
Connection Layer
-----------------

Connection Layer handles server side socket connection and MQTT protocol decoding:

1. Built on `eSockd`_ asynchronous TCP server side framework
2. TCP Acceptor Pool and Asynchronous TCP Accept
3. TCP/SSL, WebSocket/SSL
4. Max connections management
5. Access control on peer address or CIDR
6. Flow control based Leaky Bucket algorithm
7. MQTT protocol en/decode
8. MQTT connection keepalive
9. MQTT packet process

.. _session_layer:

--------------
Session Layer
--------------

Session layer processes Publish and Subscribe service of MQTT protocol:

1. It Stores the clients' subscription and finalize the QoS of subscriptions.

2. It processes the publish and delivery of QoS1/2 messages, retransmits time-messages and retains offline messages.

3. It manages Inflight Window and controls the message delivery throughput and order of transmission.

4. It retains the sent to client but not acknowledged QoS1/2 messages.

5. It retains QoS2 messages from client to server, which has not yet received a responding PUBREL message.

6. It retains QoS1/2 offline message of a persistent session, when the client is disconnected.

MQueue and Inflight Window
--------------------------

Concept of Message Queue and Inflight Window::

       |<----------------- Max Len ----------------->|
       -----------------------------------------------
 IN -> |      Messages Queue   |  Inflight Window    | -> Out
       -----------------------------------------------
                               |<---   Win Size  --->|


1. Inflight Window to store the messages delivered and await for PUBACK.

2. Enqueue messages when the inflight window is full.

3. If the queue is full, drop qos0 messages if store_qos0 is true, otherwise drop the oldest one.

The larger the inflight window size is, the higher the throughput is. The smaller the window size is, the more strict the message order is.

PacketId and MessageID
----------------------

The 16-bit PacketId is defined by MQTT Protocol Specification, used by client/server to PUBLISH/PUBACK packets. A GUID(128-bit globally unique Id) will be generated by the broker and assigned to a MQTT message.

Format of the globally unique message id::

        --------------------------------------------------------
        |        Timestamp       |  NodeID + PID  |  Sequence  |
        |<------- 64bits ------->|<--- 48bits --->|<- 16bits ->|
        --------------------------------------------------------

1. Timestamp: erlang:system_time if Erlang >= R18, otherwise os:timestamp

2. NodeId: encode node() to 2 bytes integer

3. Pid: encode pid to 4 bytes integer

4. Sequence: 2 bytes sequence in one process

The PacketId and MessageId in a End-to-End Message PubSub Sequence::

    PktId <-- Session --> MsgId <-- Router --> MsgId <-- Session --> PktId

.. _route_layer:

-------------
PuBSub Layer
-------------

The PubSub layer maintains a subscription table and is responsible to dispatch MQTT messages to subscribers.

.. image:: ./_static/images/dispatch.jpg

MQTT messages will be dispatched to the subscriber’s session, which finally delivers the messages to client.

.. _distributed_layer:

--------------
Routing Layer
--------------

The routing(distributed) layer maintains and replicates the global Topic Trie and Routing Table. The topic tire is composed of wildcard topics created by subscribers. The Routing Table maps a topic to nodes in the cluster.

For example, if node1 subscribed ‘t/+/x’ and ‘t/+/y’, node2 subscribed ‘t/#’ and node3 subscribed ‘t/a’, there will be a topic trie and route table::

    -------------------------
    |            t          |
    |           / \         |
    |          +   #        |
    |        /  \           |
    |      x      y         |
    -------------------------
    | t/+/x -> node1, node3 |
    | t/+/y -> node1        |
    | t/#   -> node2        |
    | t/a   -> node3        |
    -------------------------

The routing layer would route MQTT messages among clustered nodes by topic trie match and routing table lookup:

.. image:: ./_static/images/route.png

.. _auth_acl:

---------------------
Authentication & ACL
---------------------

EMQ X supports an extensible authentication and ACL mechanism, which is implemented in emqx_access_control, emqx_auth_mod and emqx_acl_mod. 

emqx_access_control provides APIs for registering and unregistering Auth or ACL modules::

    register_mod(auth | acl, atom(), list()) -> ok | {error, any()}.

    register_mod(auth | acl, atom(), list(), non_neg_integer()) -> ok | {error, any()}.

Authentication
---------------

emqx_auth_mod defines the behaviour of a authentication module::

    -module(emqx_auth_mod).

    -ifdef(use_specs).

    -callback init(AuthOpts :: list()) -> {ok, State :: any()}.

    -callback check(Client, Password, State) -> ok | ignore | {error, string()} when
        Client    :: mqtt_client(),
        Password  :: binary(),
        State     :: any().

    -callback description() -> string().

    -else.

    -export([behaviour_info/1]).

    behaviour_info(callbacks) ->
        [{init, 1}, {check, 3}, {description, 0}];
    behaviour_info(_Other) ->
        undefined.

    -endif.

Access Control (ACL)
--------------------

emqx_acl_mod defines the behaviour of an ACL module::

    -module(emqx_acl_mod).

    -include("emqx.hrl").

    -ifdef(use_specs).

    -callback init(AclOpts :: list()) -> {ok, State :: any()}.

    -callback check_acl({Client, PubSub, Topic}, State :: any()) -> allow | deny | ignore when
        Client   :: mqtt_client(),
        PubSub   :: pubsub(),
        Topic    :: binary().

    -callback reload_acl(State :: any()) -> ok | {error, any()}.

    -callback description() -> string().

    -else.

    -export([behaviour_info/1]).

    behaviour_info(callbacks) ->
        [{init, 1}, {check_acl, 2}, {reload_acl, 1}, {description, 0}];
    behaviour_info(_Other) ->
        undefined.

    -endif.

emqx_acl_internal implements the default access control based on 'etc/acl.conf' file::

    %%%-----------------------------------------------------------------------------
    %%%
    %%% -type who() :: all | binary() |
    %%%                {ipaddr, esockd_access:cidr()} |
    %%%                {client, binary()} |
    %%%                {user, binary()}.
    %%%
    %%% -type access() :: subscribe | publish | pubsub.
    %%%
    %%% -type topic() :: binary().
    %%%
    %%% -type rule() :: {allow, all} |
    %%%                 {allow, who(), access(), list(topic())} |
    %%%                 {deny, all} |
    %%%                 {deny, who(), access(), list(topic())}.
    %%%
    %%%-----------------------------------------------------------------------------

    {allow, {user, "dashboard"}, subscribe, ["$SYS/#"]}.

    {allow, {ipaddr, "127.0.0.1"}, pubsub, ["$SYS/#", "#"]}.

    {deny, all, subscribe, ["$SYS/#", {eq, "#"}]}.

    {allow, all}.

.. _hook:

--------------
Hooks
--------------

Defining Hook
--------------

EMQ X broker utilizes  hooks when: a client is connected / disconnected, topic(s) subscribed / unsubscribed or a message published / delivered/ acknowledged.

Following hooks are defined: 

+------------------------+----------------------------------+
| Hook                   | Description                      |
+========================+==================================+
| client.connected       | Client connected                 |
+------------------------+----------------------------------+
| client.subscribe       | client subscribes to topics      |
+------------------------+----------------------------------+
| client.unsubscribe     | Client unsubscribes to topics    |
+------------------------+----------------------------------+
| session.subscribed     | Client subscribed to topics      |
+------------------------+----------------------------------+
| session.unsubscribed   | Client unsubscribed to topics    |
+------------------------+----------------------------------+
| message.publish        | MQTT message published           |
+------------------------+----------------------------------+
| message.delivered      | MQTT message delivered           |
+------------------------+----------------------------------+
| message.acked          | MQTT message acknowledged        |
+------------------------+----------------------------------+
| client.disconnected    | Client disconnected              |
+------------------------+----------------------------------+

EMQ X uses (`Chain-of-responsibility_pattern`_) to implement hook mechanism. The callback functions registered to hook will be executed one by one::

                  --------  ok | {ok, NewAcc}   --------  ok | {ok, NewAcc}   --------
  (Args, Acc) --> | Fun1 | -------------------> | Fun2 | -------------------> | Fun3 | --> {ok, Acc} | {stop, Acc}
                  --------                      --------                      --------
                     |                             |                             |
                stop | {stop, NewAcc}         stop | {stop, NewAcc}         stop | {stop, NewAcc}
  
  
.. image:: ./_static/images/hooks_chain.jpg

The input parameters for a callback function are depending on the types of hook. Clone the emqx_plugin_template project to check the parameter in detail: 

+-----------------+------------------------+
| Return          | Description            |
+=================+========================+
| ok              | Continue               |
+-----------------+------------------------+
| {ok, NewAcc}    | Return Acc and continue|
+-----------------+------------------------+
| stop            | Break                  |
+-----------------+------------------------+
| {stop, NewAcc}  | Return Acc and break   |
+-----------------+------------------------+

Hook Implementation
-------------------

The Hook API is defined in emqx module:

.. code-block:: erlang

    -module(emqx).

    %% Hooks API
    -export([hook/4, hook/3, unhook/2, run_hooks/3]).
    hook(Hook :: atom(), Callback :: function(), InitArgs :: list(any())) -> ok | {error, any()}.

    hook(Hook :: atom(), Callback :: function(), InitArgs :: list(any()), Priority :: integer()) -> ok | {error, any()}.

    unhook(Hook :: atom(), Callback :: function()) -> ok | {error, any()}.

    run_hooks(Hook :: atom(), Args :: list(any()), Acc :: any()) -> {ok | stop, any()}.

The implementation of Hook is in emqx_hook module:

.. code-block:: erlang

    -module(emqx_hook).

    %% Hooks API
    -export([add/3, add/4, delete/2, run/3, lookup/1]).

    add(HookPoint :: atom(), Callback :: function(), InitArgs :: list(any())) -> ok.

    add(HookPoint :: atom(), Callback :: function(), InitArgs :: list(any()), Priority :: integer()) -> ok.

    delete(HookPoint :: atom(), Callback :: function()) -> ok.

    run(HookPoint :: atom(), Args :: list(any()), Acc :: any()) -> any().

    lookup(HookPoint :: atom()) -> [#callback{}].

Hook Usage
--------------

emq_plugin_template privodes examples of hook usage. Following is an example for end to end message processing:

.. code-block:: erlang

    -module(emq_plugin_template).

    -export([load/1, unload/0]).

    -export([on_message_publish/2, on_message_delivered/4, on_message_acked/4]).

    load(Env) ->
        emqx:hook('message.publish', fun ?MODULE:on_message_publish/2, [Env]),
        emqx:hook('message.delivered', fun ?MODULE:on_message_delivered/4, [Env]),
        emqx:hook('message.acked', fun ?MODULE:on_message_acked/4, [Env]).

    on_message_publish(Message, _Env) ->
        io:format("publish ~s~n", [emqx_message:format(Message)]),
        {ok, Message}.

    on_message_delivered(ClientId, _Username, Message, _Env) ->
        io:format("delivered to client ~s: ~s~n", [ClientId, emqx_message:format(Message)]),
        {ok, Message}.

    on_message_acked(ClientId, _Username, Message, _Env) ->
        io:format("client ~s acked: ~s~n", [ClientId, emqx_message:format(Message)]),
        {ok, Message}.

    unload() ->
        emqx:unhook('message.publish', fun ?MODULE:on_message_publish/2),
        emqx:unhook('message.acked', fun ?MODULE:on_message_acked/4),
        emqx:unhook('message.delivered', fun ?MODULE:on_message_delivered/4).

.. _plugin:

----------------
Plugin Design
----------------

Plugin is a normal erlang application that can be started/stopped dynamically by a running EMQ X broker.

emqx_plugins module implements the plugin mechanism and provides API to load and unload plugins::

    -module(emqx_plugins).

    -export([load/1, unload/1]).

    %% @doc Load a Plugin
    load(PluginName :: atom()) -> ok | {error, any()}.

    %% @doc UnLoad a Plugin
    unload(PluginName :: atom()) -> ok | {error, any()}.

User can load and unload plugins using the CLI command './bin/empx_ctl'::

    ./bin/emqx_ctl plugins load emq_auth_redis

    ./bin/emqx_ctl plugins unload emq_auth_redis

Plugin developer please refer to: http://github.com/emqtt/emqx_plugin_template

-----------------
Mnesia/ETS Tables
-----------------

+--------------------+--------+----------------------------------------+
| Table              | Type   | Description                            |
+====================+========+========================================+
| mqtt_trie          | mnesia | Trie Table                             |
+--------------------+--------+----------------------------------------+
| mqtt_trie_node     | mnesia | Trie Node Table                        |
+--------------------+--------+----------------------------------------+
| mqtt_route         | mnesia | Global Route Table                     |
+--------------------+--------+----------------------------------------+
| mqtt_local_route   | mnesia | Local Route Table                      |
+--------------------+--------+----------------------------------------+
| mqtt_pubsub        | ets    | PubSub Tab                             |
+--------------------+--------+----------------------------------------+
| mqtt_subscriber    | ets    | Subscriber Tab                         |
+--------------------+--------+----------------------------------------+
| mqtt_subscription  | ets    | Subscription Tab                       |
+--------------------+--------+----------------------------------------+
| mqtt_session       | mnesia | Global Session Table                   |
+--------------------+--------+----------------------------------------+
| mqtt_local_session | ets    | Local Session Table                    |
+--------------------+--------+----------------------------------------+
| mqtt_client        | ets    | Client Table                           |
+--------------------+--------+----------------------------------------+
| mqtt_retained      | mnesia | Retained Message Table                 |
+--------------------+--------+----------------------------------------+

.. _erlang:

--------------
Erlang Related
--------------

1. Using Pool, Pool and Pool... Recommending GProc lib: https://github.com/uwiger/gproc

2. Asynchronism in mind, asynchronous, asynchronous message, asynchronous message between layers. Synchronism is only for load protection.

3. Avoiding of accumulation in Mailbox. Heavily loaded process uses gen_server2

4. Messages flowing through Socket and session process must utilize hibernate mechanism. Binary handles are to recovered.

5. Using binary data, avoiding memory copying / cloning between processes.

6. ETS, ETS, ETS...Message Passing Vs ETS

7. Avoiding ETS select and match on non-key fields

8. Avoiding massive ETS read/write, ETS R/W causes memory copying. Use lookup_element, update_counter

9. Properly open ETS table{write_concurrency, true}

10. Protecting Mnesia DB transaction reducing transaction number, avoiding transaction overload.

11. Avoidng Mnesia Table index, avoiding match and select on non-key fields

.. _eSockd: https://github.com/emqtt/esockd
.. _Chain-of-responsibility_pattern: https://en.wikipedia.org/wiki/Chain-of-responsibility_pattern

