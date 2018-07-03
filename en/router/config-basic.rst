.. _ref_config-basic:

Configuring and running Lagopus software router
===============================================

This section describes how to configure and run Lagopus software router using two examples, running Lagopus software router as bridge (vlan) and router.

All examples in this section are based on VirtualBox VM having below two ethernet ports and DPDK 17.11.1 installed.
You should change port name / PCIe ``domain:bus:slot.func`` matching to your environment.

* enp0s8 (0000:00:08.0)
* enp0s9 (0000:00:09.0)

Configuration files
-------------------

Running Lagopus software router would require running both openconfigd (configuration manager) and vsw (virtual switch dataplane). Default configuration files read during boot time and its locations are as follows.

* openconfigd: /usr/local/etc/openconfigd.conf
* vsw: /usr/local/etc/vsw.conf

About openconfigd.conf
^^^^^^^^^^^^^^^^^^^^^^

You could copy and customize sample ``*.conf`` files under below GitHub repo to start openconfigd.

* https://github.com/lagopus/lagopus-router/tree/master/samples

However, openconfigd does NOT require any configuration files during start up. You could use blank openconfigd.conf file or use ``-z`` option to start openconfigd with no configuration and later set / commit configuration via CLI.

Refer to examples later in this section for openconfigd CLI commands.

About vsw.conf
^^^^^^^^^^^^^^

Valid vsw.conf is requred to start vsw. (Unlike openconfigd which does not require config file)
As you can see in below default vsw.conf configuration file, vsw configuration is sparated with ``[section]`` following with parameters related to the section.

* DPDK section

  - Set DPDK related parameters here.
  - For example, change core_mask to 0x0e if you want to use core 1,2,3.

* Openconfigd section

  - Set host and port.
  - Typically you can keep default value.

* Other sections

  - Set core(s) to be used by each module.

.. code-block:: none

   # VSW configuration file
   
   # DPDK configuration section
   [dpdk]
   core_mask = 0xfe
   # core_list = "1,2,3,4,5,6,7"
   memory_channel = 2
   pmd_path = "/usr/local/lib"
   num_elements = 131072
   cache_size = 256
   
   # Openconfigd section
   [openconfig]
   server_host = "localhost"   # Openconfigd server host
   server_port = 2650      # Openconfigd server port
   listen_port = 2653      # Port to listen for show command
   
   # ethdev configuration section
   [ethdev]
   rx_core = 2 # Slave core to use for RX
   tx_core = 3 # Slave core to use for TX
   
   # bridge configuration section
   [bridge]
   core = 2
   
   # RIF configuration section
   [rif]
   core = 3
   
   # tunnel configuration section
   [tunnel]
     # IP in IP tunnel
     [tunnel.ipip]
     inbound_core = 2
     outbound_core = 3
   
   # router configuration section
   [router]
   core = 3


Running Lagopus software router
-------------------------------

When running Lagopus software router, you should running Openconfigd and followed by vsw.

Running Openconfigd
^^^^^^^^^^^^^^^^^^^

YANG file should be specified when running Openconfigd. Typical way to do so is to move to ``yang`` directory and start openconfigd.

.. code-block:: none

   $ cd ~/go/src/github.com/lagopus/lagopus-router/yang
   ~/go/src/github.com/lagopus/lagopus-router/yang$ openconfigd -y modules:modules/policy:modules/bgp:modules/interfaces:modules/local-routing:modules/vlan:modules/rib:modules/network-instance:modules/types lagopus-router.yang

You can use ``-y`` option to speficy YANG path. Other valid OPTIONS could be shown specifying ``-h`` option.

.. code-block:: none

   # openconfigd --help
   Starting openconfigd.
   Usage:
     openconfigd [OPTIONS]
   
   Application Options:
     -c, --config-file= active config file name (default: openconfigd.conf)
     -p, --config-dir=  config file directory (default: /usr/local/etc)
     -y, --yang-paths=  comma separated YANG load path directories
     -2, --two-phase    two phase commit
     -z, --zero-config  Do not save or load config other than openconfigd.conf
   
   Help Options:
     -h, --help         Show this help message

Running vsw
^^^^^^^^^^^

DPDK related configurations are required every time you run vsw after reboot following below steps.

* set nr_hugepages
* install kernel modules (driver)
* detach / attach interfaces to DPDK driver

.. code-block:: none

   $ sudo -s
   # echo 1024 > /sys/devices/system/node/node0/hugepages/hugepages-2048kB/nr_hugepages
   # cd ~/dpdk-stable-17.11.1
   ~/dpdk-stable-17.11.1# modprobe uio
   ~/dpdk-stable-17.11.1# insmod build/kmod/igb_uio.ko
   ~/dpdk-stable-17.11.1# ip link set enp0s8 down
   ~/dpdk-stable-17.11.1# ip link set enp0s9 down
   ~/dpdk-stable-17.11.1# ./usertools/dpdk-devbind.py --bind=igb_uio enp0s8 enp0s9

Confirm two interfaces show up under "Network devices using DPDK-compatible driver".

.. code-block:: none

   ~/dpdk-stable-17.11.1# ./usertools/dpdk-devbind.py -s
   
   Network devices using DPDK-compatible driver
   ============================================
   0000:00:08.0 '82540EM Gigabit Ethernet Controller 100e' drv=igb_uio unused=e1000
   0000:00:09.0 '82540EM Gigabit Ethernet Controller 100e' drv=igb_uio unused=e1000

Start vsw.

.. code-block:: none

   $ sudo -s
   # DPDKDIR=~/dpdk-stable-17.11.1
   # env LD_LIBRARY_PATH=$DPDKDIR/build/lib vsw

You can add below options when starting vsw to specify Config file and/or to show logs while vsw running.

.. code-block:: none

   # vsw -h
   Usage of vsw:
     -f string
           Config file (default "/usr/local/etc/vsw.conf")
     -v    verbose mode

Example output when running vsw in verbose mode.

.. code-block:: none

   root@lagopus:~# env LD_LIBRARY_PATH=$DPDKDIR/build/lib vsw -v
   EAL: Detected 4 lcore(s)
   EAL: Probing VFIO support...
   EAL: PCI device 0000:00:03.0 on NUMA socket -1
   EAL:   Invalid NUMA socket, default to 0
   EAL:   probe driver: 8086:100e net_e1000_em
   EAL: PCI device 0000:00:08.0 on NUMA socket -1
   EAL:   Invalid NUMA socket, default to 0
   EAL:   probe driver: 8086:100e net_e1000_em
   EAL: PCI device 0000:00:09.0 on NUMA socket -1
   EAL:   Invalid NUMA socket, default to 0
   EAL:   probe driver: 8086:100e net_e1000_em
   EAL: PCI device 0000:00:0a.0 on NUMA socket -1
   EAL:   Invalid NUMA socket, default to 0
   EAL:   probe driver: 8086:100e net_e1000_em
   2018/05/09 02:20:46 core list: map[2:{false } 3:{false }]
   2018/05/09 02:20:46 ocdclient: Connect to "localhost:2650"
   2018/05/09 02:20:46 Agent Netlink Agent started.
   2018/05/09 02:20:46 Agent config started.

Using CLI
---------

To start CLI, enter "cli" after running openconfigd and vsw.
CLI has two modes, "Exec" and "Config".

.. code-block:: none

   $ cli
   lagopus>

Enter ``?`` to list available commands.

.. code-block:: none

   lagopus>
   Exec commands:
      exit                 End current mode and down to previous mode
      help                 Description of the interactive help system
      logout               Exit from EXEC
      quit                 End current mode and down to previous mode
      show                 Show running system information
      configure            Manipulate software configuration information

Enter ``configure`` to move to "Config" mode.

.. code-block:: none

   lagopus>configure
   lagopus#

Follow below steps to enter and apply configurtion:

* ``set`` to enter configuration

You can enter ``?`` any time to see possible config options.

.. code-block:: none

   lagopus#set interfaces interface if9 config <= enter ?
      description
      device
      driver
      enabled
      mtu
      type

* ``show`` to check current configuration.

``+`` is shown in front of config which is not applied yet.

.. code-block:: none

   +    interface if9 {
   +        config {
   +            enabled true;
   +        }
   +    }

* ``commit`` to apply configuration 

.. raw:: latex

    \clearpage

Sample Config (bridge)
----------------------

Follow below steps to configure bridge.

* Configure interface with interface-mode specified (ACCESS/TRUNK)
* Configure subinterface with vlan-id
* Configure network-instance of type L2VFI
* Configure vlans in network-instance
* Assign interface to network-instance

For example, ``set`` below config and ``commit`` to configure Lagopus software router in below diagram.
Note that ``device`` is specified in PCIe ``domain:bus:slot.func`` format. 

.. code-block:: none

   +---------------------------+
   |                           |
   | +-----------------------+ |
   | |                       | |
   | |   Bridge (Vlan 100)   | |
   | |                       | |
   | +----+-------------+----+ |
   |      |             |      |
   |   +--+--+       +--+--+   |
   |   | if0 |       | if1 |   |
   +---+--+--+-------+--+--+---+
          |             |
          +             +
        untag         untag
       vlan 100      vlan 100

.. code-block:: none

  set interfaces interface if0 config mtu 1514
  set interfaces interface if0 config driver dpdk
  set interfaces interface if0 config device "0000:00:08.0"
  set interfaces interface if0 config type ethernetCsmacd
  set interfaces interface if0 ethernet switched-vlan config interface-mode ACCESS
  set interfaces interface if0 ethernet switched-vlan config access-vlan 100
  set interfaces interface if0 subinterfaces subinterface 0 config enabled true
  set interfaces interface if0 subinterfaces subinterface 0 vlan config vlan-id 100
  set interfaces interface if0 config enabled true
  
  set interfaces interface if1 config mtu 1514
  set interfaces interface if1 config driver dpdk
  set interfaces interface if1 config device "0000:00:09.0"
  set interfaces interface if1 config type ethernetCsmacd
  set interfaces interface if1 ethernet switched-vlan config interface-mode ACCESS
  set interfaces interface if1 ethernet switched-vlan config access-vlan 100
  set interfaces interface if1 subinterfaces subinterface 0 config enabled true
  set interfaces interface if1 subinterfaces subinterface 0 vlan config vlan-id 100
  set interfaces interface if1 config enabled true
  
  set network-instances network-instance vsi1 config type L2VSI
  set network-instances network-instance vsi1 config enabled true
  set network-instances network-instance vsi1 vlans vlan 100 config status ACTIVE
  set network-instances network-instance vsi1 fdb config mac-learning true
  set network-instances network-instance vsi1 fdb config mac-aging-time 300
  set network-instances network-instance vsi1 fdb config maximum-entries 3000
  set network-instances network-instance vsi1 interfaces interface if0 subinterface 0
  set network-instances network-instance vsi1 interfaces interface if1 subinterface 0

.. raw:: latex

    \clearpage

Sample Config (router)
----------------------

Follow below steps to configure router.

* Configure interface with IPv4 address
* Configure network-instance of type L3VRF
* Assign interface to network-instance

For example, ``set`` below config and ``commit`` to configure Lagopus software router in below diagram.

.. code-block:: none

   +---------------------------+
   |                           |
   | +-----------------------+ |
   | |                       | |
   | |       L3 Routing      | |
   | |                       | |
   | +----+-------------+----+ |
   |      |             |      |
   |   +--+--+       +--+--+   |
   |   | if0 |       | if1 |   |
   +---+--+--+-------+--+--+---+
          |             |
          +             +
        untag         untag

.. code-block:: none

  set interfaces interface if0 config device "0000:00:08.0"
  set interfaces interface if0 config driver dpdk
  set interfaces interface if0 config enabled true
  set interfaces interface if0 config mtu 1514
  set interfaces interface if0 config type ethernetCsmacd
  set interfaces interface if0 subinterfaces subinterface 0 config enabled true
  set interfaces interface if0 subinterfaces subinterface 0 ipv4 addresses address 172.16.110.1 config prefix-length 24
  
  set interfaces interface if1 config device "0000:00:09.0"
  set interfaces interface if1 config driver dpdk
  set interfaces interface if1 config enabled true
  set interfaces interface if1 config mtu 1514
  set interfaces interface if1 config type ethernetCsmacd
  set interfaces interface if1 subinterfaces subinterface 0 config enabled true
  set interfaces interface if1 subinterfaces subinterface 0 ipv4 addresses address 10.10.0.1 config prefix-length 24
  
  set network-instances network-instance vrf1 config type L3VRF
  set network-instances network-instance vrf1 config enabled-address-families IPV4
  set network-instances network-instance vrf1 interfaces interface if0 subinterface 0
  set network-instances network-instance vrf1 interfaces interface if1 subinterface 0
  set network-instances network-instance vrf1 config enabled true

