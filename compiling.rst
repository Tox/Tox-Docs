Compiling Tox
=============
In order to compile Tox you'll need to do the following:

* compile all build requirements
* compile the core libraries
* compile a client

Compile all build requirements
------------------------------
Toxcore itself requires the following libraries:

* libsodium or NaCl

Toxav requires the following libraries:

* opus
* vpx

Compile the core libraries
--------------------------
You'll first need to get the core library, you can either download the latest PGP commit verified tarball from Jenkins `here <https://jenkins.libtoxcore.so/job/Sync%20Tox/lastSuccessfulBuild/artifact/toxcore.tar.gz>`_ or clone it from github at ``https://github.com/irungentoo/toxcore``

Instructions on compiling are `here <https://github.com/irungentoo/toxcore/blob/master/INSTALL.md>`_

Compile a client
----------------
Next you'll need to compile a client, they are listed `here <https://wiki.tox.im/Clients>`_

TODO: Common clients + instructions