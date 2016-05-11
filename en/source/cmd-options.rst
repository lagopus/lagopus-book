.. _ref_cmd-options:

Lagopus software switch command options and examples
=========================================================================

Lagopus vswitch command options can be classified into 3 groups.

* Global options
* Intel DPDK options
* Datapath options

Each option groups should be ordered and separated by ``--`` (double dash) like below.

.. code-block:: console

  $ sudo lagopus [global options] [-- [dpdk options] [-- datapath options] ]

  Example:
  $ sudo lagopus -l lagopus.log -- -c3 -n1 -- -p7
                 <-  global  ->    <-dpdk->   <-datapath->

Examples of how to use options are described in `Lagopus command line examples`_ later in this section.


Global option
-------------
*All mandatory options are marked (Mandatory). Others are optional.*

-d                     Run in debug mode
-v                     Show version
--version              Show version
-h                     Show help
--help                 Show help
-l filename            Specify a log/trace file path [default: syslog]
--logfile filename     Specify a log/trace file path [default: syslog]
-p filename            Specify a pid file path [default: /var/run/lt-lagopus.pid]
--pidfile filename     Specify a pid file path [default: /var/run/lt-lagopus.pid]
-C filename            Specify a configuration file (DSL format) path
--config filename      Specify a configuration file (DSL format) path

Intel DPDK Option
-----------------
*All mandatory options are marked (Mandatory). Others are optional.*

* ``-c`` : (Mandatory)

  * Speicify which CPU cores are assigned to Lagopus with a hex bitmask of CPU port to use
  * Example: ``-c9``

	* Only CPU core 0 and core 3 are assigned, then let bit0 and bit3 on.
	* 0x1001 (binary) = 9 (Hexadecimal)

* ``-n`` : (Mandatory)

  * Specify number of memory channel which CPU supports
  * Range: 1,2,4
  * Example: ``-n2``

	* a CPU supports dual memory channlel

* ``-m`` :

  * Specify how much Hugepage memory is assigned to Lagopus in mega byte.
  * Maximum hugepage memory is assigned if no option is specified.
  * Example: ``-m 512``

	* Use 512MB for Lagopus

* ``--socket-mem`` :

  * Specify how much Hugepage memory for each CPU socket is assigned to Lagopus in mega byte.
  * This option is exclusive with ``-m`` option

Datapath option
---------------
*All mandatory options are marked (Mandatory). Others are optional.*

Common option
^^^^^^^^^^^^^

* ``--no-cache`` :

  * Don't use flow cache [default: use flow cache]

* ``--fifoness TYPE`` :

  * Select packet ordering (FIFOness) mode [default: none]
  * ``none`` : FIFOness is disabled
  * ``port`` : FIFOness per each port.
  * ``flow`` : FIFOness per each flow.
  * Example: ``--fifoness flow``

	* Specify flow-level FIFOness

* ``--hashtype TYPE`` :

  * Select key-value store type for flow cache [default: intel64]
  * ``intel64`` : Hash with Intel CRC32 and XOR (64bit)
  * ``city64``  : CityHash (64bit)
  * ``murmur3`` : MurmurHash3 (32bit)
  * Example: ``--hashtype city64``

	* Use CityHash

* ``--kvstype TYPE`` :

  * Select key-value store type for flow cache [default: hashmap_nolock]

  * ``hashmap_nolock`` : Use hashmap without reader and writer lock
  * ``hashmap`` : Use hashmap with reader and writer lock
  * ``ptree``   : Use patricia tree
  * Example: ``--kvstype ptree``

	* Use patricia tree for key-value store

CPU core and packet processing
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Dataplane of Lagopus provides two options to assign CPU core and
packet processing worker.

* Automatic assignment option
* Explicit assignment option

These options are exclusive.

**Automatic assignment option**
"""""""""""""""""""""""""""""""

* ``-p PORTMASK`` : (Mandatory)

  * hexadecimal bitmask of ports to be used
  * Example: ``-p3``

	* Use port 0 and port 1

* ``-w NWORKERS``:

  * number of packet processing worker [default: assign as possible]
  * System assigns CPU core automatically.
  * Example: ``-w8``

	* Use total 8 cores for data plane

* ``--core-assign TYPE``:

  * Select automatically core assign policy [default: performance]
  * ``performance`` : Don't use HTT core. (e.g. use only 8 cores on 8C16T processor)
  * ``balance`` : Use HTT core (e.g. use 16 cores on 8C16T processor)
  * ``minimum`` : Use only one core.


**Explicit assignment option**
""""""""""""""""""""""""""""""

Current Lagopus limitation: youngest core number can not be specified among available CPU cores.

* ``--rx "(PORT, QUEUE, CORE),(PORT, QUEUE, CORE),...,(PORT, QUEUE, CORE)"``:

  * List NIC RX assignment policy of NIC RX port, queue, and CPU core with the combination of (PORT, QUEUE, CORE) and ","
  * ``PORT`` : Port number [0 - n]
  * ``QUEUE``: Queue number [default: 0]
  * ``CORE`` : CPU core number

  * Example: ``--rx '(0,0,2),(1,0,3)'``

    Assign port 0 to CPU core #2 and port 1 to CPU core #3

* ``--tx "(PORT,CORE),(PORT,CORE),...,(PORT,CORE)"``:

  * List NIC TX assignment policy of NIC TX port and CPU core with the combination of (PORT, CORE) and ","
  * ``PORT``: Port number [0 - n]
  * ``CORE``: CPU core number

  * Example: ``--tx '(0,4),(1,5)'``

    Assign port 0 TX to CPU core #4 and port 1 TX to CPU core #5

* ``--w "CORE, ..., CORE"``:

  * List of the CPU cores for packet processing with ","

    * Example: ``--w '8,9,10,11,12,13,14,15'``

      Assign CPU core 8 - 15 for packet processing


Lagopus command line examples
-----------------------------

Simple run
^^^^^^^^^^
* CPU core: 1 and 2
* Number of memory channel: 1
* NIC port: 0 and 1
* Packet processing workers are automatically assigned to CPU cores.
* Run in debug-mode

.. code-block:: console

  $ sudo lagopus -d -- -c3 -n1 -- -p3

Run with explicit assignment to achieve maximum performance
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* CPU core: 1 - 7

  * core # 2 for I/O RX
  * core # 3 for I/O TX
  * core # 4, # 5, #6, # 7 for packet processing

* Memory channel: 2
* NIC port: 0 and 1
* Run in debug-mode

.. code-block:: console

  $ sudo lagopus -d -- -cfe -n2 -- --rx '(0,0,2),(1,0,2)' --tx '(0,3),(1,3)' --w 4,5,6,7

Run with packet-ordering in flow-level
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* CPU core: 1 - 15
* Memory channel: 2
* NIC port: 0, 1, 2
* Worker assignment is automatic
* FIFOness option to ensure packet-ordering in flow-levle

.. code-block:: console

  $ sudo lagopus -- -cfffe -n2 -- -p7 --fifoness flow
