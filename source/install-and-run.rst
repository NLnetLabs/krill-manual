.. _doc_krill_install_and_run:

Install and Run
===============

Before you can start to use Krill you will need to install, configure and run
the Krill application somewhere. Please follow the steps below and you will be
ready to :ref:`get started<doc_krill_get_started>`.

Quick Start
-----------

Getting started with Krill is really easy by either installing a binary package
for Debian and Ubuntu or for Red Hat Enterprise Linux and CentOS. You can also
run with :ref:`Docker<doc_krill_running_docker>` or build from Cargo, Rust's
build system and package manager.

In case you intend to serve your RPKI certificate and ROAs to the world yourself
or you want to offer this as a service to others, you will also need to have a
public rsyncd and HTTPS web server available.

.. Note:: For the oldest platforms, Ubuntu 16.04 LTS and Debian 9, the packaged
          Krill binary is statically linked with OpenSSL 1.1.0 as this is the
          minimum version required by Krill and is higher than available in the
          official package repositories for those platforms.

.. tabs::

   .. group-tab:: Debian

       If you have a machine with an amd64/x86_64 architecture running Debian 9,
       10 or 11, you can install Krill from our `software package
       repository <https://packages.nlnetlabs.nl>`_. 
       
       First update the ``apt`` package index: 

       .. code-block:: bash

          sudo apt update

       Then install packages to allow ``apt`` to use a repository over HTTPS:

       .. code-block:: bash

          sudo apt install \
            ca-certificates \
            curl \
            gnupg \
            lsb-release

       Add the GPG key from NLnet Labs:

       .. code-block:: bash

          curl -fsSL https://packages.nlnetlabs.nl/aptkey.asc | sudo gpg --dearmor -o /usr/share/keyrings/nlnetlabs-archive-keyring.gpg

       Now, use the following command to set up the *main* repository:

       .. code-block:: bash

          echo \
          "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/nlnetlabs-archive-keyring.gpg] https://packages.nlnetlabs.nl/linux/debian \
          $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/nlnetlabs.list > /dev/null

       After updating the ``apt`` package index you can install Krill:

       .. code-block:: bash

          sudo apt update
          sudo apt install krill

       Review the generated configuration file at ``/etc/krill.conf``. **Pay 
       particular attention** to the ``service_uri`` and ``admin_token``
       settings. Tip: The configuration file was generated for you using the
       ``krillc config simple`` command.
       
       Once happy with the settings use ``sudo systemctl enable --now krill`` to
       instruct systemd to enable the Krill service at boot and to start it
       immediately. The krill daemon runs as user ``krill`` and stores its data
       in ``/var/lib/krill``. 
       
       You can check the status of Krill with:
       
       .. code-block:: bash 
       
          sudo systemctl status krill
       
       You can view the logs with: 
       
       .. code-block:: bash
       
          sudo journalctl --unit=krill

   .. group-tab:: Ubuntu

       If you have a machine with an amd64/x86_64 architecture running Ubuntu
       16.x, 18.x, 20.x or 22.x, you can install Krill from our `software
       package repository <https://packages.nlnetlabs.nl>`_. 
       
       First update the ``apt`` package index: 

       .. code-block:: bash

          sudo apt update

       Then install packages to allow ``apt`` to use a repository over HTTPS:

       .. code-block:: bash

          sudo apt install \
            ca-certificates \
            curl \
            gnupg \
            lsb-release

       Add the GPG key from NLnet Labs:

       .. code-block:: bash

          curl -fsSL https://packages.nlnetlabs.nl/aptkey.asc | sudo gpg --dearmor -o /usr/share/keyrings/nlnetlabs-archive-keyring.gpg

       Now, use the following command to set up the *main* repository:

       .. code-block:: bash

          echo \
          "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/nlnetlabs-archive-keyring.gpg] https://packages.nlnetlabs.nl/linux/ubuntu \
          $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/nlnetlabs.list > /dev/null

       After updating the ``apt`` package index you can install Krill:

       .. code-block:: bash

          sudo apt update
          sudo apt install krill

       Review the generated configuration file at ``/etc/krill.conf``. **Pay 
       particular attention** to the ``service_uri`` and ``admin_token``
       settings. Tip: The configuration file was generated for you using the
       ``krillc config simple`` command.
       
       Once happy with the settings use ``sudo systemctl enable --now krill`` to
       instruct systemd to enable the Krill service at boot and to start it
       immediately. The krill daemon runs as user ``krill`` and stores its data
       in ``/var/lib/krill``. 
       
       You can check the status of Krill with:
       
       .. code-block:: bash 
       
          sudo systemctl status krill
       
       You can view the logs with: 
       
       .. code-block:: bash
       
          sudo journalctl --unit=krill

   .. group-tab:: RHEL/CentOS

       If you have a machine with an amd64/x86_64 architecture running a
       :abbr:`RHEL (Red Hat Enterprise Linux)`/CentOS 7 or 8 distribution, or a
       compatible OS such as Rocky Linux, you can install Krill from our
       `software package repository <https://packages.nlnetlabs.nl>`_. 
       
       To use this repository, create a file named 
       :file:`/etc/yum.repos.d/nlnetlabs.repo`, enter this configuration and 
       save it:
       
       .. code-block:: text
       
          [nlnetlabs]
          name=NLnet Labs
          baseurl=https://packages.nlnetlabs.nl/linux/centos/$releasever/main/$basearch
          enabled=1
        
       Then run the following command to add the public key:
       
       .. code-block:: bash
       
          sudo rpm --import https://packages.nlnetlabs.nl/aptkey.asc
       
       You can then install Krill by running:
        
       .. code-block:: bash
          
          sudo yum install -y krill
           
       Review the generated configuration file at ``/etc/krill.conf``. **Pay 
       particular attention** to the ``service_uri`` and ``admin_token``
       settings. Tip: The configuration file was generated for you using the
       ``krillc config simple`` command.
          
       Once happy with the settings use ``sudo systemctl enable --now krill`` to
       instruct systemd to enable the Krill service at boot and to start it
       immediately. The krill daemon runs as user ``krill`` and stores its data
       in ``/var/lib/krill``. 
      
       You can check the status of Krill with:
      
       .. code-block:: bash 
      
          sudo systemctl status krill
      
       You can view the logs with: 
       
       .. code-block:: bash
       
          sudo journalctl --unit=krill
                      
   .. group-tab:: Cargo

       Assuming you have a newly installed Debian or Ubuntu machine, you will
       need to install the C toolchain, OpenSSL and Rust. You can then install
       Krill using:

       .. code-block:: bash

          sudo apt install curl build-essential libssl-dev openssl pkg-config
          curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
          source ~/.cargo/env
          cargo install --locked krill

Updating
--------

.. tabs::

   .. group-tab:: Debian

       To update an existing Krill installation, first update the repository
       using:

       .. code-block:: text

          sudo apt update

       You can use this command to get an overview of the available versions:

       .. code-block:: text

          sudo apt policy krill

       You can upgrade an existing Krill installation to the latest version
       using:

       .. code-block:: text

          sudo apt --only-upgrade install krill

   .. group-tab:: Ubuntu

       To update an existing Krill installation, first update the repository
       using:

       .. code-block:: text

          sudo apt update

       You can use this command to get an overview of the available versions:

       .. code-block:: text

          sudo apt policy krill

       You can upgrade an existing Krill installation to the latest version
       using:

       .. code-block:: text

          sudo apt --only-upgrade install krill

   .. group-tab:: RHEL/CentOS

       To update an existing Krill installation, you can use this command 
       to get an overview of the available versions:
        
       .. code-block:: bash
        
          sudo yum --showduplicates list krill
          
       You can update to the latest version using:
         
       .. code-block:: bash
         
          sudo yum update -y krill
             
   .. group-tab:: Cargo

       If you want to install the latest version of Krill using Cargo, it's
       recommended to also update Rust to the latest version first. Use the 
       ``--force`` option to  overwrite an existing version with the latest 
       release:
               
       .. code-block:: text

          rustup update
          cargo install --locked --force krill
          
Installing Specific Versions
----------------------------

Before every new release of Krill, one or more release candidates are 
provided for testing through every installation method. You can also install
a specific version, if needed.

.. tabs::

   .. group-tab:: Debian

       If you would like to try out release candidates of Routinator you can add
       the *proposed* repository to the existing *main* repository described
       earlier. 
       
       Assuming you already have followed the steps to install regular releases,
       run this command to add the additional repository:

       .. code-block:: bash

          echo \
          "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/nlnetlabs-archive-keyring.gpg] https://packages.nlnetlabs.nl/linux/debian \
          $(lsb_release -cs)-proposed main" | sudo tee /etc/apt/sources.list.d/nlnetlabs-proposed.list > /dev/null

       Make sure to update the ``apt`` package index:

       .. code-block:: bash

          sudo apt update
       
       You can now use this command to get an overview of the available 
       versions:

       .. code-block:: bash

          sudo apt policy krill

       You can install a specific version using ``<package name>=<version>``,
       e.g.:

       .. code-block:: bash

          sudo apt install krill=0.9.0~rc2-1buster

   .. group-tab:: Ubuntu

       If you would like to try out release candidates of Krill you can add
       the *proposed* repository to the existing *main* repository described
       earlier. 
       
       Assuming you already have followed the steps to install regular releases,
       run this command to add the additional repository:

       .. code-block:: bash

          echo \
          "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/nlnetlabs-archive-keyring.gpg] https://packages.nlnetlabs.nl/linux/ubuntu \
          $(lsb_release -cs)-proposed main" | sudo tee /etc/apt/sources.list.d/nlnetlabs-proposed.list > /dev/null

       Make sure to update the ``apt`` package index:

       .. code-block:: bash

          sudo apt update
       
       You can now use this command to get an overview of the available 
       versions:

       .. code-block:: bash

          sudo apt policy krill

       You can install a specific version using ``<package name>=<version>``,
       e.g.:

       .. code-block:: bash

          sudo apt install krill=0.9.0~rc2-1bionic
          
   .. group-tab:: RHEL/CentOS

       To install release candidates of Routinator, create an additional repo 
       file named :file:`/etc/yum.repos.d/nlnetlabs-testing.repo`, enter this
       configuration and save it:
       
       .. code-block:: text
       
          [nlnetlabs-testing]
          name=NLnet Labs Testing
          baseurl=https://packages.nlnetlabs.nl/linux/centos/$releasever/proposed/$basearch
          enabled=1
        
       You can use this command to get an overview of the available versions:
        
       .. code-block:: bash
        
          sudo yum --showduplicates list krill
          
       You can install a specific version using 
       ``<package name>-<version info>``, e.g.:
         
       .. code-block:: bash
         
          sudo yum install -y krill-0.9.0~rc2
                            
   .. group-tab:: Cargo

       All release versions of Krill, as well as release candidates, are
       available on `crates.io <https://crates.io/crates/krill/versions>`_,
       the Rust package registry. If you want to install a specific version of
       Krill using Cargo, explicitly use the ``--version`` option. If
       needed, use the ``--force`` option to overwrite an existing version:
               
       .. code-block:: text

          cargo install --locked --force krill --version 0.9.0-rc2

       All new features of Krill are built on a branch and merged via a
       `pull request <https://github.com/NLnetLabs/krill/pulls>`_, allowing
       you to easily try them out using Cargo. If you want to try the a specific
       branch from the repository you can use the ``--git`` and ``--branch``
       options:

       .. code-block:: text

          cargo install --git https://github.com/NLnetLabs/krill.git --branch main
          
       For more installation options refer to the `Cargo book
       <https://doc.rust-lang.org/cargo/commands/cargo-install.html#install-options>`_.

Installing with Cargo
---------------------

There are three things you need for Krill: Rust, a C toolchain and OpenSSL.
You can install Krill on any Operating System where you can fulfil these
requirements, but we will assume that you will run this on a UNIX-like OS.

Rust
""""

The Rust compiler runs on, and compiles to, a great number of platforms,
though not all of them are equally supported. The official `Rust
Platform Support <https://forge.rust-lang.org/platform-support.html>`_
page provides an overview of the various support levels.

While some system distributions include Rust as system packages,
Krill relies on a relatively new version of Rust, currently 1.45 or
newer. We therefore suggest to use the canonical Rust installation via a
tool called :command:`rustup`.

To install :command:`rustup` and Rust, simply do:

.. code-block:: bash

   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

Alternatively, visit the `official Rust website
<https://www.rust-lang.org/tools/install>`_ for other installation methods.

You can update your Rust installation later by running:

.. code-block:: bash

   rustup update

For some platforms, :command:`rustup` cannot provide binary releases to install
directly. The `Rust Platform Support
<https://forge.rust-lang.org/platform-support.html>`_ page lists
several platforms where official binary releases are not available,
but Rust is still guaranteed to build. For these platforms, automated
tests are not run so it’s not guaranteed to produce a working build, but
they often work to quite a good degree.

One such example that is especially relevant for the routing community
is OpenBSD. On this platform, `patches
<https://github.com/openbsd/ports/tree/master/lang/rust/patches>`_ are
required to get Rust running correctly, but these are well maintained
and offer the latest version of Rust quite quickly.

Rust can be installed on OpenBSD by running:

.. code-block:: bash

   pkg_add rust

Another example where the standard installation method does not work is
CentOS 6, where you will end up with a long list of error messages about
missing assembler instructions. This is because the assembler shipped with
CentOS 6 is too old.

You can get the necessary version by installing the `Developer Toolset 6
<https://www.softwarecollections.org/en/scls/rhscl/devtoolset-6/>`_ from the
`Software Collections
<https://wiki.centos.org/AdditionalResources/Repositories/SCL>`_ repository. On
a virgin system, you can install Rust using these steps:

.. code-block:: bash

   sudo yum install centos-release-scl
   sudo yum install devtoolset-6
   scl enable devtoolset-6 bash
   curl https://sh.rustup.rs -sSf | sh
   source $HOME/.cargo/env

C Toolchain
"""""""""""

Some of the libraries Krill depends on require a C toolchain to be
present. Your system probably has some easy way to install the minimum
set of packages to build from C sources. For example,
:command:`apt install build-essential` will install everything you need on
Debian/Ubuntu.

If you are unsure, try to run :command:`cc` on a command line and if there’s a
complaint about missing input files, you are probably good to go.

OpenSSL
"""""""

Your system will likely have a package manager that will allow you to install
OpenSSL in a few easy steps. For Krill, you will need :command:`libssl-dev`,
sometimes called :command:`openssl-dev`. On Debian-like Linux distributions,
this should be as simple as running:

.. code-block:: bash

    apt install libssl-dev openssl pkg-config


Building
""""""""

The easiest way to get Krill v0.9.0 RC1 is to leave it to cargo by saying:

.. code-block:: bash

   cargo install krill --git https://github.com/NLnetLabs/krill \
                       --tag v0.9.0-rc1 \
                       --locked

If you want to update an installed version, you run the same command but
add the ``-f`` flag, a.k.a. force, to approve overwriting the installed
version.

The command will build Krill and install it in the same directory
that cargo itself lives in, likely :file:`$HOME/.cargo/bin`. This means
Krill will be in your path, too.


Generate Configuration File
---------------------------

After the installation has completed, there are just two things you need to
configure before you can start using Krill. First, you will need a data
directory, which will store everything Krill needs to run. Secondly, you will
need to create a basic configuration file, specifying a secret token and the
location of your data directory.

The first step is to choose where your data directory is going to live and to
create it. In this example we are simply creating it in our home directory.

.. code-block:: bash

  mkdir ~/data

Krill can generate a basic configuration file for you. We are going to specify
the two required directives, a secret token and the path to the data directory,
and then store it in this directory.

.. parsed-literal::

  :ref:`krillc config simple<cmd_krillc_config_simple>` --token correct-horse-battery-staple --data ~/data/ > ~/data/krill.conf

.. Note:: If you wish to run a self-hosted RPKI repository with Krill you will
          need to use a different ``krillc config`` command. See :ref:`doc_krill_publication_server`
          for more details.

You can find a full example configuration file with defaults in `the
GitHub repository
<https://github.com/NLnetLabs/krill/blob/master/defaults/krill.conf>`_.

Start and Stop the Daemon
-------------------------

There is currently no standard script to start and stop Krill. You could use the
following example script to start Krill. Make sure to update the
``DATA_DIR`` variable to your real data directory, and make sure you saved
your :file:`krill.conf` file there.

.. code-block:: bash

  #!/bin/bash
  KRILL="krill"
  DATA_DIR="/path/to/data"
  KRILL_PID="$DATA_DIR/krill.pid"
  CONF="$DATA_DIR/krill.conf"
  SCRIPT_OUT="$DATA_DIR/krill.log"

  nohup $KRILL -c $CONF >$SCRIPT_OUT 2>&1 &
  echo $! > $KRILL_PID

You can use the following sample script to stop Krill:

.. code-block:: bash

  #!/bin/bash
  DATA_DIR="/path/to/data"
  KRILL_PID="$DATA_DIR/krill.pid"

  kill `cat $KRILL_PID`
