.. _ref_introduction:

Introduction to Lagopus software router
=======================================================

Lagopus software router is a yet another DPDK based software router implementation.

It has loose coupled modular-based design (like a microservice architecture) with unified configuration datastore defined by yang. Dataplane fast path is written in C + DPDK and slow path written in Go to achive both high performance and extensibility.

Hardware requirement
------------------------

Lagopus software router can run on Intel PC servers with NIC as well as Intel Desktop PC.

* CPU: >= 4 cores (Intel Xeon, Core, Atom)
* NIC: `DPDK Supported NIC <http://dpdk.org/doc/nics>`_
* Memory: >= 4GB

Lagopus can also run on a virtual machine on `Oracle VM VirtualBox <https://www.virtualbox.org/>`_, `KVM <http://www.linux-kvm.org/>`_, `VMware Workstation, Fusion and ESXi <http://www.vmware.com>`_, Microsoft Hyper-V.
Instruction of how to setup a virtual machine running on Oracl VM VirtualBox for Lagopus software router installation is described in :ref:`ref_virtualbox`.


Software requirement
------------------------

Lagopus software router can run on Linux distribution below.

* Linux

  * Ubuntu 18.04

Lagopus software router require DPDK.
Refer to :ref:`ref_installation-dpdk` for supported DPDK release and installation steps.

Development & Support
---------------------

Lagopus software router is an open source project and development is done on GitHub repo [lagopus-router]_. Please open an Issue or Pull Request for feedback.

.. [lagopus-router] https://github.com/lagopus/lagopus-router

License
---------

The code of Lagopus software switch is freely available under the Apache version 2.0
