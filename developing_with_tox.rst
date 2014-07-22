Developing With Tox
===================

.. _developing_with_tox/c-is-for-core:

"C" Is For Core
---------------
The core of Tox is written in C. Don't know a lick of C? Heard horror
stories of C programs eating children?
Don't worry, that won't preclude you from using the Tox API.

.. _developing_with_tox/shrek:

Like An Ogre, Tox Has Many [Wrappers]
-------------------------------------
API bindings/wrappers are available for users of high-level languages
so they too can use the Tox API.
A list of available language wrappers for the Tox API:

* `jToxcore <https://github.com/Tox/jToxcore>`_ *(Java)*
* `DeepEnd <https://github.com/stal888/DeepEnd>`_ *(Objective-C)*
* `PyTox <https://github.com/aitjcize/PyTox>`_ *(Python)*
* `libtoxcore-racket <https://github.com/lehitoskin/libtoxcore-racket>`_ *(Racket)*
* `haskell-tox-core <https://github.com/ollieh/haskell-tox-core>`_ *(Haskell)*
* `SharpTox <https://github.com/Impyy/SharpTox>`_ *(C#)*
* `golibtox <https://github.com/organ/golibtox/>`_ *(Go)*
* `luatox <https://github.com/peersuasive/luatox/>`_ *(Lua)*
* `guile-toxcore <https://gitorious.org/guile-toxcore/guile-toxcore/>`_ *(Guile)*


.. warning::
   These may not be 1-1 wrappers of API functions. You should read
   the documentation of your chosen wrapper for usage details.

You're Seriously Going To Write C, Then
---------------------------------------
Good, C is a good language. The next topic will cover the basics
of your application's main Tox loop.
