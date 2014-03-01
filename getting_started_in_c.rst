.. _getting_started_in_c/getting-started-with-the-c-api:

Getting Started With The C API
==============================
The central flow of a Tox client application goes basically like this
(highly simplified)

.. code-block:: c

   #define SLEEP_TIME 50000
   #define BOOTSTRAP_ADDRESS "23.226.230.47"
   #define BOOTSTRAP_PORT 33445
   #define BOOTSTRAP_KEY "A09162D68618E742FFBCA1C2C70385E6679604B2D80EA6E84AD0996A1AC8A074"

   #define MY_NAME "ImoutoBot"

   void hex_string_to_bin(const char *in, uint8_t *out) {
       ...
   }

   int main(int argc, const char *argv[]) {
       uint8_t *pub_key = malloc(TOX_CLIENT_ID_SIZE);
       hex_string_to_bin(BOOTSTRAP_KEY, pub_key);

       Tox *my_tox = tox_new(TOX_ENABLE_IPV6_DEFAULT);
       tox_set_name(my_tox, MY_NAME, strlen(MY_NAME));
       ...
       tox_bootstrap_from_address(my_tox, BOOTSTRAP_ADDRESS, TOX_ENABLE_IPV6_DEFAULT, BOOTSTRAP_PORT, pub_key);
       ...
       while (1) {
           tox_do(my_tox);
           ...
           usleep(SLEEP_TIME);
       }
       ...
       tox_kill(my_tox);
       return 0;
   }

.. _getting_started_in_c/getting-into-the-network:

Name And Identification
-----------------------
The first thing your client needs is a name. (*The Tox developers
recommend names referencing harmful substances, such as Toxic,
Poison, Venom, etc.*)

But we're not talking that kind of name. We're talking about your
Tox name, which gets shown to your friends and in group chats.

You can, of course, set your name using the API function
:ref:`api/tox_set_name`.

You can also set your user status. This shows your friends what
you are up to, and if you are busy, away, or available.

The API for doing this is broken into two functions,
:ref:`api/tox_set_status_message` and :ref:`api/tox_set_user_status`.

.. note::
   This was not shown in the sample code above.

Getting Into The Network
------------------------
.. note::
   You should read :ref:`core_concepts/up-by-the-bootstraps` to
   learn more about bootstrapping.

The C API provides two functions for dealing with bootstrapping.

* :ref:`api/tox_bootstrap_from_address`

You should call one of these as soon as you are finished preparing
your ``Tox *`` object.

.. _getting_started_in_c/lets-tox-do-it:

Let's ``tox_do()`` It
---------------------
The ``tox_do()`` function is the centre point of the Tox API.
It encapsulates everything that is needed to retain a connection
to the network in one function call.

Your main loop must call ``tox_do()`` at least 20 times per second.
In turn, ``tox_do()`` will invoke your registered callbacks.

.. note::
   This is rather inefficient, especially on low-power systems
   such as smartphones. For a more efficient way of running
   ``tox_do()``, see :ref:`getting_started_in_c/patience-is-a-virtue`.

.. _getting_started_in_c/call-me-back-maybe:

Call Me [Back] Maybe
--------------------
.. figure:: _static/tox_loop.png
   :alt: The circle of life.

When important events happen on the Tox connection, tox_do will
invoke callbacks that you specify with the following API
functions.

* :ref:`api/tox_callback_friend_request`
* :ref:`api/tox_callback_friend_message`
* :ref:`api/tox_callback_friend_action`
* :ref:`api/tox_callback_name_change`
* :ref:`api/tox_callback_status_message`
* :ref:`api/tox_callback_user_status`
* :ref:`api/tox_callback_typing_change`
* :ref:`api/tox_callback_read_receipt`
* :ref:`api/tox_callback_connection_status`
* :ref:`api/tox_callback_group_invite`
* :ref:`api/tox_callback_group_message`
* :ref:`api/tox_callback_group_action`
* :ref:`api/tox_callback_group_namelist_change`
* :ref:`api/tox_callback_file_send_request`
* :ref:`api/tox_callback_file_control`
* :ref:`api/tox_callback_file_data`

(*Click on a setter function above to see the required function
signature of your callback function.*)

Phew, that was a lot of functions! Don't worry, you only have
to set callbacks for the events you want to receive.

.. note::
   This was not shown in the sample code above.

.. _getting_started_in_c/patience-is-a-virtue:

Wait For Events To Come To You
------------------------------
I said earlier that calling ``tox_do()`` 20 times per second
was inefficient. You really don't have to call it that many
times per second, but what if something important happened?
This is what ``tox_wait...`` was designed to fix. It works
like POSIX ``select(2)``, so you can wait for something to
happen on the Tox connection rather than poll for it.

* :ref:`api/tox_wait_prepare`
* :ref:`api/tox_wait_execute`
* :ref:`api/tox_wait_cleanup`

Getting Ready
^^^^^^^^^^^^^
.. code-block:: c

    uint16_t rtmp = 0;
    tox_wait_prepare(my_tox, NULL, &rtmp);
    uint8_t tox_wait_buffer = malloc(rtmp);

``tox_wait_execute()`` requires a buffer to perform its work.
We use the ``tox_wait_prepare()`` function to get the required
buffer size, which will be returned in ``rtmp``. Then, we just
``malloc(rtmp)`` the right size.

Doing The Work
^^^^^^^^^^^^^^
.. code-block:: c

    int error = 0;
    error = tox_wait_execute(my_tox, tox_wait_buffer, rtmp, 999);
    tox_wait_cleanup(my_tox, tox_wait_buffer, rtmp);
    free(tox_wait_buffer);
    tox_do(my_tox);

``tox_wait_execute()`` will block until you need to call
``tox_do()``, or the timeout is reached (we used 999 milliseconds
in the example). Generally, you should call ``tox_do()`` anyway
if the timeout is reached.

After calling ``tox_wait_execute()``, we need to call
``tox_wait_cleanup()`` with the same arguments, except
timeout. The buffer we allocated earlier is no longer needed, so
it should be freed.

Putting It All Together
^^^^^^^^^^^^^^^^^^^^^^^
Here is our example program again, but using ``tox_wait...``
instead of a naïve loop.

.. code-block:: c

    #define BOOTSTRAP_ADDRESS "23.226.230.47"
    #define BOOTSTRAP_PORT 33445
    #define BOOTSTRAP_KEY "A09162D68618E742FFBCA1C2C70385E6679604B2D80EA6E84AD0996A1AC8A074"

    #define MY_NAME "ImoutoBot"

    void hex_string_to_bin(const char *in, uint8_t *out) {
        ...
    }

    int main(int argc, const char *argv[]) {
        uint8_t *pub_key = malloc(TOX_CLIENT_ID_SIZE);
        hex_string_to_bin(BOOTSTRAP_KEY, pub_key);

        Tox *my_tox = tox_new(TOX_ENABLE_IPV6_DEFAULT);
        tox_set_name(my_tox, MY_NAME, strlen(MY_NAME));
        ...
        tox_bootstrap_from_address(my_tox, BOOTSTRAP_ADDRESS, TOX_ENABLE_IPV6_DEFAULT, BOOTSTRAP_PORT, pub_key);
        ...
        while (1) {
            uint16_t rtmp = 0;
            tox_wait_prepare(my_tox, NULL, &rtmp);
            uint8_t tox_wait_buffer = malloc(rtmp);
            int error = 0;
            error = tox_wait_execute(my_tox, tox_wait_buffer, rtmp, 999);
            tox_wait_cleanup(my_tox, tox_wait_buffer, rtmp);
            free(tox_wait_buffer);

            tox_do(my_tox);
            ...
        }
        ...
        tox_kill(my_tox);
        return 0;
    }

.. note::
   We don't actually need to allocate a new buffer every time we
   call ``tox_wait_execute()``. Can you make the example code
   reuse a single buffer?
