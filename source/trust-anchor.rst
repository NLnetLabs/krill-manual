.. _doc_krill_trust_anchor:

Krill as a Trust Anchor
=======================

Krill can be set up to operate an RPKI Trust Anchor (TA). An RPKI TA
serves as an `entry point for RPKI validators <https://rpki.readthedocs.io/en/latest/rpki/using-rpki-data.html#connecting-to-the-trust-anchor>`_.
There are currently `five globally used TAs <https://rpki.readthedocs.io/en/latest/rpki/introduction.html#mapping-the-resource-allocation-hierarchy-into-the-rpki>`_
operated by the five RIRs, where each RIR is responsible for IPv4, IPv6
and AS number resources that are allocated to them by IANA.

If you are not an RIR, then you will not need to run your own RPKI TA for
normal RPKI operations. You would operate one or more RPKI CAs that get
their IPv4, IPv6 and ASN number resources under one or more of the RIR
operated TAs.

That said, some users may want to operate their own TA outside of the
TAs provided by the RIRs for testing, study or research reasons. Or perhaps
even to manage private use address space.

Furthermore, this documentation may be of interest to readers who simply
wish to understand how Krill is used to the operate a TA.

Overview
^^^^^^^^

The Krill TA is logically separated into one 'Proxy' and one 'Signer'
component which are associated with each other.

The TA Signer is responsible for generating and using the TA RPKI key. It
is designed to be operated using its own standalone command line tool
called ``krillta``. For improved security this tool can be used on a
system that is kept disconnected from the network and offline when it is
not in use, and optionally an HSM could be used for handling the key.

The TA Proxy always lives inside Krill itself and is responsible for all
*online* operations such as handling :rfc:`6492` communications with a
child CA and publishing materials signed by the TA Signer using the
:rfc:`8181` communication protocol with a Publication Server. The TA
Proxy uses its own "identity" key and certificate for these protocols.

The TA Proxy is responsible for managing which child CA(s) can operate
under the TA, and what resources they are entitled to. When the TA Proxy
first receives an :rfc:`6492` request from a child they reply with a
dedicated not-performed-response (code 1104) indicating that the request is
scheduled for processing. The child can (and does) keep sending the same
request, and it will get another dedicated not-performed-response (code 1101)
indicating that the request was already scheduled.

Contrary to the other not-performed-responses these codes do not indicate
that any error occurred. These codes exist to support the asynchronous
signing by the offline TA Signer.

The actual signing process is initiated by the TA Proxy which generates
a request for the signer through the CLI/API. The TA Signer then processes
the request, does all the necessary signing and generates a response.
This response is then given to the TA Proxy which will then publish any
new signed objects. The TA Proxy will also keep the now completed response
for the child CA, which will be returned the next time that the child CA
sends their request to the TA Proxy.

Even though in principle there could be multiple child CAs under the TA
Proxy our design is intended to use a single child CA only. Furthermore,
theoretically, this child CA could access the TA Proxy from a remote
system (another Krill or RPKI CA installation) - but at this time we only
support a local Krill CA for this purpose.

One key advantage of this model is that it allows us to trigger a re-sync
of the local CA child with its TA Proxy parent immediately after the
latter processed the TA Signer response.

Internet Number Resources claimed by TA
---------------------------------------

We include *all* IPv4, IPv6 and AS number resources on the TA certificate,
as well as the one immediate child CA. This child CA is in effect the
acting **online** Trust Anchor in this model. This **online** CA then
acts as a normal parent to any number of other CAs which can be co-hosted,
or remote, as desired.

Note that people may have concerns about a TA that claims all possible
internet number resources, because this way multiple TAs claim the same
resources and it's impossible for an observer (validator) to know which
TA is supposed to be authoritative. Attestations found under each
configured TA are considered equally valid by RPKI validation software.

It would be better in this regard if the TA certificates would only claim
those resources that the operating organisation is actually responsible for,
but the problem with this is that this makes it hard to make changes to
that set of resources. For `this reason <https://www.nro.net/regional-internet-registries-are-preparing-to-deploy-all-resources-rpki-service/>`_
the RIR TA certificates currently all claim all internet number resources,
though in practice they will only sign NIR or member certificates for
resources that are actually allocated.

Changing the TA certificate resources could be supported, but this would
mean that a full exchange between an online system and a possibly
**offline** TA Signer (in case of Krill) is performed whenever those
resources need to change.

A reasonable compromise for this set up could be to include one more CA -
let's call it the **operational** TA - as the only child of the **online**
TA, and use that as the parent CA for other children. This way the
resources can be constrained and identified with relative ease, while
they can still be modified in a timely manner when needed. This would result
in a model similar to how the `RIPE NCC TA Structure <https://www.ripe.net/manage-ips-and-asns/resource-management/rpki/ripe-ncc-rpki-trust-anchor-structure>`_
is defined.

On the other hand, technical arguments can be raised to advocate that a
single global Trust Anchor should be used and that that should be the
parent for the RIR CAs, rather then having five such TAs. From a technical
point of view this model makes a lot of sense and it was initially
`recommended <https://www.iab.org/documents/correspondence-reports-documents/docs2010/iab-statement-on-the-rpki/>`_
by the Internet Architecture Board (IAB). But, the political and
operational realities make this model infeasible causing the IAB to
`reconsider <https://www.iab.org/documents/correspondence-reports-documents/2018-2/iab-statement-on-the-rpki/>`_
their previous statement on this matter.

Note that if you are using Krill as a local TA for testing purposes, or
for private address space management perhaps, you have the option of
limiting the **online** TA resources to a smaller set, but if you do there
is currently no support to change that resource set after initial setup.



Set Up
^^^^^^

At this time you can have only 1 Trust Anchor per Krill instance. We
believe that there is not likely going to be a need for managing multiple
TAs in a single installation.

Krillta Command Line Tool
-------------------------

Krill now includes ``krillta``. This is used to manage both the TA Signer,
in which case it will expect to keep its state on a local disk and the TA
Proxy, in which case it will connect to the Krill server in the same way
that ``krillc`` does.

The communication between the TA Proxy and Signer is done using messages
encoded in simple JSON files. We are planning to wrap these messages in
signed CMS - much like the CMS used in :rfc:`6492` - but this has not
yet been done at this time. But, note that when we do this, this will
not alter the process described below - it will improve security by
ensuring that the Proxy and Signer will only accept properly signed
requests and responses, and in the process we will have a verifiable
audit trail of the interactions.

.. NOTE:: It is still undecided whether we will include ``krillta`` in the
   packages we build. Because this is only needed by a handful of users
   we may end up not including it by default, but instead require that
   users install it using "cargo" instead.


Enable TA Support
-----------------

Add the following to your ``krill.conf`` files:

.. code-block:: text

  ta_support_enabled = true


Initialise TA Proxy
-------------------

The first step in the actual set up of the Krill TA Signer and Proxy
couple is to initialise the TA Proxy. This will create an empty TA Proxy
that has an identity key for communication, and pretty much nothing else.

.. code-block:: bash

  krillta proxy init


Initialise Publication Server
-----------------------------

We recommend that you set up and use a Publication Server in the same
Krill instance that hosts your TA Proxy, and online TA child for that
matter, which we will get to in a bit.

The reason for this is that communication will be more efficient, and
more importantly less error prone. I.e. it's unlikely that the same
Krill instance would work for the TA Proxy but refuse to work for its
Publication Server.

The setup of a Krill Publication Server is described
:ref:`here<doc_krill_publication_server>`.

TA Proxy Publisher Request
--------------------------

Get the TA Proxy :rfc:`8183` Publisher Request XML file and save it
so it can be uploaded tot he Publication Server:

.. code-block:: bash

  krillta proxy repo request > ./pub-req.xml

Add TA Proxy as Publisher
-------------------------

Add the TA Proxy as a publisher and capture the :rfc:`8183` Repository
Response XML:

.. code-block:: bash

  krillc pubserver publishers add --request ./pub-req.xml >./repo-res.xml

.. Note:: The Krill TA uses "ta" as its name (handle in RFC terms).
     Krill Publication Servers normally add the handle name as a sub-dir
     to the global base rsync path (``sia_base`` in RFC terms). However,
     if the handle is "ta", then no sub-dir will be added. The reason is
     that this way recursive rsync fetches for the TA certificate's
     publication point will get the full repository content in one go.

Configure Repository for TA Proxy
---------------------------------

Now add the Publication Server (and its associated Repository) to the
TA Proxy:

.. code-block:: bash

  krillta proxy repo configure --response ./repo-res.xml


Configure the TA Signer
-----------------------

Create a working directory where your TA Signer can keep its state and
log file. Then create a configuration file. If you use ``/etc/krillta.conf``
as the configuration file, then ``krillta`` will be able to find it
automatically, otherwise use ``-c /path/to/krillta.conf`` to override
this default.

The configuration file must at least contain a setting for the data
directory. Other settings are optional - you only need to change them
if you want to change the default logging and/or use an HSM.

.. NOTE:: At this moment "timing" parameters for the TA are hard coded. Child
   CA certificates are signed (and re-signed) with a validity of 52 weeks.
   The CRL and MFT next update and MFT EE certificate not after time are
   set to 12 weeks after the moment of signing. We may add support for
   overriding these values if desired.

Example configuration file:

.. code-block::

  ######################################################################################
  #                                                                                    #
  #                                      DATA                                          #
  #                                                                                    #
  ######################################################################################

  # Specify the directory where the TA Signer will store its data.
  data_dir = "/var/lib/krillta/data"

  ######################################################################################
  #                                                                                    #
  #                                     LOGGING                                        #
  #                                                                                    #
  ######################################################################################

  # Log level
  #
  # The maximum log level ("off", "error", "warn", "info", or "debug") for
  # which to log messages.
  #
  # Defaults to "warn"
  #
  ### log_level = "warn"

  # Log type
  #
  # Where to log to. One of "stderr" for stderr, "syslog" for syslog, or "file"
  # for a file in which case $data_dir/krillta.log will be used. This cannot (yet)
  # be overridden.
  #
  # Defaults to "file"
  #
  ### log_type = "file"

  ######################################################################################
  #                                                                                    #
  #                                SIGNER CONFIGURATION                                #
  #                                                                                    #
  ######################################################################################

  #
  # By default OpenSSL is used for key generation and signing.
  #
  # But.. The usual Krill HSM support should also work in this context. If you want to
  # use an HSM please read the documentation here:
  # https://krill.docs.nlnetlabs.nl/en/stable/hsm.html
  #
  # Note that this configuration cannot be changed after the TA Signer has been
  # initialised. Or rather.. where for normal Krill CAs defaults may be changed and
  # key rolls can be used to start using a different signer, there is no key roll
  # support for the TA. This may be implemented in future in which case we would
  # also support RPKI Signed TALs for this process.


Initialise the TA Signer
------------------------

The TA Signer is always associated with a single TA Proxy. We initialised the
TA Proxy and configured a repository for it in the earlier steps. We now
need to export some of this information so that we can an initialise the
one single TA Signer for that Proxy.

Step 1: Get the proxy ID

.. code-block:: bash

  krillta proxy id --format json > ./proxy-id.json

Step 2: Get the proxy repo contact

.. code-block:: bash

  krillta proxy repo contact --format json  >./proxy-repo.json

Step 3: Initialise

Here you need to use the files saved in steps 1 and 2.

In addition to this you will need to specify the URIs that should be used
on the Trust Anchor Locator (TAL). Of course that TA certificate does not
yet exist - we need to know the URIs so it can be generated properly. You
will be able to download the TA certificate at a later stage. For now,
make sure that you choose URIs (rsync and HTTPS) where you will host a
copy of that certificate later.

Note that the TA certificate can *not* be hosted in the normal rsync and
RRDP repository of your publication server. You can use the same hardware,
web server and rsync daemon, but you will need different endpoints as the
TA certificate itself is not published using the :rfc:`8181` Publication
Protocol.

.. code-block:: bash

  krillta signer init --proxy_id ./proxy-id.json \
                      --proxy_repository_contact ./proxy-repo.json \
                      --tal_https <HTTPS URI for TA cert on TAL>
                      --tal_rsync <RSYNC URI for TA cert on TAL>


Associate the TA Signer with the Proxy
--------------------------------------

Get the TA Signer 'info' JSON file and save it:

.. code-block:: bash

  krillta signer show > ./signer-info.json


Then 'initialise' the signer associated with the TA Proxy. (we should
probably rename this to 'associate' instead):

.. code-block:: bash

  krillta proxy signer init --info ./signer-info.json


At this point you should see that the TA certificate is available in
Krill under the ``/ta/ta.cer`` endpoint. Copy it and place it where
your web server and rsync daemon can serve it. You will most likely
need a dedicated configuration for this in your web server as it's a
different path from the usual RRDP content, and you will need a separate
rsyncd module.

You should also see that a manifest and CRL were published for your
TA. These files should be published in your Publication Server's base
rsync directory. As explained above, the "ta" does not use a sub-dir.


Create Child CA under TA
------------------------

As mentioned in the overview section we recommend creating a single
child CA under the TA, with all resources. This will in effect be the
acting "online" TA.

Step 1: Create the "online" CA

.. code-block:: bash

  krillc add --ca online

Step 2: Add "online" as a child of "ta"

.. code-block:: bash

  krillc show --ca online --format json >./online.json
  krillta proxy children add --info ./online.json

Step 3: Add "ta" as a parent of "online"

.. code-block:: bash

  krillta proxy children response --child online >./res.xml
  krillc parents add --ca online --parent ta --response ./res.xml

Step 3: Add "online" as a Publisher

.. code-block:: bash

  krillc repo request --ca online ./pub-req.xml
  krillc pubserver publishers add --request ./pub-req.xml > ./repo-res.xml
  krillc repo configure --ca online --response ./repo-res.xml

Now there should be a pending CSR from "online" to its parent "ta". It
will keep sending the CSR periodically, but it will will get a not-performed-response
with codes 1104 or 1101, indicating that the CSR is received and is
scheduled for signing. You may see messages to this effect in the log -
this is not alarming.

If you follow the exchange process described below then the TA Signer will
sign the certificate. Since the "online" CA lives in the same Krill
instance as the TA Proxy it will be made aware of this update immediately
and get its signed certificate without further delay.


Typical Proxy Signer Exchange
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The typical exchange between the Proxy and Signer follows these steps:

- Make the request in the Proxy
- Download the Proxy request
- Process the Proxy request
- Save the Signer response
- Upload the Signer response

Make a TA Proxy Request
-----------------------

.. code-block:: bash

  krillta proxy signer make-request


*Note that the ``krillta`` subcommand combination ``proxy signer`` is
used for actions for the ``proxy`` relating to its associated ``signer``.

Download the TA Proxy Request
-----------------------------

.. code-block:: bash

  krillta proxy signer show-request > ./request.json

.. Note:: We may change the format from json to signed CMS in the near future,
probably before this is released.

Process TA Proxy Request
------------------------

.. code-block:: bash

  krillta signer process --request ./request.json

Save the TA Signer Response
---------------------------

.. code-block:: bash

  krillta signer last > ./response.json


Upload the Signer Response
--------------------------

.. code-block:: bash

  krillta proxy signer process-response --response ./response.json


Auditing
^^^^^^^^

You can review the exchanges seen by the TA Signer. The default output
uses JSON and contains a lot of information. The text output is somewhat
friendlier to the human eye:

.. code-block:: bash

  krillta signer exchanges --format text
