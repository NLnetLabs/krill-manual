.. _doc_krill_upgrade:

Upgrading to this Version
=========================





API Changes in v0.9.0
---------------------

There are a number of API changes between v0.9.0 and previous versions. The main reasons for these
changes are:

1. Krill no longer has the concept of embedded CA parent-child or repo-ca relations. If you have
   multiple CAs in a single Krill instance and/or a Publication Server, then Krill will now always
   use the official RFC protocol - even if both entities live in the same Krill instance.
2. We wanted to make the API consistent.

But most importantly: **We wanted to make the API stable so we can work towards Krill 1.0**

Here we will list all CLI commands and API calls that were changed between Krill 0.8.2 and this
version.

.. Warning:: This document is a work in progress, it is incomplete. We are in the process of
     updating it and you can expect this to be completed before 23 April 18:00 CET.


krillc parents update
^^^^^^^^^^^^^^^^^^^^^

The 'update' command has been removed and is now folded in to `krillc parents add`.

krillc parents add
^^^^^^^^^^^^^^^^^^

If you add a parent which already exists for your CA, then this will act as an 'update' instead. I.e.
the previously known :rfc:`8183` Parent Response for the parent will be replaced.

The CLI command is unchanged:

.. code-block:: text

  $ krillc parents add --ca newca --parent testbed --response ./parent-response.xml

But there were changes to the API.

Adding a parent can be done by posting XML or JSON to on of the following paths:

.. code-block:: text

  /api/v1/cas/<ca>/parents
  /api/v1/cas/<ca>/parents/<handle>

The `<handle>` is the LOCAL name that your CA will use for this parent. Regardless of how they
like to call themselves. If it is omitted then it will be extracted from the XML `parent_handle`.
If it is specified for a JSON POST but _differs_ from the `handle` in the JSON body, then an
error is returned.

The server will verify in all cases that the parent can be reached. If there was no parent for the
name a parent will be added, otherwise the parent contact details will be updated.

The JSON body has to include the local name by which the CA will refer to its parent, this is also the
name shown to the user in the UI. The local name maps to the handle field in the JSON below. The second
component is the contact. Krill used to support an embedded type, but this is no longer supported.

Instead of a JSON member under `contact` we now have "type": "rfc6492" here. We still have this type
because this allows for the notion of Trust Anchor - which we use in test setups - and it keeps the
door open to future additions (eg if there ever is an RFC 6492 bis). The remainder of the structure
is unchanged, and maps to the RFC 8183 Parent Response XML, but then in JSON format. Note that the
parent_handle is the handle that the parent wants the CA to use in messages sent to it - and it may be
different from the local name stored in handle.

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

And the default text response is still the :rfc:`8183` Parent Response XML for the parent. But, the JSON
response body was changed, and now includes an explicit "type": "rfc6492":

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

krillc repo update
^^^^^^^^^^^^^^^^^^

This command has been renamed to `krillc repo configure`:

.. code-block:: text

  $ krillc repo configure --ca newca --response ./data/new-ca-repository-response.xml

The API has also changed. The path is unchanged, but the following to add an "embedded" repository
is **no longer supported**:

.. code-block:: text

  {
    "tag": "string",
    "id_cert": "string",
    "child_handle": "string"
  }

The API end-point will accept either plain :rfc:`8183` Repository Response XML, or a JSON
equivalent. In comparison to previous versions of Krill `rfc8181` was renamed to `repository_response`:

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

The CLI command and API path are unchanged, but `rfc8181` was renamed to `repository_response` in
the JSON response.


krillc history and  krillc action
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The API and JSON are unchanged, but these commands have now been renamed to `krillc history commands`
and `krillc history details`.
