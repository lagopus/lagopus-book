.. -*- coding: utf-8; -*-

.. _ref_installation-dpdk:

Installation with DPDK
===============================

This section describes how to install DPDK-enabled Lagopus on Linux installed bear-metal server with DPDK-enabled NICs and its basic configuration. Lagopus software switch leverages DPDK for high-performance packet processing and forwarding.  DPDK [#]_ provide a set of high-performance network-related library and user-space polling-based drivers in order to accelerate network I/O and forwarding performance on top of PC servers with standard NICs. DPDK allows both packet processing and NIC device handling in user-space not in kernel-space, therefore, special configuration are required compared with the Lagopus with raw-socket configuration (none-DPDK configuration).

To enable DPDK configuration on Lagopus software switch, you have to configure DPDK environment before its execution. Usually Ethernet NIC are under the control of Linux kernel. Therefore you can see these NIC name by ``ip`` command, for example, ``ethX`` or ``emX``. DPDK-enabled NICs are supposed to be managed under the userspace DPDK process and userspace drivers. As a result, DPDK-enabled NICs are supposed to be out of the control of Linux kernel. Then you will not see the name of DPDK-controlled NICs by ``ip`` command.

DPDK-enabled Lagopus can run on a hypervisor-based virtual machine, such as KVM, VMware, and VirtualBox. Therefore, you can test DPDK-enabled Lagopus on your environment.
This installation steps are confirmed on VirtualBox VM, which should be identical to the steps on bear-metal server.
If you want to run Lagopus with raw-socket configuration instead of DPDK-enabled Lagopus, refer to :ref:`ref_installation-rawsocket`.

.. [#] For detail DPDK information, check http://dpdk.org/

Minimum hardware requirement
---------------------------------

Lagopus software switch with DPDK configuration uses multiple CPU cores to achieve high-performance network I/O and processing performance. Therefore, you have to prepare a host machine that meets the following minimum hardware requirements.

- # of CPU cores: **2 or more**
- Memory size: **2 GB or more**
- NIC: DPDK-enabled NICs

.. warning::
   **You need 2 or more processors (CPU cores) to run Lagopus software switch using DPDK**.



Software Versions
-----------------

* Lagopus : `Lagopus software switch 0.2.6 <https://github.com/lagopus/lagopus/releases/tag/v0.2.6>`_
* OS: `Ubuntu Server 16.04 LTS <http://www.ubuntu.com/download/server>`_
* DPDK: 2.2 or above


Installation steps
--------------------------

* Install Ubuntu to a bear-metal server or a VM.
* Install necessary packages.

  .. code-block:: console

    $ sudo apt-get update
    $ sudo apt-get install build-essential libexpat-dev libgmp-dev \
      libssl-dev libpcap-dev byacc flex git \
      python-dev python-pastedeploy python-paste python-twisted


* Download Lagopus source code.

  You have two options to get Lagopus source code by downloading tar file or by ``git``.

  .. code-block:: console

    $ wget https://github.com/lagopus/lagopus/archive/v0.2.6.tar.gz
    $ tar xvf v0.2.6.tar.gz
    $ ls
    lagopus-0.2.6  v0.2.6.tar.gz
    $

  Or

  .. code-block:: console

    $ git clone -b v0.2.6 --recursive https://github.com/lagopus/lagopus.git
    $ ls
    lagopus
    $

* Compile DPDK-enabled Lagopus software switch

  The source code of DPDK library is automatically download by git submodule mechanism. You don't have to download by manual.

  .. code-block:: console

     $ ./configure
     $ make


  .. note::

     If you find an error in configure process or make process, check git submodule setting. Sometimes git submodule operation may be failed due to firewall or proxy setting on your environment.

     .. code-block:: console

         /home/<usr>/lagopus-0.2.6/mk/make_dpdk.sh /home/<usr>/lagopus-0.2.6 src/dpdk \
                     "x86_64" "linuxapp" gcc
         fatal: Not a git repository (or any of the parent directories): .git
         make[1]: *** [dpdk] Error 1
         make[1]: Leaving directory `/home/<usr>/lagopus-0.2.3'
         make: *** [prerequisite] Error 2
         configure: error: Prerequisite failure.

* Install lagopus package

  .. code-block:: console

     $ sudo make install

Setup DPDK
----------------------

To enable DPDK on your environment, you have three steps, kernel module setup, hugepage setup, DPDK-enabled NIC assignment.
The above compilation process of Lagopus software switch, DPDK library and drivers are compiled automatically. Thus you just perform the following DPDK configurations.


Setup kernel module
^^^^^^^^^^^^^^^^^^^

DPDK provides two kernel modules, ``igb_uio`` and ``rte_kni``, to realize userspace NIC drivers and network packet processsing. The ``igb_uio`` is a wrapper module for DPDK-enabled NICs on top of UIO_ module of Linux kernel. The ``igb_uio`` allows userspace DPDK driver access to memory-mapped registers on NICs directly.
The ``rte_kni`` module enables packet frame exchanges between an user space DPDK application and network stack in kernel space.

.. _UIO: https://www.kernel.org/doc/htmldocs/uio-howto/about.html

* Load UIO and kernel modules.

  The kernel modules built are available in the ``src/dpdk/build/kmod`` directory.

    .. code-block:: console

       $ sudo modprobe uio
       $ cd lagopus
       $ sudo insmod ./src/dpdk/build/kmod/igb_uio.ko
       $ sudo insmod ./src/dpdk/build/kmod/rte_kni.ko
       $ lsmod | egrep 'uio|kni'
       rte_kni               282624  0
       igb_uio                16384  0
       uio                    20480  1 igb_uio
       $

    .. note::

       You have to perform the above steps above after the OS reboot.

Setup Hugepages
^^^^^^^^^^^^^^^^^^^^

Make hugepages available to DPDK. You can setup hugepage in two ways:

1. Manual configuration: Repeate steps after reboot if you select this.
2. Script configuration: Select this to keep it permanent after reboot.

.. note::

   When configured manually with , you need the same steps after rebooting OS.


1. Manual confguration (undone after reboot)

   Perform the following commands

   .. code-block:: console

      $ sudo sh -c "echo 256 >  /sys/devices/system/node/node0/hugepages/hugepages-2048kB/nr_hugepages"
      $ sudo mkdir -p /mnt/huge
      $ sudo mount -t hugetlbfs nodev /mnt/huge

2. Script configuration (permanent after reboot)

   * Reserve 256 pages of 2 MB hugepages in linux by adding the following line in  ``/etc/sysctl.conf``.

     .. code-block:: console

        $ sudo vi /etc/sysctl.conf
        vm.nr_hugepages = 256
        $

   * Enable permanent across reboots. Add a directory for hugepages and the following line to ``/etc/fstab`` so that mount point can be made permanent across reboots.

     .. code-block:: console

        $ sudo mkdir -p /mnt/huge
        $ sudo vi /etc/fstab
        nodev /mnt/huge hugetlbfs defaults 0 0

   * Confirm HugePages are configured correctly by the below commands.

     .. code-block:: console

        $ grep -i "HugePages" /proc/meminfo
        AnonHugePages:         0 kB
        HugePages_Total:     256
        HugePages_Free:      256
        HugePages_Rsvd:        0
        HugePages_Surp:        0
        Hugepagesize:       2048 kB
        $ mount | grep huge
        nodev on /mnt/huge type hugetlbfs (rw)
        $

NIC (Network Interface Card) assignment
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The following steps detatch the control and management of NICs, which will be used for DPDK application, from Linux kernel.

In this example, the host has three NICs, we are going to use ``eth1``, ``eth2`` for Lagopus software swtich.
Perform the following steps to enable NICs for DPDK application.

.. note::

   * You need to unbound NIC from kernel (ixgbe driver) before using it with DPDK.
   * You will lose connection to the OS if you unbound NIC used for management plane (ex: ssh).



* Check PCI ID of the NICs you want to use for DPDK with ``dpdk_nic_bind.py`` script.

  ``dpdk_nic_bind.py`` script displays DPDK-enabled NIC information, such as PCI ID and interface name in Linux kernel.

  .. code-block:: console

     $ cd lagopus
     $ sudo ./src/dpdk/tools/dpdk_nic_bind.py --status

     Network devices using DPDK-compatible driver
     ============================================
     <none>

     Network devices using kernel driver
     ===================================
     0000:00:03.0 '82540EM Gigabit Ethernet Controller' if=eth0 drv=e1000 unused= *Active*
     0000:00:08.0 '82545EM Gigabit Ethernet Controller (Copper)' if=eth1 drv=e1000 unused=
     0000:00:09.0 '82545EM Gigabit Ethernet Controller (Copper)' if=eth2 drv=e1000 unused=

     Other network devices
     =====================
     <none>
     $


  In this example, You can see PCI IDs of ``eth1`` and ``eth2`` from the output, ``0000:00:08.0`` and ``0000:00:09.0``.

  .. list-table:: NIC configuation before NIC unbound operation
     :header-rows: 1

     * - PCI ID
       - Linux IF name
       - Bounded by Linux
     * - 0000:00:03.0
       - eth0
       - Yes
     * - 0000:00:08.0
       - eth1
       - Yes
     * - 0000:00:09.0
       - eth2
       - Yes


* Unbound NICs from ixgbe driver and registerd with igb_uio driver.

   Perform the dpdk_nic_bind with the PCI IDs to be unbounded from Linux kernel.

   .. code-block:: console

       ~/lagopus$ sudo ./src/dpdk/tools/dpdk_nic_bind.py --bind=igb_uio 0000:00:08.0 0000:00:09.0
       ~/lagopus$ sudo ./src/dpdk/tools/dpdk_nic_bind.py --status

       Network devices using DPDK-compatible driver
       ============================================
       0000:00:08.0 '82545EM Gigabit Ethernet Controller (Copper)' drv=igb_uio unused=
       0000:00:09.0 '82545EM Gigabit Ethernet Controller (Copper)' drv=igb_uio unused=

       Network devices using kernel driver
       ===================================
       0000:00:03.0 '82540EM Gigabit Ethernet Controller' if=eth0 drv=e1000 unused=igb_uio *Active*

       Other network devices
       =====================
       <none>

* Memorize the DPDK NIC configuration

   After the unbound NICs from Linux kernel, the NIC which was bounded to ``eth1`` is accesssed by ``DPDK port #0`` or PCI ID (``0000:00:08.0``) directly. The NIC which was bounded by ``eth2`` is also accessed by ``DPDK port #1`` or PCI ID (``0000:00:09.0``).

  .. list-table:: NIC configuation after NIC unbound operation
     :header-rows: 1

     * - PCI ID
       - Linux IF name
       - Bounded by Linux
       - DPDK ready
       - DPDK port #
     * - 0000:00:03.0
       - eth0
       - Yes
       - No
       -
     * - 0000:00:08.0
       - eth1
       - No
       - Yes
       - 0
     * - 0000:00:09.0
       - eth2
       - No
       - Yes
       - 1

Setup Lagopus configuration file
----------------------------------------


Example Lagopus configuration (DSL format) can be found at "misc/examples/lagopus.dsl".
``lagopus.dsl`` file must be located at the same directory of the executable of ``lagopus``, or under ``/usr/local/etc/lagopus/``.

* Copy sample configuration file under ``/usr/local/etc/lagopus/``.

  .. code-block:: console

     $ sudo mkdir /usr/local/etc/lagopus/
     $ cd ~/lagopus-0.2.6
     $ sudo cp misc/examples/lagopus.dsl /usr/local/etc/lagopus/lagopus.dsl

* Edit configuration file suited to your environment.

  * Example:

    * One OpenFlow controller: "127.0.0.1"
    * ``eth0``: management interface. (Thus does not appear in the configuration)
    * ``DPDK port #0`` which was ``eth1`` and ``DPDK port #1`` which was ``eth2``: Lagopus dataplane ports. These two ports are accessed with DPDK.

    .. code-block:: console

       $ sudo vi /usr/local/etc/lagopus/lagopus.dsl
       channel channel01 create -dst-addr 127.0.0.1 -protocol tcp
       controller controller01 create -channel channel01 -role equal -connection-type main
       interface interface01 create -type ethernet-dpdk-phy -port-number 0
       interface interface02 create -type ethernet-dpdk-phy -port-number 1
       port port01 create -interface interface01
       port port02 create -interface interface02
       bridge bridge01 create -controller controller01 -port port01 1 -port port02 2 -dpid 0x1
       bridge bridge01 enable
       $

Running / Stopping Lagopus software switch
---------------------------------------------


DPDK command option
^^^^^^^^^^^^^^^^^^^^^^^^

In order to run Lagopus softwarwe switch with DPDK configuration, you need to specify DPDK-related options in ``lagopus`` command: which CPU cores are assigned to packet processing (``-c``), how many memory channels the host has (``-n``), which DPDK ports are used (``-p``).

In this example, the host has four CPU cores and two memory channel and you try to assign CPU core #1 and CPU core #0 for network I/O and processing and DPDK port #0 and DPDK port #1.
With ``-c`` option, you should specify which CPU cores are assigned to Lagopus software switch. The value of ``-c`` option uses the hexadecimal notation that its *N* -bit shows whehter CPU core # *N* is used or not for DPDK. The following table help your understanding of CPU flags.

.. list-table:: DPDK CPU flag
   :header-rows: 1

   * - cpu core # 3
     - cpu core # 2
     - cpu core # 1
     - cpu core # 0
     - flag in binary
     - flag in hexadecimal
   * - 0
     - 0
     - 1
     - 1
     - 0x0011
     - 0x3


With ``-p`` option, you should specify which DPDK ports are assigned to Lagopus software switch. The value of ``-p`` option also uses the hexadecimal notation that its *N* -bit shows whehter DPDK port # *N* is used or not for DPDK.

.. list-table:: DPDK NIC flag
   :header-rows: 1

   * - DPDK port # 1
     - DPDK port # 2
     - flag in binary
     - flag in hexadecimal
   * - 1
     - 1
     - 0x11
     - 0x3



Run Lagopus software switch
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* Peform the following command to run Lagopus software switch in foreground with debug mode.

  .. code-block:: console

     $ sudo lagopus -d -- -c3 -n2 -- -p3

Or

* Peform the following command to run Lagopus software switch in background.

  .. code-block:: console

     $ sudo lagopus -- -c3 -n2 -- -p3


Operation of Lagopus software switch with ``lagosh``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* Enter ``show version`` command from lagosh to confirm it's running.
* Enter ``stop`` command from lagosh to stop Lagopus vswitch.

  .. code-block:: console

     $ lagosh
     Lagosh> show version
     {
         "product-name": "Lagopus",
         "version": "0.2.6-release"
     }
     Lagosh> stop
     Lagosh> show version
     Socket connection refused.  Lagopus is not running?
     Lagosh> exit
     $

* For more configuration options, refer to :ref:`ref_cmd-options`
* For more information about lagosh, refer to :ref:`ref_lagosh`


.. note::

   Lagopus software switch with DPDK use pooling-based packet processing. Therefore you will see lagopus consume 100% of CPUs by  ``top`` command.

   .. code-block:: console

      top - 15:50:26 up 6 min,  1 user,  load average: 0.34, 0.13, 0.07
      Tasks:  83 total,   2 running,  81 sleeping,   0 stopped,   0 zombie
      %Cpu0  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
      %Cpu1  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
      KiB Mem:   3081312 total,   686804 used,  2394508 free,    20676 buffers
      KiB Swap:  3143676 total,        0 used,  3143676 free.    78792 cached Mem

        PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
       1205 root      20   0 1037728  16264   6764 S 100.2  0.5   0:25.58 lagopus
