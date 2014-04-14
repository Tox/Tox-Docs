.. _getting_started_in_c/getting-started-with-the-c-api:

Getting Started With The C API
==============================
The central flow of the client should be as follows, in Pseudo-Code and in C. Read further down the page for more 
details on each essential part of these examples.

.. code-block:: c

   #define SLEEP_TIME 50000
   #define BOOTSTRAP_ADDRESS "23.226.230.47"
   #define BOOTSTRAP_PORT 33445
   #define BOOTSTRAP_KEY "A09162D68618E742FFBCA1C2C70385E6679604B2D80EA6E84AD0996A1AC8A074"

   #define MY_NAME "ImoutoBot"

   void hex_string_to_bin(const char *in, uint8_t *out) {
       ...
   }

   void MyFriendRequestCallback(Tox *tox, uint8_t * public_key, uint8_t * data, uint16_t length, void *userdata) {
      ...
   }

   void MyFriendMessageCallback(Tox *tox, int friendnumber, uint8_t * message, uint32_t length, void *userdata) {
      ...
   }

   ...

   int main(int argc, const char *argv[]) {
       uint8_t *pub_key = malloc(TOX_CLIENT_ID_SIZE);
       hex_string_to_bin(BOOTSTRAP_KEY, pub_key);
      
       Tox *my_tox = tox_new(TOX_ENABLE_IPV6_DEFAULT);
       
       /* Register the callbacks */
       tox_callback_friend_request(my_tox, MyFriendRequestCallback, NULL);
       tox_callback_friend_message(my_tox, MyFriendMessageCallback, NULL);
       ...
       
       /* Define or load some user details for the sake of it */
       tox_set_name(my_tox, MY_NAME, strlen(MY_NAME)); // Sets the username 
       tox_set_status_message(my_tox, uint8_t *status, uint16_t length); // user status is pre-defined ints for "online", "offline" etc.
       tox_set_user_status(my_tox, uint8_t userstatus); // status message is a string the user can set 
       
       ...
       
       tox_bootstrap_from_address(my_tox, BOOTSTRAP_ADDRESS, TOX_ENABLE_IPV6_DEFAULT, BOOTSTRAP_PORT, pub_key); // connect to a bootstrap to get into the network
       
       ...
       
       while (1) {
           tox_do(my_tox); // will call the callback functions defined and registered 
           
           ...
           
           usleep(SLEEP_TIME); // sleep for cpu usage, tox_wait() can be used instead for blocking
       }
       
       ...
       
       tox_kill(my_tox);
       return 0;
   }

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

.. _getting_started_in_c/user-details:

User Details
-----------------------
Clients should set the user details before connecting to a bootstrap. 

The most essential detail needed is a username which is shown to the user's friends after having being connected to them

``tox_set_name(my_tox, MY_NAME, strlen(MY_NAME));``

As well as a username, you may also set a user status which defines their state of availability; online, offline, away and busy.
These are part of an enumeration, TOX_USERSTATUS and not strings

``tox_set_user_status(my_tox, uint8_t userstatus);``

Lastly, a user can also have a status message which is a string

``tox_set_status_message(my_tox, uint8_t *status, uint16_t length);``

.. _getting_started_inc_/getting-into-the-network

Getting Into The Network
------------------------
.. note::
   You should read :ref:`core_concepts/up-by-the-bootstraps` to
   learn more about bootstrapping.

Once you've registered your callbacks and set your user details, you now want to connect to a bootstrap to get into the network

``tox_bootstrap_from_address(my_tox, BOOTSTRAP_ADDRESS, TOX_ENABLE_IPV6_DEFAULT, BOOTSTRAP_PORT, pub_key);``

This function accepts both an IP and a hostname for the bootstrap address. You can also enable IPV6 by passing a non-zero
value for ``TOX_ENABLE_IPV6_DEFAULT``

.. _getting_started_in_c/lets-tox-do-it:

Let's ``tox_do()`` It
---------------------
The ``tox_do()`` function is the centre point of the Tox API.
It encapsulates everything that is needed to retain a connection
to the network in one function call. Your main loop must call ``tox_do()`` at least 20 times per second.
In turn, ``tox_do()`` will invoke your registered callbacks.

.. _getting_started_in_c/patience-is-a-virtue:

Wait For Events To Come To You
------------------------------
It can be very inefficient calling tox_do() 20 times a second, but what if something important happened?
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
Use the ``tox_wait_prepare()`` function to get the required
buffer size, which will be returned in ``rtmp``. Then, just
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
Here is the example C program again, but using ``tox_wait...``
instead of a naïve sleep loop.

.. code-block:: c

    #define BOOTSTRAP_ADDRESS "23.226.230.47"
    #define BOOTSTRAP_PORT 33445
    #define BOOTSTRAP_KEY "A09162D68618E742FFBCA1C2C70385E6679604B2D80EA6E84AD0996A1AC8A074"

    #define MY_NAME "ImoutoBot"

    void hex_string_to_bin(const char *in, uint8_t *out) {
        ...
    }

    void hex_string_to_bin(const char *in, uint8_t *out) {
       ...
    }

    void MyFriendRequestCallback(Tox *tox, uint8_t * public_key, uint8_t * data, uint16_t length, void *userdata) {
      ...
    }

    void MyFriendMessageCallback(Tox *tox, int friendnumber, uint8_t * message, uint32_t length, void *userdata) {
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
   You don't actually need to allocate a new buffer every time we
   call ``tox_wait_execute()``.
