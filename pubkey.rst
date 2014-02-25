Understanding a Tox ID
========================

A Tox ID you distribute is a public key, nospam value, and checksum; in hexadecimal format. This results in 64 characters + 8 characters + 4 characters, for 76.

So something like 

``56A1ADE4B65B86BCD51CC73E2CD4E542179F47959FE3E0E21B4B0ACDADE5185520B3E6FC5D64``

Can be broken down to

``Public Key:[56A1ADE4B65B86BCD51CC73E2CD4E542179F47959FE3E0E21B4B0ACDADE51855] nospam:[20B3E6FC] checksum:[5D64]``

Public key
==========
The public key is generated from the ``crypto_box_keypair`` function by NaCL.
This is explained better `here <http://nacl.cr.yp.to/box.html>`_.

nospam
==========
A nospam is a randomly generated string appended to the key. A client may change his nospam without changing his key at any time, blocking requests to his ID.

checksum
==========
The checksum is a simple xor checksum of both the public key and nospam, and is only used to ensure the presented key is a valid Tox ID.
