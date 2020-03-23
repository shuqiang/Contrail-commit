EVPN-type5
NLRI（Network Layer Reachability Information）

Type-2 route, MAC with IP advertisement route—Type-2 routes are per-VLAN routes, so only PEs that are part of a VNI need these routes. EVPN allows an end host’s IP and MAC addresses to be advertised within the EVPN Network Layer reachability information (NLRI). This allows for control plane learning of ESI MAC addresses. Because there are many type-2 routes, a separate route-target auto-derived per VNI helps to confine their propagation. This route type is supported by all EVPN switches and routers.

A MAC/IP Advertisement route type specific EVPN NLRI consists of the
   following:

                +---------------------------------------+
                |  RD (8 octets)                        |
                +---------------------------------------+
                |Ethernet Segment Identifier (10 octets)|
                +---------------------------------------+
                |  Ethernet Tag ID (4 octets)           |
                +---------------------------------------+
                |  MAC Address Length (1 octet)         |
                +---------------------------------------+
                |  MAC Address (6 octets)               |
                +---------------------------------------+
                |  IP Address Length (1 octet)          |
                +---------------------------------------+
                |  IP Address (0, 4, or 16 octets)      |
                +---------------------------------------+
                |  MPLS Label1 (3 octets)               |
                +---------------------------------------+
                |  MPLS Label2 (0 or 3 octets)          |
                +---------------------------------------+


Type-5 route, IP prefix Route—An IP prefix route provides encoding for inter-subnet forwarding. In the control plane, EVPN type-5 routes are used to advertise IP prefixes for inter-subnet connectivity across data centers. To reach a tenant using connectivity provided by the EVPN type-5 IP prefix route, data packets are sent as Layer 2 Ethernet frames encapsulated in the VXLAN header over the IP network across the data centers.

An IP Prefix Route Type for IPv4 has the Length field set to 34 and
   consists of the following fields:


    +---------------------------------------+
    |      RD   (8 octets)                  |
    +---------------------------------------+
    |Ethernet Segment Identifier (10 octets)|
    +---------------------------------------+
    |  Ethernet Tag ID (4 octets)           |
    +---------------------------------------+
    |  IP Prefix Length (1 octet, 0 to 32)  |
    +---------------------------------------+
    |  IP Prefix (4 octets)                 |
    +---------------------------------------+
    |  GW IP Address (4 octets)             |
    +---------------------------------------+
    |  MPLS Label (3 octets)                |
    +---------------------------------------+
