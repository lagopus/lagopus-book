.. _ref_config-examples:

Example configurations of Lagopus router
========================================

In this section, we will introduce configuration examples of Lagopus router in various use cases.

* Layer 2 with multiple Vlans
* Layer 3 with multiple Vlans
* Layer 3 with multiple VRFs

You can find more examples under : https://github.com/lagopus/lagopus-router/tree/master/samples/


Layer 2 with multiple Vlans
---------------------------

* To configure multiple Vlans, create single (1) L2VSI network-instance with multiple vlans configured with ACTIVE status.

  .. code-block:: none

   set network-instances network-instance vsi1 config type L2VSI
   set network-instances network-instance vsi1 config enabled true
   set network-instances network-instance vsi1 vlans vlan 100 config status ACTIVE
   set network-instances network-instance vsi1 vlans vlan 200 config status ACTIVE

* subinterface should be assigned to the network-instance (``vsi1`` in this example).

  .. code-block:: none

   set network-instances network-instance vsi1 interfaces interface if0 subinterface 0
   set network-instances network-instance vsi1 interfaces interface if1 subinterface 0
   set network-instances network-instance vsi1 interfaces interface if2 subinterface 100
   set network-instances network-instance vsi1 interfaces interface if2 subinterface 200

* vlan-id should be configured for both interface and subinterface level. Example below shows interface and subinterface setting for both ACCESS and TRUNK interfaces.

  .. code-block:: none

   set interfaces interface if1 ethernet switched-vlan config interface-mode ACCESS
   set interfaces interface if1 ethernet switched-vlan config access-vlan 200
   set interfaces interface if1 subinterfaces subinterface 0 vlan config vlan-id 200
   
   set interfaces interface if2 ethernet switched-vlan config interface-mode TRUNK
   set interfaces interface if2 ethernet switched-vlan config trunk-vlans 100
   set interfaces interface if2 subinterfaces subinterface 100 vlan config vlan-id 100

.. raw:: latex

    \clearpage

Sample diagram and config
^^^^^^^^^^^^^^^^^^^^^^^^^

Refer to below diagram and config commands to configure Lagopus software router as Layer 2 switch with multiple Vlans.

.. code-block:: none

   +--------------------------------------------------------------+
   |                                                              |
   | +----------------------------------------------------------+ |
   | | VSI1                                                     | |
   | |  +-----------------+               +-----------------+   | |
   | |  |   Bridge        |               |   Bridge        |   | |
   | |  |   Vlan100       |               |   Vlan200       |   | |
   | |  +---+---------+---+               +----+--------+---+   | |
   | |      |         |                        |        |       | |
   | +----------------------------------------------------------+ |
   |        |         |                        |        |         |
   |        |         |    +-------------------+        |         |
   |        |         |    |                            |         |
   |        |         +-------------------+             |         |
   |        |              |              |             |         |
   |    +---+---+      +---+---+      +---+---+     +---+---+     |
   |    | if0.0 |      | if1.0 |      | if2.0 |     | if2.1 |     |
   |    +---+---+      +---+---+      +---+---+     +---+---+     |
   |        | VID:100      | VID:200      | VID:100     | VID:200 |
   |        |              |              +------+------+         |
   |        |              |                     |                |
   |     +--+--+        +--+--+               +--+--+             |
   |     | if0 |        | if1 |               | if2 |             |
   +-----+--+--+--------+--+--+---------------+--+--+-------------+
            |              |                     |
            |              |                     |
            +              +                     +
          untag           untag                 tagged
         vlan 100        vlan 200            vlan 100,200

.. code-block:: none

   set interfaces interface if0 config mtu 1514
   set interfaces interface if0 config driver dpdk
   set interfaces interface if0 config device 0000:00:08.0
   set interfaces interface if0 config type ethernetCsmacd
   set interfaces interface if0 ethernet switched-vlan config interface-mode ACCESS
   set interfaces interface if0 ethernet switched-vlan config access-vlan 100
   set interfaces interface if0 subinterfaces subinterface 0 config enabled true
   set interfaces interface if0 subinterfaces subinterface 0 vlan config vlan-id 100
   set interfaces interface if0 config enabled true
   
   set interfaces interface if1 config mtu 1514
   set interfaces interface if1 config driver dpdk
   set interfaces interface if1 config device 0000:00:09.0
   set interfaces interface if1 config type ethernetCsmacd
   set interfaces interface if1 ethernet switched-vlan config interface-mode ACCESS
   set interfaces interface if1 ethernet switched-vlan config access-vlan 200
   set interfaces interface if1 subinterfaces subinterface 0 vlan config vlan-id 200
   set interfaces interface if1 subinterfaces subinterface 0 config enabled true
   set interfaces interface if1 config enabled true
   
   set interfaces interface if2 config mtu 1518
   set interfaces interface if2 config driver dpdk
   set interfaces interface if2 config device 0000:00:0a.0
   set interfaces interface if2 config type ethernetCsmacd
   set interfaces interface if2 ethernet switched-vlan config interface-mode TRUNK
   set interfaces interface if2 ethernet switched-vlan config trunk-vlans 100
   set interfaces interface if2 ethernet switched-vlan config trunk-vlans 200
   set interfaces interface if2 subinterfaces subinterface 100 vlan config vlan-id 100
   set interfaces interface if2 subinterfaces subinterface 100 config enabled true
   set interfaces interface if2 config enabled true
   set interfaces interface if2 subinterfaces subinterface 200 vlan config vlan-id 200
   set interfaces interface if2 subinterfaces subinterface 200 config enabled true
   set interfaces interface if2 config enabled true
   
   # network-instnace vsi1
   set network-instances network-instance vsi1 config type L2VSI
   set network-instances network-instance vsi1 config enabled true
   set network-instances network-instance vsi1 vlans vlan 100 config status ACTIVE
   set network-instances network-instance vsi1 vlans vlan 200 config status ACTIVE
   set network-instances network-instance vsi1 fdb config mac-learning true
   set network-instances network-instance vsi1 fdb config mac-aging-time 300
   set network-instances network-instance vsi1 fdb config maximum-entries 3000
   set network-instances network-instance vsi1 interfaces interface if0 subinterface 0
   set network-instances network-instance vsi1 interfaces interface if1 subinterface 0
   set network-instances network-instance vsi1 interfaces interface if2 subinterface 100
   set network-instances network-instance vsi1 interfaces interface if2 subinterface 200


Layer 3 with multiple Vlans
---------------------------

Follow below steps to configure Layer 3 router with multiple Vlans.

* Create rifs and assign vlan-id with interface-mode ACCESS. Note that driver of rif interface is ``local`` while it was ``dpdk`` for physical interface.
* Assign IPv4 address to the rif which will be routers' own address.

.. code-block:: none

   set interfaces interface rif0 config driver local
   set interfaces interface rif0 ethernet switched-vlan config interface-mode ACCESS
   set interfaces interface rif0 ethernet switched-vlan config access-vlan 100
   set interfaces interface rif0 subinterfaces subinterface 0 vlan config vlan-id 100
   set interfaces interface rif0 subinterfaces subinterface 0 ipv4 addresses address 10.0.0.1 config prefix-length 24

* Assign rif subinterface to L2VSI network-instance.

.. code-block:: none

   set network-instances network-instance vsi1 interfaces interface rif0 subinterface 0

* Assign rif subinterface to L3VRF network-instance

.. code-block:: none

   set network-instances network-instance vrf1 interfaces interface rif0 subinterface 0

.. raw:: latex

    \clearpage

Sample diagram and config
^^^^^^^^^^^^^^^^^^^^^^^^^

Refer to below diagram and config commands to configure Lagopus software router as Layer 3 router with multiple Vlans.

.. code-block:: none


   +--------------------------------------------------------------+
   |                                                              |
   | +----------------------------------------------------------+ |
   | |                                                          | |
   | | VRF1                                                     | |
   | |                                                          | |
   | +-------------+---------------------------------+----------+ |
   |               |                                 |            |
   | +------+  +---+----+              +------+  +---+----+       |
   | | rif0 +--+ rif0.0 |              | rif1 +--+ rif1.0 |       |
   | +------+  +---+----+              +------+  +---+----+       |
   |               |                                 |            |
   | +----------------------------------------------------------+ |
   | | VSI1        |                                 |          | |
   | |  +----------+------+               +----------+------+   | |
   | |  |   Bridge        |               |   Bridge        |   | |
   | |  |   Vlan100       |               |   Vlan200       |   | |
   | |  +---+---------+---+               +----+--------+---+   | |
   | |      |         |                        |        |       | |
   | +----------------------------------------------------------+ |
   |        |         |                        |        |         |
   |        |         |    +-------------------+        |         |
   |        |         |    |                            |         |
   |        |         +-------------------+             |         |
   |        |              |              |             |         |
   |    +---+---+      +---+---+      +---+---+     +---+---+     |
   |    | if0.0 |      | if1.0 |      | if2.0 |     | if2.1 |     |
   |    +---+---+      +---+---+      +---+---+     +---+---+     |
   |        | VID:100      | VID:200      | VID:100     | VID:200 |
   |        |              |              +------+------+         |
   |        |              |                     |                |
   |     +--+--+        +--+--+               +--+--+             |
   |     | if0 |        | if1 |               | if2 |             |
   +-----+--+--+--------+--+--+---------------+--+--+-------------+
            |              |                     |
            |              |                     |
            +              +                     +
          untag           untag                 tagged
         vlan 100        vlan 200            vlan 100,200

.. code-block:: none

   set interfaces interface if0 config mtu 1514
   set interfaces interface if0 config driver dpdk
   set interfaces interface if0 config device 0000:00:08.0
   set interfaces interface if0 config type ethernetCsmacd
   set interfaces interface if0 ethernet switched-vlan config interface-mode ACCESS
   set interfaces interface if0 ethernet switched-vlan config access-vlan 100
   set interfaces interface if0 subinterfaces subinterface 0 config enabled true
   set interfaces interface if0 subinterfaces subinterface 0 vlan config vlan-id 100
   set interfaces interface if0 config enabled true
   
   set interfaces interface if1 config mtu 1514
   set interfaces interface if1 config driver dpdk
   set interfaces interface if1 config device 0000:00:09.0
   set interfaces interface if1 config type ethernetCsmacd
   set interfaces interface if1 ethernet switched-vlan config interface-mode ACCESS
   set interfaces interface if1 ethernet switched-vlan config access-vlan 200
   set interfaces interface if1 subinterfaces subinterface 0 vlan config vlan-id 200
   set interfaces interface if1 subinterfaces subinterface 0 config enabled true
   set interfaces interface if1 config enabled true
   
   set interfaces interface if2 config mtu 1518
   set interfaces interface if2 config driver dpdk
   set interfaces interface if2 config device 0000:00:0a.0
   set interfaces interface if2 config type ethernetCsmacd
   set interfaces interface if2 ethernet switched-vlan config interface-mode TRUNK
   set interfaces interface if2 ethernet switched-vlan config trunk-vlans 100
   set interfaces interface if2 ethernet switched-vlan config trunk-vlans 200
   set interfaces interface if2 subinterfaces subinterface 100 vlan config vlan-id 100
   set interfaces interface if2 subinterfaces subinterface 100 config enabled true
   set interfaces interface if2 config enabled true
   set interfaces interface if2 subinterfaces subinterface 200 vlan config vlan-id 200
   set interfaces interface if2 subinterfaces subinterface 200 config enabled true
   set interfaces interface if2 config enabled true
   
   set interfaces interface rif0 config mtu 1514
   set interfaces interface rif0 config driver local
   set interfaces interface rif0 config type ethernetCsmacd
   set interfaces interface rif0 ethernet switched-vlan config interface-mode ACCESS
   set interfaces interface rif0 ethernet switched-vlan config access-vlan 100
   set interfaces interface rif0 subinterfaces subinterface 0 config enabled true
   set interfaces interface rif0 subinterfaces subinterface 0 vlan config vlan-id 100
   set interfaces interface rif0 subinterfaces subinterface 0 ipv4 addresses address 10.0.0.1 config prefix-length 24
   set interfaces interface rif0 config enabled true
   
   set interfaces interface rif1 config mtu 1514
   set interfaces interface rif1 config driver local
   set interfaces interface rif1 config type ethernetCsmacd
   set interfaces interface rif1 ethernet switched-vlan config interface-mode ACCESS
   set interfaces interface rif1 ethernet switched-vlan config access-vlan 200
   set interfaces interface rif1 subinterfaces subinterface 0 config enabled true
   set interfaces interface rif1 subinterfaces subinterface 0 vlan config vlan-id 200
   set interfaces interface rif1 subinterfaces subinterface 0 ipv4 addresses address 10.1.0.1 config prefix-length 24
   set interfaces interface rif1 config enabled true
   
   # network-instance vsi1
   set network-instances network-instance vsi1 config type L2VSI
   set network-instances network-instance vsi1 config enabled true
   set network-instances network-instance vsi1 vlans vlan 100 config status ACTIVE
   set network-instances network-instance vsi1 vlans vlan 200 config status ACTIVE
   set network-instances network-instance vsi1 fdb config mac-learning true
   set network-instances network-instance vsi1 fdb config mac-aging-time 300
   set network-instances network-instance vsi1 fdb config maximum-entries 3000
   set network-instances network-instance vsi1 interfaces interface if0 subinterface 0
   set network-instances network-instance vsi1 interfaces interface if1 subinterface 0
   set network-instances network-instance vsi1 interfaces interface if2 subinterface 100
   set network-instances network-instance vsi1 interfaces interface if2 subinterface 200
   set network-instances network-instance vsi1 interfaces interface rif0 subinterface 0
   set network-instances network-instance vsi1 interfaces interface rif1 subinterface 0
   
   # network-instnace vrf1
   set network-instances network-instance vrf1 config type L3VRF
   set network-instances network-instance vrf1 config enabled true
   set network-instances network-instance vrf1 config enabled-address-families IPV4
   set network-instances network-instance vrf1 interfaces interface rif0 subinterface 0
   set network-instances network-instance vrf1 interfaces interface rif1 subinterface 0


Layer 3 with multiple VRFs
--------------------------

VRFs are used to separate routing table to allow IP address overlap amoung multiple routing domains. (ex: when multiple tenants are attached to the same router)

Follow below steps to configure Layer 3 router with multiple VRFs.

* Create interfaces with same IPv4 network address.

.. code-block:: none

   set interfaces interface if1 subinterfaces subinterface 0 ipv4 addresses address 10.0.0.1 config prefix-length 24
   set interfaces interface if2 subinterfaces subinterface 0 ipv4 addresses address 10.0.0.1 config prefix-length 24

* Create multiple VRFs and assign interfaces.

.. code-block:: none

   set network-instances network-instance vrf1 interfaces interface if2 subinterface 0
   set network-instances network-instance vrf2 interfaces interface if3 subinterface 0

.. raw:: latex

    \clearpage

Sample diagram and config
^^^^^^^^^^^^^^^^^^^^^^^^^

Refer to below diagram and config commands to configure Lagopus software router as Layer 3 router with multiple VRFs.

Note that two rifs, rif0.0/rif1.0, are also created and attached to vlan100 so you can test reachability via VRF1 and VRF2 from client attached to if0.

.. code-block:: none

   +-------------------------------------------------------------------+
   |                                                                   |
   | +----------------------------+     +----------------------------+ |
   | |                            |     |                            | |
   | |  VRF1                      |     |  VRF2                      | |
   | |                            |     |                            | |
   | +-------------+----+---------+     +--------------+-----+-------+ |
   |               |    |                              |     |         |
   |               |    |                +-------------+     |         |
   |               |    |                |                   |         |
   |               |    +------------------------+           |         |
   |               |                     |       |           |         |
   | +------+  +---+----+  +------+  +---+----+  |           |         |
   | | rif0 +--+ rif0.0 |  | rif1 +--+ rif1.0 |  |           |         |
   | +------+  +---+----+  +------+  +---+----+  |           |         |
   |               |                     |       |           |         |
   | +----------------------------------------+  |           |         |
   | | VSI1        |                     |    |  |           |         |
   | |  +----------+---------------------+--+ |  |           |         |
   | |  |   Bridge                          | |  |           |         |
   | |  |   Vlan100                         | |  |           |         |
   | |  +---+-------------------------------+ |  |           |         |
   | |      |                                 |  |           |         |
   | +----------------------------------------+  |           |         |
   |        |                                    |           |         |
   |    +---+---+                            +---+---+   +---+---+     |
   |    | if0.0 |                            | if1.0 |   | if2.0 |     |
   |    +---+---+                            +---+---+   +---+---+     |
   |        | VID:100                    VID:200 |           | VID:300 |
   |        |                                    |           |         |
   |        |                                    |           |         |
   |     +--+--+                              +--+--+     +--+--+      |
   |     | if0 |                              | if1 |     | if2 |      |
   +-----+--+--+------------------------------+--+--+-----+--+--+------+
            |                                    |           |
            |                                    |           |
            +                                    +           +
          untag                                untag       untag
         vlan 100                             vlan 200    vlan 300


.. code-block:: none

   set interfaces interface if0 config mtu 1514
   set interfaces interface if0 config driver dpdk
   set interfaces interface if0 config device 0000:00:08.0
   set interfaces interface if0 config type ethernetCsmacd
   set interfaces interface if0 ethernet switched-vlan config interface-mode ACCESS
   set interfaces interface if0 ethernet switched-vlan config access-vlan 100
   set interfaces interface if0 subinterfaces subinterface 0 vlan config vlan-id 100
   set interfaces interface if0 subinterfaces subinterface 0 config enabled true
   set interfaces interface if0 config enabled true
   
   set interfaces interface if1 config mtu 1514
   set interfaces interface if1 config driver dpdk
   set interfaces interface if1 config device 0000:00:09.0
   set interfaces interface if1 config type ethernetCsmacd
   set interfaces interface if1 ethernet switched-vlan config interface-mode ACCESS
   set interfaces interface if1 ethernet switched-vlan config access-vlan 200
   set interfaces interface if1 subinterfaces subinterface 0 vlan config vlan-id 200
   set interfaces interface if1 subinterfaces subinterface 0 ipv4 addresses address 10.0.0.1 config prefix-length 24
   set interfaces interface if1 subinterfaces subinterface 0 config enabled true
   set interfaces interface if1 config enabled true
   
   set interfaces interface if2 config mtu 1514
   set interfaces interface if2 config driver dpdk
   set interfaces interface if2 config device 0000:00:0a.0
   set interfaces interface if2 config type ethernetCsmacd
   set interfaces interface if2 ethernet switched-vlan config interface-mode ACCESS
   set interfaces interface if2 ethernet switched-vlan config access-vlan 300
   set interfaces interface if2 subinterfaces subinterface 0 vlan config vlan-id 300
   set interfaces interface if2 subinterfaces subinterface 0 ipv4 addresses address 10.0.0.1 config prefix-length 24
   set interfaces interface if2 subinterfaces subinterface 0 config enabled true
   set interfaces interface if2 config enabled true
   
   set interfaces interface rif0 config mtu 1514
   set interfaces interface rif0 config driver local
   set interfaces interface rif0 config type ethernetCsmacd
   set interfaces interface rif0 ethernet switched-vlan config interface-mode ACCESS
   set interfaces interface rif0 ethernet switched-vlan config access-vlan 100
   set interfaces interface rif0 subinterfaces subinterface 0 config enabled true
   set interfaces interface rif0 subinterfaces subinterface 0 vlan config vlan-id 100
   set interfaces interface rif0 subinterfaces subinterface 0 ipv4 addresses address 192.168.0.1 config prefix-length 24
   set interfaces interface rif0 config enabled true
   
   set interfaces interface rif1 config mtu 1514
   set interfaces interface rif1 config driver local
   set interfaces interface rif1 config type ethernetCsmacd
   set interfaces interface rif1 ethernet switched-vlan config interface-mode ACCESS
   set interfaces interface rif1 ethernet switched-vlan config access-vlan 100
   set interfaces interface rif1 subinterfaces subinterface 0 config enabled true
   set interfaces interface rif1 subinterfaces subinterface 0 vlan config vlan-id 100
   set interfaces interface rif1 subinterfaces subinterface 0 ipv4 addresses address 192.168.0.2 config prefix-length 24
   set interfaces interface rif1 config enabled true
   
   # network-instnace vsi1
   set network-instances network-instance vsi1 config type L2VSI
   set network-instances network-instance vsi1 config enabled true
   set network-instances network-instance vsi1 vlans vlan 100 config status ACTIVE
   set network-instances network-instance vsi1 fdb config mac-learning true
   set network-instances network-instance vsi1 fdb config mac-aging-time 300
   set network-instances network-instance vsi1 fdb config maximum-entries 3000
   set network-instances network-instance vsi1 interfaces interface if0 subinterface 0
   set network-instances network-instance vsi1 interfaces interface rif0 subinterface 0
   set network-instances network-instance vsi1 interfaces interface rif1 subinterface 0
   
   # network-instnace vrf1
   set network-instances network-instance vrf1 config type L3VRF
   set network-instances network-instance vrf1 config enabled true
   set network-instances network-instance vrf1 config enabled-address-families IPV4
   set network-instances network-instance vrf1 interfaces interface rif0 subinterface 0
   set network-instances network-instance vrf1 interfaces interface if1 subinterface 0
   
   # network-instnace vrf2
   set network-instances network-instance vrf2 config type L3VRF
   set network-instances network-instance vrf2 config enabled true
   set network-instances network-instance vrf2 config enabled-address-families IPV4
   set network-instances network-instance vrf2 interfaces interface rif1 subinterface 0
   set network-instances network-instance vrf2 interfaces interface if2 subinterface 0

