.. _ref_debug:

Appendix: How to debug Lagopus software router
==============================================

This section describes how to debug issues of Lagopus software router.

Running vsw in verbose mode
---------------------------

The most simple way to start debugging is to run vsw in verbose mode.

To run vsw in verbose mode, use ``-v`` option when running vsw.

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

Building vsw with DEBUG flag
----------------------------

You can build Lagopus software router (vsw) with DEBUG flag to enable debug log following below steps.

.. code-block:: none


   $ cd ~/go/src/github.com/lagopus/vsw
   ~/go/src/github.com/lagopus/vsw$ export CGO_CFLAGS="-D DEBUG"
   ~/go/src/github.com/lagopus/vsw$ go build
   ~/go/src/github.com/lagopus/vsw$ go install
 

