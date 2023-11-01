.. _doc_krill_manage_aspas:

Manage ASPA Objects
===================

.. Warning:: The IETF is still discussing and recently changed the ASPA
             object profile. Krill currently supports the latest version
             of the profile.

             Any existing ASPA objects that you created before Krill
             0.14.0 will be recreated on upgrade. If you used the now
             deprecated option to limit a provider to a certain AFI,
             then that limit will be silently dropped on upgrade.

             Also note that ASPA objects are only supported in the API
             and CLI for the moment. The API changed in 0.14.0 and will
             now use simple numbers for the ASNs in JSON. The CLI still
             uses the more human friendly string notation which allows
             both simple numners, e.g. "65000", or "AS65000" to be used.

             We plan to add support for ASPA management in the UI in
             future, probably shortly after the ASPA profile will have
             made through IETF last call.

ASPA Configurations
-------------------

As with ROA support, Krill lets operators *define* the ASPA *configurations*
for which they want to have ASPA objects. The actual ASPA *objects* are then
created by Krill under any parent where the 'customer AS' is in the set of
received resources. I.e. if theoretically your CA would receive this same ASN
under two different parents, then Krill would create an ASPA object with the
same content under each.

Furthermore, just like with ROAs, Krill issues these objects with a default
validity time of 52 weeks, and will automatically re-issue these objects 4 weeks
before they would expire - as long as a configuration still exists and the
customer ASN is held by your CA.

ASPA Configuration Notation
---------------------------

ASPA objects allow operators to specify a list of provider ASNs, in the sense
of BGP rather than in terms of business relations, where their own 'customer'
ASN can send updates.

Krill uses the following notation style to make it easy to define such
configurations when using the CLI:

.. code-block:: text

   AS65000 => AS65001, AS65002, AS65003

.. Important:: You can only have ONE ASPA configuration for each customer ASN.
              This is because Krill MUST (RFC) create a single ASPA object, for
              all provider ASNs. The provider list may not be empty.

Add or Update an ASPA
---------------------

You can add a new, or update (replace) any existing, ASPA definition using
the following command:

.. code-block:: text

  $ krillc aspas add --aspa "AS65000 => AS65001, AS65002, AS65003"

This uses the following API call:

.. code-block:: text
  POST:
    https://localhost:3000/api/v1/cas/ca/aspas
  Headers:
    content-type: application/json
    Authorization: Bearer secret
  Body:
  {
    "add_or_replace": [
      {
        "customer": 65000,
        "providers": [
          65000,
          65002,
          65003
        ]
      }
    ],
    "remove": []
  }


.. Note:: The CLI only allows the addition of one ASPA configuration at
        a time. But the API allows multiple additions or updates and/or
        removals to be sent in a single call.


List ASPAs
----------

CLI:

.. code-block:: text

  $ krillc aspas list
  AS65000 => AS65001, AS65002, AS65003


API:

.. code-block:: text

  GET:
    https://localhost:3000/api/v1/cas/ca/aspas
  Headers:
    Authorization: Bearer secret

JSON response:

.. code-block:: text

  $ krillc aspas list --format json
  [
    {
      "customer": 65000,
      "providers": [
        65001,
        65002,
        65003
      ]
    }
  ]


Update an ASPA
---------------

You can add or remove providers to/from the ASPA configuration for one of
your customer ASNs:

Using the CLI:

.. code-block:: text

  $ krillc aspas update --customer AS65000 --add "AS65005" --remove "AS65001"

Or using the API:

.. code-block:: text

  krillc aspas update --customer AS65000 --add "AS65005" --remove "AS65001" --api
  POST:
    https://localhost:3000/api/v1/cas/ca/aspas/as/AS65000
  Headers:
    content-type: application/json
    Authorization: Bearer secret
  Body:
  {
    "added": [
      65005
    ],
    "removed": [
      65001
    ]
  }

.. Tip:: The update function is designed to be idempotent. You can use
         this function to add a provider for your customer AS, even if
         you did not yet have any ASPA defined. Krill will then just
         create a new ASPA config for you. If you try adding a provider
         that is already listed for the customer, then the operation will
         simply have no effect. If you remove the last provider, then
         Krill will remove the entire ASPA configuration and object for
         your customer AS.

Remove an ASPA
---------------

You can remove the ASPA configuration for a given customer ASN.

Using the CLI:

.. code-block:: text

  $ krillc aspas remove --customer AS65000

Or using the API:

.. code-block:: text

  krillc aspas remove --customer AS65000 --api
  POST:
    https://localhost:3000/api/v1/cas/ca/aspas
  Headers:
    content-type: application/json
    Authorization: Bearer secret
  Body:
  {
    "add_or_replace": [],
    "remove": [
      65000
    ]
  }
