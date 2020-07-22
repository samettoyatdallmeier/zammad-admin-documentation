API Reference
=============

For most users,
:ref:`the sample scripts from the Setup Guide <checkmk-api-alerts>`
will do the job just fine.
But if you want more fine-grained control—for instance,
to create high- and low-priority tickets
for different types of system events—then
you’ll have to customize the data those scripts send to Zammad.

Example
-------

The following script will automatically set all tickets it creates to high priority
and assign them to charlie@chrispresso.com.

.. code-block:: bash

   #!/bin/bash

   curl -X POST \
     -F "event_id=$NOTIFY_HOSTPROBLEMID" \
     -F "host=$NOTIFY_HOSTNAME" \
     -F "state=$NOTIFY_HOSTSTATE" \
     -F "text=$NOTIFY_HOSTOUTPUT" \
     -F "priority=3 high" \
     -F "owner=charlie@chrispresso.com" \
     https://zammad.example.com/api/v1/...

How does it work?
-----------------

There are two kinds of data you can pass to the API:

Checkmk parameters
   are required, and make up the **contents** of the resulting tickets/articles.
   They also determine whether an event creates a new ticket
   or updates/closes an existing one.
Ticket attributes
   are optional, and can be used to **adjust settings** on newly created tickets
   (*e.g.,* set the owner, group, priority, or state).

These parameters/attributes are passed in the form of key-value pairs.
To include them in your API request,
simply add an extra “form” option for each one (``-F "key=value"``)
to your script’s ``curl`` command line, as in the example above.

.. hint:: 💡 **It's just an API endpoint!**

   When using Checkmk integration, messages need to be formatted in a certain way,
   but that *doesn’t mean the messages actually have to come from Checkmk*.

   If you use another monitoring tool that’s not officially supported by Zammad,
   there’s probably a way to make it work with your Checkmk callback URL.

Checkmk Parameters
------------------

When a notification is received, Zammad creates a new article
containing the details of the event that triggered it:

.. figure:: /images/system/integrations/checkmk/checkmk-parameters.png
   :alt: Checkmk article body
   :align: center

These details come from the fields listed below,
which correspond to parameters provided by Checkmk (``$NOTIFY_*``).
Technically, they can be replaced with whatever you want,
but it’s not recommended.

**Required fields are marked with an asterisk (\*).**

event_id\*
   A unique ID for the system event. (``$NOTIFY_SERVICEPROBLEMID`` / ``$NOTIFY_HOSTPROBLEMID``)

host\*
   The hostname of the system that the event originated from. (``$NOTIFY_HOSTNAME``)

   Used to determine if a new event belongs to an existing ticket.
   Also used in the subject line of the resulting article (“<host> is <state>”).

service
   The name of the service that the event originated from. (``$NOTIFY_SERVICEDESC``)

   Used to determine if a new event belongs to an existing ticket.

   Displayed as ``-`` when omitted.

state\*
   The current state of the service or host in question. (``$NOTIFY_SERVICESTATE`` / ``$NOTIFY_HOSTSTATE``)

   Used to detect when a ticket should be auto-closed (*i.e.,* on ``OK``/``UP``).
   Also used in the subject line of the resulting article (“<host> is <state>”).

text
   The output of the process that triggered the event. (``$NOTIFY_SERVICEOUTPUT`` / ``$NOTIFY_HOSTOUTPUT``)

   Displayed as ``-`` when omitted.

Ticket Attributes
-----------------

.. figure:: /images/system/integrations/checkmk/finding-object-names.png
   :alt: The Object Manager attribute panel displays built-in and custom attribute names.
   :align: center
   :width: 80%

   Find a complete list of ticket attributes in the Object Manager.

Pass ticket attributes to change the settings of the ticket
that will be created for this event.
(Note that these attributes will be ignored
if the event belongs to an existing ticket.)

Why would you want to do this?
Maybe you have only one IT guy,
and all system monitoring issues should be automatically assigned to him.
Or, maybe you’re creating multiple notification rules
so that database outages take higher priority
than disk space warnings.

In most cases, you’ll probably want to set one of the following:

* group
* owner
* state
* priority

but in practice, you can set almost any attribute,
including :doc:`custom ones you created through the Object Manager </system/objects>`.

.. note:: 🙅 The following attributes are **not customizable**:

   * title
   * id
   * ticket number
   * customer
   * created_by_id
   * updated_by_id

How do I know what values I can set?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

group / state / priority
   .. figure:: /images/system/integrations/checkmk/ticket-attribute-names.png
      :alt: See possible values for certain attributes in the ticket pane.
      :scale: 50%
      :align: center

   Refer to the dropdown menus in the ticket pane:

   .. code:: bash

      -F "group=Users"
      -F "state=closed"
      -F "priority=3 high"

owner
   Use an email address or username:

   .. code:: bash

      -F "owner=it@chrispresso.com"

For other attributes, you may need to play around
in the `Rails console <https://docs.zammad.org/en/latest/admin/console.html>`_
to find out what kinds of values they accept.

.. warning:: 😵 **Invalid values → unpredictable behavior**

   In some cases, a ticket will be created with the default values instead—but
   in others, it may not be created at all!
