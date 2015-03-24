.. _getting_started_in_c/getting-started-with-the-c-api:

Getting Started With The C API
==============================
To start with a brief example of how a C client structure should look is given, followed by each important section explained in more detail

.. code-block:: c

   #define SLEEP_TIME 50000
   #define BOOTSTRAP_ADDRESS "23.226.230.47"
   #define BOOTSTRAP_PORT 33445
   #define BOOTSTRAP_KEY "A09162D68618E742FFBCA1C2C70385E6679604B2D80EA6E84AD0996A1AC8A074"

   #define MY_NAME "ImoutoBot"
   #define MY_STATUS_MESSAGE "This is a message!"

   void hex_string_to_bin(const char *in, uint8_t *out) {
       ...
   }

   void MyFriendRequestCallback(Tox *tox, const uint8_t *public_key, const uint8_t *message, size_t length, void *user_data) {
      ...
   }

   void MyFriendMessageCallback(Tox *tox, uint32_t friend_number, TOX_MESSAGE_TYPE type, const uint8_t *message, size_t length, void *user_data) {
      ...
   }

   ...

   int main(int argc, const char *argv[]) {
       uint8_t *bootstrap_pub_key = malloc(TOX_PUBLIC_KEY_SIZE);
       hex_string_to_bin(BOOTSTRAP_KEY, bootstrap_pub_key);

       /* Create a default Tox */
       Tox *my_tox = tox_new(NULL, NULL, 0, NULL);

       /* Register the callbacks */
       tox_callback_friend_request(my_tox, MyFriendRequestCallback, NULL);
       tox_callback_friend_message(my_tox, MyFriendMessageCallback, NULL);

       ...

       /* Define or load some user details for the sake of it */
       tox_self_set_name(my_tox, MY_NAME, strlen(MY_NAME), NULL); // Sets the username
       tox_self_set_status_message(my_tox, MY_STATUS_MESSAGE, strlen(MY_STATUS_MESSAGE), NULL); // Sets the status message

       /* Set the user status to TOX_USER_STATUS_NONE. Other possible values:
          TOX_USER_STATUS_AWAY and TOX_USER_STATUS_BUSY */
       tox_self_set_status(my_tox, TOX_USER_STATUS_NONE);

       ...

       /* Bootstrap from the node defined above */
       tox_bootstrap(my_tox, BOOTSTRAP_ADDRESS, BOOTSTRAP_PORT, bootstrap_pub_key, NULL);

       ...

       while (1) {
           tox_iterate(my_tox); // will call the callback functions defined and registered 

           ...

           usleep(tox_iteration_interval(my_tox)); // sleep until the next iteration should happen
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

When important events happen on the Tox connection, tox_iterate will
invoke callbacks that you specify with the following API
functions.

* `tox_callback_self_connection_status <https://libtoxcore.so/api/tox_8h.html#ab38a7512be865980d45819a3ab7e5e5a>`_
* `tox_callback_friend_name <https://libtoxcore.so/api/tox_8h.html#a09d71ba40072133d03da17422dd06bf0>`_
* `tox_callback_friend_status_message <https://libtoxcore.so/api/tox_8h.html#aac5e8d3bef2a458e0e287a2de7cf9604>`_
* `tox_callback_friend_status <https://libtoxcore.so/api/tox_8h.html#ad1878862d94e2c2ba1d51e761b02efae>`_
* `tox_callback_friend_connection_status <https://libtoxcore.so/api/tox_8h.html#aa7d891aaf1f15ee03f55b66227744157>`_
* `tox_callback_friend_typing <https://libtoxcore.so/api/tox_8h.html#acca76b201e0c38c871f3913f1ae99a07>`_
* `tox_callback_friend_read_receipt <https://libtoxcore.so/api/tox_8h.html#aafcb609f32feff42d9bc9ded9f771931>`_
* `tox_callback_friend_request <https://libtoxcore.so/api/tox_8h.html#a2cf9a901fd5db3b6635a3ece389cc349>`_
* `tox_callback_friend_message <https://libtoxcore.so/api/tox_8h.html#a31635691f5ee3ee6ee061215d18087ae>`_
* `tox_callback_file_recv_control <https://libtoxcore.so/api/tox_8h.html#abb0eca9253a594357dfa0da0c9c64a0d>`_
* `tox_callback_file_chunk_request <https://libtoxcore.so/api/tox_8h.html#ade12d3a935a20a1e2a87afa1799343a9>`_
* `tox_callback_file_recv <https://libtoxcore.so/api/tox_8h.html#a2838aa05de2c47f58a45645248303b60>`_
* `tox_callback_file_recv_chunk <https://libtoxcore.so/api/tox_8h.html#ad828f18b7b4901f258fe7132b1bec4f6>`_
* `tox_callback_friend_lossy_packet <https://libtoxcore.so/api/tox_8h.html#a17c90611298a86c1d132fdfb5aa52e00>`_
* `tox_callback_friend_lossless_packet <https://libtoxcore.so/api/tox_8h.html#a4394a6985e8d6d652894b89211a8062e>`_

(*Click on a setter function above to see the required function
signature of your callback function.*)

Phew, that was a lot of functions! Don't worry, you only have to set callbacks for the events you want to receive.

.. _getting_started_in_c/user-details:

User Details
-----------------------
Clients should set the user details before connecting to a bootstrap. 

The most essential detail needed is a username which is shown to the user's friends after having connected to them

``tox_self_set_name(my_tox, MY_NAME, strlen(MY_NAME), TOX_ERR_SET_INFO *error);``

As well as a username, you may also set a user status which defines their state of availability; online, offline, away and busy.
These are part of an enumeration, TOX_USER_STATUS and not strings

``tox_set_status(my_tox, uint8_t userstatus);``

Lastly, a user can also have a status message which is a string

``tox_self_set_status_message(my_tox, uint8_t *status, uint16_t length, TOX_ERR_SET_INFO *error);``

.. _getting_started_inc_/getting-into-the-network

Getting Into The Network
------------------------
.. note::
   You should read :ref:`core_concepts/up-by-the-bootstraps` to
   learn more about bootstrapping.

Once you've registered your callbacks and set your user details, you now want to connect to a bootstrap to get into the network

``tox_bootstrap_from_address(my_tox, BOOTSTRAP_ADDRESS, BOOTSTRAP_PORT, bootstrap_pub_key);``

This function accepts both an IP and a hostname for the bootstrap address.

.. _getting_started_in_c/lets-tox-do-it:

Let's ``tox_iterate()`` over it
---------------------
The ``tox_iterate()`` function is the centre point of the Tox API.
It encapsulates everything that is needed to retain a connection to the network in one function call.
Your main loop should call ``tox_iterate()`` in the interval that is given by ``tox_iteration_interval()``.
In turn, ``tox_iterate()`` will invoke your registered callbacks.
