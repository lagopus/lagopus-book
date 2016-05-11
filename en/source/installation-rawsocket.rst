:lagopus-version: 0.2.6

.. _ref_installation-rawsocket:

Installation with raw-socket configuration
==================================================================================

This section describes how to install Lagopus on Ubuntu installed a bear-metal server and basic configuration using raw-socket mode. This Lagopus configuration is designed for a test of OpenFlow controller application. This configuration does not require special hardware nor DPDK NICs, therefore, you can install any environment on a bare-metal server and a virtual machine [#]_.
If you want to enable "DPDK" for performance and scalability, please refer to :ref:`ref_installation-dpdk`.

.. [#] Actual steps are confirmed on VMware Fusion and VirtualBox VM, which should be identical to the steps on bear-metal server.


Required software versions
--------------------------------
* Lagopus: `Lagopus software switch 0.2.6`_
* Linux distribution: `Ubuntu Server 16.04 LTS`_

.. _Lagopus software switch 0.2.6: https://github.com/lagopus/lagopus/releases/tag/v0.2.6
.. _Ubuntu Server 16.04 LTS: http://www.ubuntu.com/download/server


Lagopus installation steps
--------------------------
* Install Linux to a bear-metal server or a VM.
* Install prerequisite software packages.

  .. code-block:: console

     $ sudo apt-get update
     $ sudo apt-get install build-essential libexpat-dev libgmp-dev \
                  libssl-dev libpcap-dev byacc flex git \
                  python-dev python-pastedeploy python-paste python-twisted \
                  wget


* Download source code.

  .. code-block:: console

     $ wget https://github.com/lagopus/lagopus/archive/v0.2.6.tar.gz
     $ tar xvf v0.2.6.tar.gz
     $ ls
     lagopus-0.2.6  v0.2.6.tar.gz
     $

  or

  .. code-block:: console

     $ git clone --depth=1 -b v0.2.6 https://github.com/lagopus/lagopus.git
     $ ls
     lagopus
     $


* Configure and compile Lagopus software switch with raw-socket mode option.

  .. code-block:: console

     $ cd lagopus-0.2.6
     $ ./configure --disable-dpdk
     $ make

* Install Lagopus software switch


  .. code-block:: console

     $ sudo make install

Setup Lagopus configuration file
--------------------------------
Example Lagopus configuration (DSL format) can be found at "misc/examples/lagopus.dsl".
``lagopus.dsl`` file must be located at the same directory of the executable of lagopus software switch, or under ``/usr/local/etc/lagopus/``.

* Copy sample configuration file under ``/usr/local/etc/lagopus/``.

  .. code-block:: console

     $ sudo mkdir /usr/local/etc/lagopus/
     $ cd ~/lagopus-0.2.6
     $ sudo cp misc/examples/lagopus.dsl /usr/local/etc/lagopus/lagopus.dsl

* Edit configuration file suited to your environment.

  * Example:

    * One OpenFlow controller: "127.0.0.1"
    * ``eth0``:  Management interface. (Thus does not appear in the configuration)
    * ``eth1``, ``eth2`` are used as Lagopus dataplane port. These two port are accessed with raw-socket mode.

    .. code-block:: console

       $ sudo vi /usr/local/etc/lagopus/lagopus.dsl
       channel channel01 create -dst-addr 127.0.0.1 -protocol tcp
       controller controller01 create -channel channel01 -role equal -connection-type main
       interface interface01 create -type ethernet-rawsock -device eth1
       interface interface02 create -type ethernet-rawsock -device eth2
       port port01 create -interface interface01
       port port02 create -interface interface02
       bridge bridge01 create -controller controller01 -port port01 1 -port port02 2 -dpid 0x1
       bridge bridge01 enable
       $

Running Lagopus software switch
---------------------------------------


Enter ``$ sudo lagopus`` to run Lagopus software switch. The basic process management of Lagopus software switch is done by lagopus command line [#]_ .

After the execution of Lagopus software switch, its operation and management are executed by ``lagosh`` [#]_ that is CLI tool for network operators. ``lagosh`` command line syntax is very similar to the existing network equipment.
Enter ``show version`` command to confirm it's running or not.
Enter ``stop`` command from lagosh to stop Lagopus software switch.



.. code-block:: console

  $ sudo lagopus
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

.. [#] For more configuration options, refer to :ref:`ref_cmd-options`
.. [#] For more information about lagosh, refer to :ref:`ref_lagosh`
