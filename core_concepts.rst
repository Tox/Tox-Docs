Core concepts
=============

.. _core_concepts/public-keys:

The Tox ID
----------
.. figure:: _static/public_key.png
   :alt: Something about a musty, deranged man who cackles out a
         string of random letters and numbers...

This is a typical Tox ID that you currently give out to friends.
It is a public key, nospam value, and checksum concatenated
in hexadecimal format. The result is the 76-character string
shown above.

.. figure:: _static/public_key_bd.png
   :alt: > implying checksum

(*The three parts of the a Tox ID.*)

Public Key
^^^^^^^^^^
The public key is generated from the ``crypto_box_keypair`` function
by NaCl.

This is explained better `here <http://nacl.cr.yp.to/box.html>`__.
In the current implementation of NaCl, it is 32 bytes (64 hexadecimal
characters).

``nospam`` Value
^^^^^^^^^^^^^^^^
The ``nospam`` value is a randomly generated number appended to the
key. A friend request sent without knowing the correct ``nospam``
value will be ignored.

The ``nospam`` value can be changed at any time without affecting
the public key, stopping all requests to your current ID. This makes
it effective for fighting spam (its original purpose!).

Checksum
^^^^^^^^
The checksum is a simple XOR checksum of the public key and
``nospam`` value. It is used to to quickly verify the integrity
of the Tox ID.

.. _core_concepts/up-by-the-bootstraps:

Connecting to the network
--------------------
Because Tox has no central servers, you will need to know someone
who is already "in" the network before you can successfully
connect your client.

The Tox core handles this in one of two ways:

* Automatically bootstrapping to a Tox client that it finds
  on the LAN, and
* Bootstrapping to a known peer that your code provides.

.. note::
   If you would like that data in a more machine-friendly
   format, an unofficial JSON list is available
   `here <https://dist-build.tox.im/Nodefile.json>`__,
   updated hourly from the wiki.
   Additionally, a script to scrape entries off the wiki
   is available
   `here <https://github.com/Jman012/Tox-DHTservers-Updater>`__.
