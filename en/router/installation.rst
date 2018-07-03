.. _ref_installation:

How to install Lagopus software router
======================================

Follow below steps to install Lagopus software router.

1. Install DPDK
2. Install Go
3. Install openconfigd and cli
4. Install vsw
5. Download lagopus-router

Install DPDK
------------

Refer to :ref:`ref_installation-dpdk` for steps to install DPDK.


Install Go
----------

* Install Go

  On Ubuntu 18.04 LTS, Go can be installed using apt.

  .. code-block:: none


   $ sudo apt install golang-go
   $ go version
   go version go1.10.1 linux/amd64

* Install dep

  "dep" is used to download and install dependencies.

  .. code-block:: none


   $ sudo apt install go-dep

* Configure Go related path

  .. code-block:: none


   $ vi ~/.bashrc
   export GOPATH=$HOME/go
   export PATH=$GOPATH/bin:$PATH

* Create Go related directories

  .. code-block:: none


   ~$ mkdir -p go/src
   ~$ mkdir -p go/pkg
   ~$ mkdir -p go/bin


Install openconfigd and cli
---------------------------

https://github.com/coreswitch/openconfigd/

Follow below steps written in README.md of upstream openconfigd repo [openconfigd/README.md]_

.. [openconfigd/README.md] https://github.com/coreswitch/openconfigd/blob/master/README.md 

.. code-block:: none


   $ go get github.com/coreswitch/openconfigd/openconfigd
   $ go get github.com/coreswitch/openconfigd/cli_command
   $ cd $GOPATH/src/github.com/coreswitch/openconfigd/cli
   ./configure
   make
   sudo make install
   $ cd $GOPATH/src/github.com/coreswitch/openconfigd/bash_completion.d
   sudo cp cli /etc/bash_completion.d/


Install vsw
-----------

https://github.com/lagopus/vsw

* Create lagopus directory
* git clone vsw repo.
* Install dependencies using "dep"
* Install vsw 

.. code-block:: none


   $ mkdir -p go/src/github.com/lagopus
   $ cd go/src/github.com/lagopus
   ~/go/src/github.com/lagopus$ git clone https://github.com/lagopus/vsw.git
   ~/go/src/github.com/lagopus/vsw$ dep ensure
   ~/go/src/github.com/lagopus/vsw$ go install


Download lagopus-router
-----------------------

https://github.com/lagopus/lagopus-router

Lagopus router repo has "YANG" files required to run openconfigd with Lagopus software router. Git clone it so you can use it when running openconfigd.

.. code-block:: none


   $ cd ~/go/src/github.com/lagopus/
   ~/go/src/github.com/lagopus$ git clone https://github.com/lagopus/lagopus-router.git

