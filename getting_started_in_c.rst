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
* :ref:`api/tox_callback_group_title`
* :ref:`api/tox_callback_group_namelist_change`
* :ref:`api/tox_callback_avatar_info`
* :ref:`api/tox_callback_avatar_data`
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

The most essential detail needed is a username which is shown to the user's friends after having connected to them

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

``tox_bootstrap_from_address(my_tox, BOOTSTRAP_ADDRESS, BOOTSTRAP_PORT, bootstrap_pub_key);``

This function accepts both an IP and a hostname for the bootstrap address.

.. _getting_started_in_c/lets-tox-do-it:

Let's ``tox_do()`` It
---------------------
The ``tox_do()`` function is the centre point of the Tox API.
It encapsulates everything that is needed to retain a connection
to the network in one function call. Your main loop must call ``tox_do()`` at least 20 times per second.
In turn, ``tox_do()`` will invoke your registered callbacks.
