Compiling Tox
=============
In order to compile Tox you'll need to do the following:

* compile all build requirements
* compile the core libraries
* compile a client

Compile all build requirements
------------------------------
Toxcore itself requires the following libraries:

* `libsodium <http://doc.libsodium.org/installation/README.html>`_ or `NaCl <http://nacl.cr.yp.to/install.html>`_

Toxav requires the following libraries:

* libopus
* libvpx

Both of these libraries can be installed using the instructions `here <https://github.com/irungentoo/toxcore/blob/master/INSTALL.md#libtoxav>`_ 

Compile the core libraries
--------------------------
You'll first need to get the core library, you can either download the latest PGP commit verified tarball from Jenkins `here <https://jenkins.libtoxcore.so/job/Sync%20Tox/lastSuccessfulBuild/artifact/toxcore.tar.gz>`_ or clone it from Github at ``https://github.com/irungentoo/toxcore``

Instructions on compiling are `here <https://github.com/irungentoo/toxcore/blob/master/INSTALL.md>`_

Compile a client
----------------
Next you'll need to compile a client, they are listed `here <https://wiki.tox.im/Clients>`_

`qTox <https://github.com/tux3/qTox#compiling-on-gnu-linux>`_ is a graphical, cross-platform client written for the Toxcore by Tux3 in C++ using QT.

`Toxic <https://github.com/Tox/toxic#installation>`_ is a CLI client for UNIX-like operating systems that uses Ncurses.

`uTox <https://github.com/notsecure/wintox/blob/master/docs/INSTALL.md>`_ is a graphical, cross-platform client written by notsecure.
