.. _doc_krill_failure_scenarios:

Krill Failure and Recovery Scenarios
====================================


Krill CA temporarily unavailable
--------------------------------

Issue: the Krill instance for your CA is unavailable

Consequences:
1. You cannot change ROAs
2. You cannot change delegations to child CAs
3. Krill will not update its repository

The RPKI objects which were published by your CA will remain
unchanged. This means that as long as your manifest and CRL
are not expired, your existing ROAs will remain valid.

Krill uses a default validity time of 24 hours for manifests
and CRLs, and replaces them 8 hours before they would expire.
This means that from the moment of the outage you have 8-24
hours to get Krill back up and running and prevent that your
objects will be invalidated.

It is possible to change these defaults if you want to have
more time to deal with potential issues. However, we recommend
that you avoid using long validity times because in theory
they could make you vulnerable to replay attacks where a malicious
actor feeds old objects to RPKI validators. This attack is not
trivial, but it's not impossible either.

A reasonable compromise could be to use a validity time of 36 hours,
and have Krill reissue manifests and CRLs 24 hours before they would
expire. You can achieve this by adding the following directives
to your configuration file:

.. code-block:: text

  timing_publish_next_hours = 36
  timing_publish_hours_before_next = 24
