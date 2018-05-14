.. _ref_architecture:

Lagopus software router architecture
====================================

To achive flexibilty, modularity and performance at same time, Lagopus software router is designed to be modular, and similar to web services built using microservice architecture, you can easily add (remove) modules to add (remove) feature to the router.

Lagopus software router consists of 3 components:

* Data Store and CLI (Openconfigd)
* Routing Agent (Zebra)
* Data Plane (vsw)

.. image:: images/lagopus-architecture.png

Data Store and CLI (Openconfigd)
--------------------------------

Lagopus software router uses an Open Source configuration manager [Openconfigd]_ as Data Store.
Openconfigd also provies CLI (Command Line Interface) so users can show status / stats and configure Lagopus software router.

.. [Openconfigd] https://github.com/coreswitch/openconfigd

 
Routing Agent (Zebra)
---------------------

Lagopus software router is designed so it could be used with various routing agent.
Currently it's confirmed to work with [Zebra]_ Open Source routing agent.

Lagopus software router will create tap interface when interface with IP address is configured. It will pass all packets designated to its address to corresponding tap interface. Thus Routing Agent could receive routing packet (OSPF, BGP etc) via tap interface.

Lagopus software router will retrive routing information via Netlink which Routing Agent will update when it learn any route changes.

.. [Zebra] https://github.com/coreswitch/zebra


Data Plane (vsw)
----------------

Lagopus software router dataplane, [vsw]_ , consists of multiple modules which has different functions (features) for processing packets. Example of modules currently available are Bridge (L2), Router (L3) and Tunnel.

Fast path is written using C utilizing [DPDK]_ to achive high performance packet processing.
Slow path is written using [Go]_ and uses [cgo]_ to call fast path C functions.

.. [vsw] https://github.com/lagopus/vsw
.. [DPDK] http://dpdk.org
.. [Go] https://golang.org/
.. [cgo] https://golang.org/cmd/cgo/

