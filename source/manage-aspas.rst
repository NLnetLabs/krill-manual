.. _doc_krill_manage_aspas:

Manage ASPA Objects
===================

.. Warning:: The IETF is still discussing and recently changed the ASPA
             object profile. Krill currently supports the previous (V0)
             version of the profile. RPKI validators may reject these
             objects.

             The change to be aware of is that the new version dropped
             the optional AFI limit for provider ASNs. Krill 0.14.0 will
             support the new profile. Any existing configurations will be
             re-imported on upgrade (excluding the AFI limit). This
             migration will remove the history for pre-0.14.0 ASPA
             configurations so that we can remove the AFI limit concept
             from the code, thus improving future maintainability.

             In short, you can experiment with ASPA objects in Krill, but
             keep the above in mind.

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
ASN can send updates. Providers can optionally be restricted to IPv4 or IPv6
only.

Krill uses the following notation style to make it easy to define such
configurations when using the CLI:

.. code-block:: text

   AS65000 => AS65001, AS65002(v4), AS65003(v6)

.. Important:: You can only have ONE ASPA configuration for each customer ASN.
              This is because Krill MUST (RFC) create a single ASPA object, for
              all provider ASNs.

              The provider list may not be empty, and it must have entries
              for both IPv4 and IPv6. If you explicitly want to state that
              your customer AS has providers for one AFI, but not for the
              other then you will need to add either AS0(v4) or AS0(v6).

Add an ASPA
-----------

You can add ASPA definition using the following command:

.. code-block:: text

  $ krillc aspas add --aspa "AS65000 => AS65001, AS65002(v4), AS65003(v6)"

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
        "customer": "AS65000",
        "providers": [
          "AS65001",
          "AS65002(v4)",
          "AS65003(v6)"
        ]
      }
    ],
    "remove": []
  }


List ASPAs
----------

CLI:

.. code-block:: text

  $ krillc aspas list
  AS65000 => AS65001, AS65002(v4), AS65003(v6)


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
      "customer": "AS65000",
      "providers": [
        "AS65001",
        "AS65002(v4)",
        "AS65003(v6)"
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
      "AS65005"
    ],
    "removed": [
      "AS65001"
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
      "AS65000"
    ]
  }
