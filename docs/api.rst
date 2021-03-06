Raiden's API Documentation
##########################


Contents:

.. toctree::
   :maxdepth: 2



Introduction
*************
Raiden has a Restful API with URL endpoints corresponding to actions that the user can perform with his channels. The endpoints accept and return JSON encoded objects. The api query string always contains the api version in order to differentiate queries to
different API versions. All queries start with: ``/api/<version>/``



JSON Object Encoding
********************

The ojects that are sent to and received from the API are JSON-encoded. Following are the common objects used in the API.

Channel Object
===============
::

    {
       'channel_address': '0x2a65aca4d5fc5b5c859090a6c34d164135398226',
       'partner_address': '0x61c808d82a3ac53231750dadc13c777b59310bd9',
       'token_address': '0xea674fdde714fd979de3edf0f56aa9716b898ec8',
       'balance': 35000000,
       'state': 'open',
       'settle_timeout': 100
    }



A channel object consists of a

- ``channel_address`` should be a ``string`` containing the hexadecimal address of the
  channel

- ``partner_address`` should be a ``string`` containing the hexadecimal address of the
  partner with whom we have opened a channel

- ``token_address`` should be a ``string`` containing the hexadecimal address of the
  token we are trading in the channel.

- ``balance`` should be an integer of the amount of the ``token_address`` token we have
  deposited in the channel.

- ``state`` should be the current state of the channel represented by a string.
  Possible value are:
  - ``'open'``: The channel is open and tokens are tradeable
  - ``'closed'``: The channel has been closed by a participant
  - ``'settled'``: The channel has been closed by a participant and also settled.

- ``'settle_timeout'``: The number of blocks that are required to be mined from the time that ``close()`` is called
                        until the channel can be settled with a call to ``settle()``.


Event Object
==============

Channels events are encoded as json objects with the event arguments as attributes
of the dictionary, with one difference. The ``event_type`` is also added for all events to easily distinguish between events.






Endpoints
***********

Following are the available API endpoints with which you can interact with Raiden.

Querying Information About Channels and Assets
===============================================

Querying specific channel
--------------------------

There are multiple ways to query information about your channels. The most direct, if you know the channel address, is to query the channel master resource endpoint ``/api/<version>/channels/<channel_address`` with a ``GET`` request.

Example Request
^^^^^^^^^^^^^^^

``GET /api/1/channels/0x2a65aca4d5fc5b5c859090a6c34d164135398226/``

Example Response
^^^^^^^^^^^^^^^^
::

    {
        'channel_address': '0x2a65aca4d5fc5b5c859090a6c34d164135398226',
        'partner_address': '0x61c808d82a3ac53231750dadc13c777b59310bd9',
        'token_address': '0xea674fdde714fd979de3edf0f56aa9716b898ec8',
        'balance': 35000000,
        'state': 'open',
        'settle_timeout': 100
    }


The other ways to query information about the channels is to navigate the document
hierarchy of the API and find the channel address which will lead you to the master
resource as shown above.

Querying All Channels
--------------------------

By making a ``GET`` request to ``/api/<version>/channels`` you can get a list of all non-settled channels.


Example Request
^^^^^^^^^^^^^^^

``GET /api/1/channels/``

Example Response
^^^^^^^^^^^^^^^^

``200 OK`` and

::

    [
        {
            'channel_address': '0x2a65aca4d5fc5b5c859090a6c34d164135398226',
            'partner_address': '0x61c808d82a3ac53231750dadc13c777b59310bd9',
            'token_address': '0xea674fdde714fd979de3edf0f56aa9716b898ec8',
            'balance': 35000000,
            'state': 'open',
            'settle_timeout': 100
        }, {
            ...
        }
    ]

Possible Responses
^^^^^^^^^^^^^^^^^^

+------------------+---------------------------+
| HTTP Code        | Condition                 |
+==================+===========================+
| 200 OK           | For a succesful Query     |
+------------------+---------------------------+
| 500 Server Error | Internal Raiden node error|
+------------------+---------------------------+



Querying all traded Assets
--------------------------

By making a ``GET`` request to ``/api/<version>/tokens`` you can get a list of addresses of all
tokens we have channels open for.


Example Request
^^^^^^^^^^^^^^^

``GET /api/1/tokens/``

Example Response
^^^^^^^^^^^^^^^^
``200 OK`` and

::

    {
        ['0xea674fdde714fd979de3edf0f56aa9716b898ec8',
         '0x61bb630d3b2e8eda0fc1d50f9f958ec02e3969f6',
         ...
        ]
    }

Possible Responses
^^^^^^^^^^^^^^^^^^

+------------------+---------------------------+
| HTTP Code        | Condition                 |
+==================+===========================+
| 200 OK           | For a succesful Query     |
+------------------+---------------------------+
| 500 Server Error | Internal Raiden node error|
+------------------+---------------------------+


Querying All Partners for an Asset
-----------------------------------

By making a ``GET`` request to ``/api/<version>/tokens/<token_address>/partners`` you can get a list of all partners
you have non-settled channels with.

Example Request
^^^^^^^^^^^^^^^

``GET /api/1/tokens/0x61bb630d3b2e8eda0fc1d50f9f958ec02e3969f6/partners/``

Example Response
^^^^^^^^^^^^^^^^
``200 OK``

::


    [
        {
            'partner_address': '0x61c808d82a3ac53231750dadc13c777b59310bd9',,
            'channel': '/api/channels/0x2a65aca4d5fc5b5c859090a6c34d164135398226'
        }, {
            ...
        }
    ]


Notice that you also get a link to the channel resource for the channel between you
and each partner for the token.

Possible Responses
^^^^^^^^^^^^^^^^^^

+------------------+---------------------------+
| HTTP Code        | Condition                 |
+==================+===========================+
| 200 OK           | For a succesful Query     |
+------------------+---------------------------+
| 302 Redirect     | If the user accesses the  |
|                  | channel link endpoint     |
+------------------+---------------------------+
| 500 Server Error | Internal Raiden node error|
+------------------+---------------------------+



Channel Management
==================

Open Channel
------------

You can create a channel by posting a channel object to the following endpoint.

``PUT /api/<version>/channels``

Since it is a new channel, the channel object's ``channel_address`` and ``status``
field will be ignored and can be omitted.

Example Request
^^^^^^^^^^^^^^^

``PUT /api/1/channels`` with payload:::


    {
        'partner_address': '0x61c808d82a3ac53231750dadc13c777b59310bd9',
        'token_address': '0xea674fdde714fd979de3edf0f56aa9716b898ec8',
        'balance': 35000000,
        'settle_timeout': 100
    }



The ``balance`` field will signify the initial deposit you wish to make to the channel.

The request to the endpoint should later return the fully created channel object
from which we can find the address of the channel.

Example Response
^^^^^^^^^^^^^^^^
``201 Created`` and
::

    {
        'channel_address': '0x2a65aca4d5fc5b5c859090a6c34d164135398226',
        'partner_address': '0x61c808d82a3ac53231750dadc13c777b59310bd9',
        'token_address': '0xea674fdde714fd979de3edf0f56aa9716b898ec8',
        'balance': 35000000,
        'state': 'open',
        'settle_timeout': 100
    }

Possible Responses
^^^^^^^^^^^^^^^^^^

+------------------+---------------------------+
| HTTP Code        | Condition                 |
+==================+===========================+
| 201 Created      | For succesful Creation    |
+------------------+---------------------------+
| 400 Bad Request  | If the provided json is in|
|                  | some way malformed        |
+------------------+---------------------------+
| 500 Server Error | Internal Raiden node error|
+------------------+---------------------------+


Close Channel
--------------

You can close an existing channel by making a ``PATCH`` request to its endpoint and altering the state to closed.

``PATCH /api/<version>/channels/<channel_address>/``

Example Request
^^^^^^^^^^^^^^^

``PATCH /api/1/channels/0x2a65aca4d5fc5b5c859090a6c34d164135398226/``

with payload
``{'state':'closed'}``

Example Response
^^^^^^^^^^^^^^^^
``200 OK`` with
::

    {
        'channel_address': '0x2a65aca4d5fc5b5c859090a6c34d164135398226',
        'partner_address': '0x61c808d82a3ac53231750dadc13c777b59310bd9',
        'token_address': '0xea674fdde714fd979de3edf0f56aa9716b898ec8',
        'balance': 35000000,
        'state': 'closed',
        'settle_timeout': 100
    }

Possible Responses
^^^^^^^^^^^^^^^^^^

+------------------+---------------------------+
| HTTP Code        | Condition                 |
+==================+===========================+
| 200 OK           | For succesful Closing     |
+------------------+---------------------------+
| 400 Bad Request  | If the provided json is in|
|                  | some way malformed        |
+------------------+---------------------------+
| 500 Server Error | Internal Raiden node error|
+------------------+---------------------------+


Settle Channel
---------------

You can settle an existing channel by making a ``PATCH`` request to its endpoint and altering the state to settled.

``PATCH /api/<version>/channels/<channel_address>/``

Example Request
^^^^^^^^^^^^^^^

``PATCH /api/1/channels/0x2a65aca4d5fc5b5c859090a6c34d164135398226/``

with payload
``{'state':'settled'}``

Example Response
^^^^^^^^^^^^^^^^
``200 OK`` with
::

    {
        'channel_address': '0x2a65aca4d5fc5b5c859090a6c34d164135398226',
        'partner_address': '0x61c808d82a3ac53231750dadc13c777b59310bd9',
        'token_address': '0xea674fdde714fd979de3edf0f56aa9716b898ec8',
        'balance': 0,
        'state': 'settled',
        'settle_timeout': 100
    }

Possible Responses
^^^^^^^^^^^^^^^^^^

+------------------+---------------------------+
| HTTP Code        | Condition                 |
+==================+===========================+
| 200 OK           | For succesful Settlement  |
+------------------+---------------------------+
| 400 Bad Request  | If the provided json is in|
|                  | some way malformed        |
+------------------+---------------------------+
| 500 Server Error | Internal Raiden node error|
+------------------+---------------------------+

Deposit to a Channel
---------------------

You can deposit more of a particular token to a channel by updating the ``balance``
field of the channel in the corresponding endpoint with a ``PATCH`` http request.

``PATCH /api/<version>/channels/<channel_address>/``

Example Request
^^^^^^^^^^^^^^^

``PATCH /api/1/channels/0x2a65aca4d5fc5b5c859090a6c34d164135398226/``

with payload

``{'balance': 100}``

Example Response
^^^^^^^^^^^^^^^^
``200 OK`` with
::

    {
        'channel_address': '0x2a65aca4d5fc5b5c859090a6c34d164135398226',
        'partner_address': '0x61c808d82a3ac53231750dadc13c777b59310bd9',
        'token_address': '0xea674fdde714fd979de3edf0f56aa9716b898ec8',
        'balance': 100,
        'state': 'open',
        'settle_timeout': 100
    }

Possible Responses
^^^^^^^^^^^^^^^^^^

+------------------+---------------------------+
| HTTP Code        | Condition                 |
+==================+===========================+
| 200 OK           | For succesful Deposit     |
+------------------+---------------------------+
| 400 Bad Request  | If the provided json is in|
|                  | some way malformed        |
+------------------+---------------------------+
| 500 Server Error | Internal Raiden node error|
+------------------+---------------------------+

Querying Events
================

Events are kept by the node. Once an event endpoint is queried the relevant events
from either the beginning of time or the given block are returned.

Events are queried by two different endpoints depending on whether they are related
to a specific channel or not.

All events can be filtered down by providing the query string argument ``from_block``
to signify the block from which you would like the events to be returned.

Querying general network events
---------------------------------


You can query for non-channel specific events by making a ``GET`` request to the
endpoint of the token registry contract. ``GET /api/<version>/events/network/<token_registry_address>``

Example Request
^^^^^^^^^^^^^^^

``GET /api/1/events/network/0x4bb96091ee9d802ed039c4d1a5f6216f90f81b01``

Example Response
^^^^^^^^^^^^^^^^
``200 OK`` with
::

    [
        {
            'event_type': 'AssetAdded',
            'token_address': '0xea674fdde714fd979de3edf0f56aa9716b898ec8',
            'channel_manager_address': '0xc0ea08a2d404d3172d2add29a45be56da40e2949'
        }, {
            'event_type': 'AssetAdded',
            'token_address': '0x91337a300e0361bddb2e377dd4e88ccb7796663d'
            'channel_manager_address': '0xc0ea08a2d404d3172d2add29a45be56da40e2949'
        }, {
            ...
        }
        ...
    ]

Possible Responses
^^^^^^^^^^^^^^^^^^

+------------------+---------------------------+
| HTTP Code        | Condition                 |
+==================+===========================+
| 200 OK           | For succesful Query       |
+------------------+---------------------------+
| 400 Bad Request  | If the provided query     |
|                  | string is  malformed      |
+------------------+---------------------------+
| 500 Server Error | Internal Raiden node error|
+------------------+---------------------------+

Querying channel events
------------------------

You can query for events tied to a specific channel by making a ``GET`` request to the event endpoint of its address. ``GET /api/<version>/events/channels/<channel_registry_address>``

Example Request
^^^^^^^^^^^^^^^

``GET /api/1/events/channels/0x2a65aca4d5fc5b5c859090a6c34d164135398226?from_block=1337``

Example Response
^^^^^^^^^^^^^^^^
``200 OK`` with
::

    [
        {
            'event_type': 'ChannelNewBalance',
            'participant': '0xea674fdde714fd979de3edf0f56aa9716b898ec8',
            'balance': 150000,
            'block_number': 54388
        }, {
            'event_type': 'TransferUpdated',
            'token_address': '0x91337a300e0361bddb2e377dd4e88ccb7796663d',
            'channel_manager_address': '0xc0ea08a2d404d3172d2add29a45be56da40e2949'
        },
        ...
    ]

Possible Responses
^^^^^^^^^^^^^^^^^^

+------------------+---------------------------+
| HTTP Code        | Condition                 |
+==================+===========================+
| 200 OK           | For succesful Query       |
+------------------+---------------------------+
| 400 Bad Request  | If the provided query     |
|                  | string is  malformed      |
+------------------+---------------------------+
| 500 Server Error | Internal Raiden node error|
+------------------+---------------------------+
