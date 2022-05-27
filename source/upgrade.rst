.. _doc_krill_upgrade:

Upgrading Krill
===============

Upgrade
-------

Krill upgrades may sometimes require that existing data is migrated
to a new format used by the new release. Krill will perform these
migrations automatically in case you install a new version of Krill
and restart it.

As the first step of this upgrade, any data that needs to be migrated
is prepared under a new directory called :file:`upgrade-data` under the
:file:`data_dir` you configured. If you used a package to install Krill
then the latter would be :file:`/var/lib/krill/data`.

If all is well then Krill will rename directories under the :file:`data_dir`
and archive your old data structures under directories called
:samp:`arch_cas_{version}` and/or :samp:`arch_pubd_{version}`. You can
safely remove these directories in order to save space later.

It is unlikely that a data migration should fail. We use automated and
manual testing to make sure that these migrations work. But, of course
even with testing things can still go wrong. If the preparation step
fails then krill will exit with an error and refuse to start the new
version.

If this happens, then you can abort the upgrade by re-installing your
previous version of krill and starting that. And, please do let us know
by `making an issue <https://github.com/NLnetLabs/krill/issues>`_.


Prepare Upgrade with krillup
----------------------------

If the fully automated upgrade process seems a bit too scary to you,
then we recommend that you perform this step manually *before* upgrading
krill itself.

Starting with Krill 0.9.5 we have introduced a new command line tool
that can be used to help prepare for krill migrations.

If you built Krill using Cargo then you will find that a new binary
called :command:`krillup` is installed alongside with krill. But, if you are
using the packages that we provide then you can install and upgrade
this binary separately. For example on a Debian system:

.. code-block:: bash

  sudo apt install krillup

If you install and/or upgrade :command:`krillup` first, before upgrading
Krill itself then you will be able to prepare and verify an upgrade while
Krill is running. This is especially useful for large operations because
some of these upgrades can take a while. By using the separate tool any
downtime is limited. Furthermore, if the preparation should unexpectedly
fail, then there will be no need to reinstall a previous version of
Krill. You can simply abort the upgrade.

:command:`krillup` only needs to be told where your config file lives.
Here we use it to prepare an upgrade, where no actual data migration is
needed. This is not an error, so it will just report that the upgrade
does not require preparation:

.. code-block:: text

  $ krillup -c ./defaults/krill.conf
  2022-02-18 16:51:26 [INFO] Prepare upgrade using configuration file: ./defaults/krill.conf
  2022-02-18 16:51:26 [INFO] Processing data from: ./data
  2022-02-18 16:51:26 [INFO] Saving prepared data to: ./data/upgrade-data
  2022-02-18 16:51:26 [INFO] No preparation is needed for the upgrade from 0.9.3-rc1 to 0.9.5-rc1.


.. Important:: Once migrated data cannot be rolled back to the format
               of a previous Krill version. So, while an upgrade can
               be aborted, it cannot be undone — other than by restoring
               data from the point before the upgrade and accepting that
               any changes since then will have been lost.

               So, please read up on :ref:`important
               changes<doc_krill_important_changes>` to see if you would be
               affected by functionality or API changes before you upgrade.

Important Changes
-----------------

.. _doc_krill_important_changes:

v0.10.0
~~~~~~~

JSON Field Name Changes
^^^^^^^^^^^^^^^^^^^^^^^

When migrating support for RFC 6492, 8181 and 8183 into the base library
`rpki-rs` (issue #765) we renamed some fields which are also used in the
JSON structures of the Krill API:

+-------------+-----------------------+------------------------------+
| pre-0.10.0  | 0.10.0                | reason                       |                                                                        |
+=============+=======================+==============================+
| v4          | ipv4                  | More decscriptive            |
+-------------+-----------------------+------------------------------+
| v6          | ipv6                  | More decscriptive            |
+-------------+-----------------------+------------------------------+
| base_uri    | sia_base              | Term used in :rfc:`8183`     |
+-------------+-----------------------+------------------------------+
| rpki_notify | rrdp_notification_uri | Term used in :rfc:`8183`     |
+-------------+-----------------------+------------------------------+

We still accept the old names as aliases on input, but if you are parsing
JSON responses yourself then you will need to update your code to accept
the new names.

Parent Status Reporting
^^^^^^^^^^^^^^^^^^^^^^^

The parent status API and CLI text response now include the last known
full :rfc:`6492` "Resource Class List Response" content that your CA
received.


v0.9.3 to v0.9.5
~~~~~~~~~~~~~~~~

There are no API changes or data migrations.

After upgrading the Publication Server (if you run one) will use ``1`` as
the first RRDP serial number, instead of ``0``. Furthermore, you will now
be able to configure the timeout for a complete :RFC:`6492` and :RFC:`8181`
client HTTP request-response round-trip to the parent or publisher,
excluding the time required to establish the connection, using
`post_protocol_msg_timeout_seconds`.

v0.9.0/1/2 to v0.9.3
~~~~~~~~~~~~~~~~~~~~

There are no API changes, but users may want to be aware that the
'next update' time for manifests and CRLs has been changed from a
fixed 24 hours (by default) to 24 hours and a random amount of extra
time between 0 and 240 minutes (4 hours). This does not affect the
validity of objects, but may lead to surprises if you are monitoring
that republication would happen withing 17 hours after last publication
(8 hours before objects would expire). This can now take up to 21 hours
(using defaults).

Furthermore experimental ASPA support was added, but it's hidden in
the CLI until the ASPA standards reach stability in the IETF. If you
want to read more about the experimental ASPA support in Krill then
have a look here:

https://krill.docs.nlnetlabs.nl/en/prototype-aspa-support/manage-aspas.html


v0.9.0/1 to v0.9.2
~~~~~~~~~~~~~~~~~~

The Prometheus metrics have been updated. The metric ``krill_cas_roas``
has been renamed to ``krill_cas_bgp_roas_total`` for consistency. Please
have a look at the updated :ref:`monitoring page<doc_krill_monitoring>`
for more details.

v0.8.2 and below to v0.9.x
~~~~~~~~~~~~~~~~~~~~~~~~~~

There are a number of API changes between v0.9.0 and previous versions.
The main reasons for these changes are:

1. Krill no longer has the concept of embedded CA parent-child or
   repo-ca relations. If you have multiple CAs in a single Krill
   instance and/or a Publication Server, then Krill will now always
   use the official RFC protocol - even if both entities live in the
   same Krill instance.
2. We wanted to make the API consistent.

But most importantly: **We wanted to make the API stable so we can
work towards Krill 1.0**

Here we will list all CLI commands and API calls that were changed
between Krill 0.8.2 and this version. This list should be complete, so
old CLI commands not listed here should not have changed.

In case you do find something that we overlooked please let us know!

krillc parents update
^^^^^^^^^^^^^^^^^^^^^

The :command:`update` command has been removed and is now folded in to
:command:`krillc parents add`.

krillc parents add
^^^^^^^^^^^^^^^^^^

If you add a parent which already exists for your CA, then this will
act as an 'update' instead. I.e. the previously known :rfc:`8183`
Parent Response for the parent will be replaced.

The CLI command is unchanged:

.. code-block:: text

  $ krillc parents add --ca newca --parent testbed --response ./parent-response.xml

But there were changes to the API.

Adding a parent can be done by posting XML or JSON to on of the
following paths:

.. code-block:: text

  /api/v1/cas/<ca>/parents
  /api/v1/cas/<ca>/parents/<handle>

The ``<handle>`` is the LOCAL name that your CA will use for this parent.
Regardless of how they like to call themselves. If it is omitted then
it will be extracted from the XML ``parent_handle``. If it is specified
for a JSON POST but _differs_ from the ``handle`` in the JSON body, then
an error is returned.

The server will verify in all cases that the parent can be reached. If
there was no parent for the name a parent will be added, otherwise the
parent contact details will be updated.

The JSON body has to include the local name by which the CA will refer
to its parent, this is also the name shown to the user in the UI. The
local name maps to the handle field in the JSON below. The second
component is the contact. Krill used to support an embedded type, but
this is no longer supported.

Instead of a JSON member under ``contact`` we now have ``"type": "rfc6492"``
here. We still have this type because this allows for the notion of
Trust Anchor - which we use in test setups - and it keeps the door open
to future additions (eg if there ever is an RFC 6492 bis). The remainder
of the structure is unchanged, and maps to the :RFC:`8183` Parent Response
XML, but then in JSON format. Note that the parent_handle is the handle
that the parent wants the CA to use in messages sent to it - and it may
be different from the local name stored in handle.

OLD JSON:

.. code-block:: json

  {
    "handle": "testbed",
    "contact": {
      "rfc6492": {
        "tag": null,
        "id_cert": "MIIDNDCCAhygAwIBAgIBATANBgkqhkiG9w0BAQsFADAzMTEwLwYDVQQDEyg5N0VEOUFCMUQ4Q0JBNzFBMTJEQjE2MTU4OTA3Njk4QUI0QTAzMUQ5MB4XDTIwMDkxNjA5MTAxMloXDTM1MDkxNjA5MTUxMlowMzExMC8GA1UEAxMoOTdFRDlBQjFEOENCQTcxQTEyREIxNjE1ODkwNzY5OEFCNEEwMzFEOTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMnEJkDvrR7iY0VoRGvajDWxo2krplOnZynM1kgXtN8L3StS6YE7/sXvoG1C1pRPs/SBZ7gK6WvFlqdScZ6kbTVH51e+pLUV9Q7Uxqm4lSzWTmnjT/CmRRXqmcPlcPcAm8rhUW6GrZQ2mllil4pkZ+JNGugSQUJJb1bGg5+Et/YdIEDEO1stAIsNkfkAyELAeFULLhs0MuXpSKp/ZKu+IgMSt+Z/7is+qFt4cgMuiZRuADw8hTDoMuZpoxIqXeh4Nf3bUU06MXGgrpabVzArs11UVyXDC4ZG4oOsYDTNgIL5VYaBjiHtw+s+FWHYI3iTzwV8th2C1JI6LOOBkxZdxQUCAwEAAaNTMFEwDwYDVR0TAQH/BAUwAwEB/zAdBgNVHQ4EFgQUl+2asdjLpxoS2xYViQdpirSgMdkwHwYDVR0jBBgwFoAUl+2asdjLpxoS2xYViQdpirSgMdkwDQYJKoZIhvcNAQELBQADggEBAB34RLGHufEpypzvDFzffkS7Oet9TUZSV1nB7EPGA7BJLvUnJt2SAv+0LhFRup518oQMpeM8HxA7vcRMt6JNTWydW/bYp/NnAk+u+Hw5AIwxuoGWgwyHXZh1xJFhwD35SqjMhxbo15J090+22zwAa8t6aqQAZhvACs2Jst1aHnnJEduQzGVZYLIYvGv5/K0t0i0eE5hINhtAy0hFGwteXms8/qA/mExsrjubC69SudPlMA3q8p2RmuwqmjSlwDjU1XrJ1j5wMCqeBoh8EnaMe+HVduQGHm0nHJbF3klz9mz3Tc6CILT4XA5mJq1g0LXypJ9c6KxZFoC10ce/enulLYw=",
        "parent_handle": "testbed",
        "child_handle": "newca",
        "service_uri": "https://testbed.rpki.nlnetlabs.nl/rfc6492/testbed"
      }
    }
  }

Was changed to:

.. code-block:: json

  {
    "handle": "my_parent",
    "contact": {
      "type": "rfc6492",
      "tag": null,
      "id_cert": "MIIDNDCCAhygAwIBAgIBATANBgkqhkiG9w0BAQsFADAzMTEwLwYDVQQDEyhFOTBDMjE3MzRDMkMzNzBBOTFBODQ3NUNCNEYwRTc1REE0RDBGMEJGMB4XDTIxMDMyOTA3NTg0NFoXDTM2MDMyOTA4MDM0NFowMzExMC8GA1UEAxMoRTkwQzIxNzM0QzJDMzcwQTkxQTg0NzVDQjRGMEU3NURBNEQwRjBCRjCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBANcL8DFS3AQyI8HewRH2Xkh6RNIfCSb7mJDaS6dHwp2Dns0VZ07SjA/vVYxq1F1w2yQ/VoTr1dvEHxJ+SDayMcFVktWCObiY8tcPhvWG+OdaX9ckDJhsOEEvdVEogwiGacNs7yXJPbqDBptJtbR8/CauF9OqMqjkB/8xkGmBoY5OI/V2832jkp7LPsbyET0RMQN7fgSpGbewvkaZVxGU3pHh5kT1nzPTXrwjxNMXgpunSEY7zR20vYCvsYYbxnSwFNbSMSL+Jgpa+HWPUc0ydqk2Dn3XneHqClu3O37URxcvI+th4+rECNp6/qlqlZK+tkppI2LkSBhTV5+n7cGA8ZsCAwEAAaNTMFEwDwYDVR0TAQH/BAUwAwEB/zAdBgNVHQ4EFgQU6Qwhc0wsNwqRqEdctPDnXaTQ8L8wHwYDVR0jBBgwFoAU6Qwhc0wsNwqRqEdctPDnXaTQ8L8wDQYJKoZIhvcNAQELBQADggEBAG9DNu26d2S9b15NzzaArLg3Ac/nVmqDlK/1sWZNUXFWP4dt1wLTjDWnceyS8mI7Yx8dH/Fez60m4lp4dD45eeaXfbjP2cWnh3n/PLGE70Nj+G0AnUhUmwiTl0H6Px1xn8fZouhv9MEheaZJA+M4NF77+Nmkp2P3WI4cvIS7Te7R/7XpwSr29lVNtYjmRlrBDXx/bMFSgFL61mrtj/l6G8OB40w+sAwO0XKUj1vUUpfIXc3ISCo0LNT9JSPcgy1SZWfmLb98q4HuvxekhkIPRzW7vlb/NBXGarZmKc+HQjE2aXcIewhen2OoTSNda2jSSuEWZuWzZu0aMCKwFBNHLqs=",
      "parent_handle": "testbed",
      "child_handle": "newca",
      "service_uri": "https://localhost:3000/rfc6492/testbed"
    }
  }

krillc parents contact
^^^^^^^^^^^^^^^^^^^^^^

The CLI command was unchanged:

.. code-block:: text

  $ krillc parents contact --parent testbed

And the default text response is still the :rfc:`8183` Parent Response
XML for the parent. But, the JSON response body was changed, and now
includes an explicit ``"type": "rfc6492"``:

OLD:

.. code-block:: text

  {
    "rfc6492": {
      "tag": null,
      "id_cert": "MIIDNDCCAhygAwIBAgIBATANBgkqhkiG9w0BAQsFADAzMTEwLwYDVQQDEyg5N0VEOUFCMUQ4Q0JBNzFBMTJEQjE2MTU4OTA3Njk4QUI0QTAzMUQ5MB4XDTIwMDkxNjA5MTAxMloXDTM1MDkxNjA5MTUxMlowMzExMC8GA1UEAxMoOTdFRDlBQjFEOENCQTcxQTEyREIxNjE1ODkwNzY5OEFCNEEwMzFEOTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMnEJkDvrR7iY0VoRGvajDWxo2krplOnZynM1kgXtN8L3StS6YE7/sXvoG1C1pRPs/SBZ7gK6WvFlqdScZ6kbTVH51e+pLUV9Q7Uxqm4lSzWTmnjT/CmRRXqmcPlcPcAm8rhUW6GrZQ2mllil4pkZ+JNGugSQUJJb1bGg5+Et/YdIEDEO1stAIsNkfkAyELAeFULLhs0MuXpSKp/ZKu+IgMSt+Z/7is+qFt4cgMuiZRuADw8hTDoMuZpoxIqXeh4Nf3bUU06MXGgrpabVzArs11UVyXDC4ZG4oOsYDTNgIL5VYaBjiHtw+s+FWHYI3iTzwV8th2C1JI6LOOBkxZdxQUCAwEAAaNTMFEwDwYDVR0TAQH/BAUwAwEB/zAdBgNVHQ4EFgQUl+2asdjLpxoS2xYViQdpirSgMdkwHwYDVR0jBBgwFoAUl+2asdjLpxoS2xYViQdpirSgMdkwDQYJKoZIhvcNAQELBQADggEBAB34RLGHufEpypzvDFzffkS7Oet9TUZSV1nB7EPGA7BJLvUnJt2SAv+0LhFRup518oQMpeM8HxA7vcRMt6JNTWydW/bYp/NnAk+u+Hw5AIwxuoGWgwyHXZh1xJFhwD35SqjMhxbo15J090+22zwAa8t6aqQAZhvACs2Jst1aHnnJEduQzGVZYLIYvGv5/K0t0i0eE5hINhtAy0hFGwteXms8/qA/mExsrjubC69SudPlMA3q8p2RmuwqmjSlwDjU1XrJ1j5wMCqeBoh8EnaMe+HVduQGHm0nHJbF3klz9mz3Tc6CILT4XA5mJq1g0LXypJ9c6KxZFoC10ce/enulLYw=",
      "parent_handle": "testbed",
      "child_handle": "newca",
      "service_uri": "https://testbed.rpki.nlnetlabs.nl/rfc6492/testbed"
    }
  }

NEW:

.. code-block:: text

  {
    "type": "rfc6492",
    "tag": null,
    "id_cert": "MIIDNDCCAhygAwIBAgIBATANBgkqhkiG9w0BAQsFADAzMTEwLwYDVQQDEyg5N0VEOUFCMUQ4Q0JBNzFBMTJEQjE2MTU4OTA3Njk4QUI0QTAzMUQ5MB4XDTIwMDkxNjA5MTAxMloXDTM1MDkxNjA5MTUxMlowMzExMC8GA1UEAxMoOTdFRDlBQjFEOENCQTcxQTEyREIxNjE1ODkwNzY5OEFCNEEwMzFEOTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMnEJkDvrR7iY0VoRGvajDWxo2krplOnZynM1kgXtN8L3StS6YE7/sXvoG1C1pRPs/SBZ7gK6WvFlqdScZ6kbTVH51e+pLUV9Q7Uxqm4lSzWTmnjT/CmRRXqmcPlcPcAm8rhUW6GrZQ2mllil4pkZ+JNGugSQUJJb1bGg5+Et/YdIEDEO1stAIsNkfkAyELAeFULLhs0MuXpSKp/ZKu+IgMSt+Z/7is+qFt4cgMuiZRuADw8hTDoMuZpoxIqXeh4Nf3bUU06MXGgrpabVzArs11UVyXDC4ZG4oOsYDTNgIL5VYaBjiHtw+s+FWHYI3iTzwV8th2C1JI6LOOBkxZdxQUCAwEAAaNTMFEwDwYDVR0TAQH/BAUwAwEB/zAdBgNVHQ4EFgQUl+2asdjLpxoS2xYViQdpirSgMdkwHwYDVR0jBBgwFoAUl+2asdjLpxoS2xYViQdpirSgMdkwDQYJKoZIhvcNAQELBQADggEBAB34RLGHufEpypzvDFzffkS7Oet9TUZSV1nB7EPGA7BJLvUnJt2SAv+0LhFRup518oQMpeM8HxA7vcRMt6JNTWydW/bYp/NnAk+u+Hw5AIwxuoGWgwyHXZh1xJFhwD35SqjMhxbo15J090+22zwAa8t6aqQAZhvACs2Jst1aHnnJEduQzGVZYLIYvGv5/K0t0i0eE5hINhtAy0hFGwteXms8/qA/mExsrjubC69SudPlMA3q8p2RmuwqmjSlwDjU1XrJ1j5wMCqeBoh8EnaMe+HVduQGHm0nHJbF3klz9mz3Tc6CILT4XA5mJq1g0LXypJ9c6KxZFoC10ce/enulLYw=",
    "parent_handle": "ta",
    "child_handle": "testbed",
    "service_uri": "https://localhost:3000/rfc6492/ta"
  }

krillc repo request
^^^^^^^^^^^^^^^^^^^

The CLI is unchanged, but the endpoints for getting the :rfc:`8183`
Publisher Request XML and JSON have moved from :file:`repo`, and are now
under :file:`id`:

.. code-block:: text

  /api/v1/cas/<name>/repo/request.xml  -> /api/v1/cas/<name>/id/publisher_request.xml
  /api/v1/cas/<name>/repo/request.json -> /api/v1/cas/<name>/id/publisher_request.json


krillc repo update
^^^^^^^^^^^^^^^^^^

This command has been renamed to :command:`krillc repo configure`:

.. code-block:: text

  $ krillc repo configure --ca newca --response ./data/new-ca-repository-response.xml

The API has also changed. The path is unchanged, but the following to
add an "embedded" repository is **no longer supported**:

.. code-block:: text

  {
    "tag": "string",
    "id_cert": "string",
    "child_handle": "string"
  }

The API end-point will accept either plain :rfc:`8183` Repository
Response XML, or a JSON equivalent. In comparison to previous versions
of Krill `rfc8181` was renamed to `repository_response`:

.. code-block:: text

  {
    "repository_response": {
      "tag": null,
      "publisher_handle": "publisher",
      "id_cert": "MIID..6g==",
      "service_uri": "https://repo.example.com/rfc8181/publisher/",
      "repo_info": {
        "base_uri": "rsync://localhost/repo/ca/",
        "rpki_notify": "https://localhost:3000/rrdp/notification.xml"
      }
    }
  }

krillc repo show
^^^^^^^^^^^^^^^^

The CLI command and API path are unchanged, but ``rfc8181`` was renamed
to ``repository_response`` in the JSON response.


krillc children add
^^^^^^^^^^^^^^^^^^^

The CLI is unchanged, but because 'embedded' children are no longer
supported we were able to simplify the JSON from:

.. code-block:: text

  {
    "handle": "ca",
    "resources": {
      "asn": "AS1",
      "v4": "10.0.0.0/8",
      "v6": "::"
    },
    "auth": {
      "rfc8183": {
        "tag": null,
        "child_handle": "ca",
        "id_cert": "<base64>"
      }
    }
  }

To this:

.. code-block:: text

  {
    "handle": "ca",
    "resources": {
      "asn": "AS1",
      "v4": "10.0.0.0/8",
      "v6": "::"
    },
    "id_cert": "<base64>"
  }


krillc history and krillc action
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The API and JSON are unchanged, but these commands have now been
renamed to ``krillc history commands`` and ``krillc history details``.
