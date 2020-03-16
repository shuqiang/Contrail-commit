The below gives overview of different modules invovlved

# Contrail-Config
## The below gives overview of different modules invovlved

   +--------------------------------------------------------------------------+
   |                                                                          |
   |                                                                          |
   |                                                                          |
   |                                     +--------------------------+         |         +-------------+
   |                                     |      authentication      |<----------------->|  Keystone   |
   |                                     +-------------^------------+         |         +-------------+
   |                                                   |                      |
   |                                                   |                      |
   |    +----------------------+         +-------------v------------+         |         +-------------+
   |    | VirtualNetworkServer |<------->|         http/rest        |<----------------->|  REST API   |
   |    +-----------^----------+         +--------------------------+         |         +-------------+
   |       ip_alloc |                                                         |
   |                |                                                         |
   |                | ip_alloc_req                                            |
   |    +-----------v----------+                                              |
   |    |       AddrMgmt       |                                              |
   |    +----------------------+                                              |
   |                                                                          |
   |                                                                          |
   |                                     +--------------------------+         |         +-------------+
   |                               +---->|        VncZkClient       |<----------------->|Zookeeper    |
   |                               |     +--------------------------+         |         +-------------+
   |                               |                                          |
   |service data diagram           |                                          |
   |-------------------------------+------------------------------------------|------------------------
   |                               |                                          |
   |                               |     +--------------------------+         |         +-------------+
   |                               +---->| VncServerCassandraClient |<----------------->|Cassandra    |
   |                               |     +--------------------------+         |         +-------------+
   |                               |                                          |
   |-------------------------------+------------------------------------------|------------------------
   |technology data diagram        |                                          |
   |                               |     +--------------------------+         |         +-------------+
   |                               +---->|    VncServerKombuClient  |<----------------->|RabbitMQ     |
   |                                     +--------------------------+         |         +-------------+
   |                                                                          |
   |                                                                          |
   |                                                                          |
   |                                                                          |
   +--------------------------------------------------------------------------+

VNC Config Api Server:
---------------------
VNC configure API server manages interaction
between http/rest, address management, authentication and database interfaces.

    Http/rest:
    ---------
    	Python bottle web

    	HTTP:vn_ip_alloc_http_post ---> VirtualNetworkServer:ip_alloc ---> AddrMgmt:ip_alloc_req
            uuid|
                |fq_name
                v
      	VncCassandraClient:uuid_to_fq_name

    Address management:
    ------------------

    Authentication:
    --------------
    If the API server is setup to use keystone for authentication (common for OpenStack deployments), then users must first obtain a token using keystone command line client or keystone’s REST interface. Then, this token should be sent in the header of the request. If authentication is disabled, then this auth token is not needed.

    Database interface:
    ------------------


++++++++++++++++++
    for example:
++++++++++++++++++

add virtual network:
-------------------
REST API----->http/rest----->authentication/keystone----->
