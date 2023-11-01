.. _doc_krill_backup:

Backups and Restores
====================

Krill stores all its data under the data directory that you configured
using the :file:`storage_uri`, or :file:`data_dir` directive.

It is recommended to make backups of this directory for disaster recovery.

To recover you can restore this data and start Krill. If you do, make sure
that you use the same Krill version as the version that produced the
backed up data - as data may be modified (or migrated as we often call it)
on upgrade. Furthermore, beware that if you restore old data then your latest changes
may not be not be included.

If you use Krill as a CA under a parent, e.g. your NIR or RIR, and you
do not self-publish, e.g. you publish at your NIR or RIR, and you did
not make any delegations to child CAs of your own, then a data restore
should be mostly safe. It can happen that your latest ROA configs are
not included, so you will need to double check that, but other than that
your Krill instance will re-sync with the parent and repository on restart.

If you delegated certificates to child CAs, then things may become more
complicated if the child resources changed, or the child performed a
key roll-over since your last backup. In that case, the child will be
recover when they re-sync with you as a parent. However, this can take
up to 36 hours (under default config options at the child). So, you may
need to reach out to your child CAs and ask them to perform a manual
sync, either by using the button on the "Parents" tab in the UI, or by
running `krillc bulk refresh`.

If you are using Krill as a publication server, and you needed to restore
old data, then the content of the repository is very likely to have
changed. This means that RPKI validators would become very confused about
regressions in the content (in particular the "serial" attribute in the
RRDP data might regress). To resolve this issue, you will need to perform
an RRDP session reset using the following command:

```
krillc pubserver server session-reset
```

In addition to this, you will find that CAs that publish at your publication
server will not be aware that their latest publication update(s) may be
missing. This can mean that e.g. their latest ROAs are not there, or that
their manifest or CRL is expired (the CA thinks they published an update).
To resolve this, you will need to reach out to all publishing CAs and
ask them to to perform a manual sync, either by using the button on the
"Repository" tab in the UI, or by running `krillc bulk sync`.
