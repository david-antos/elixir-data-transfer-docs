==============================
AAI Integration: mapping users
==============================

*Mikael Borg*

Dec 2017

Background
==========

Users of ELIXIR gridftp endpoints are authenticated using X509
certificates. The certificate user identities (subjects) must be mapped
to local accounts. This is done in the configuration file
/etc/grid-security/grid-mapfile.

However, the ELIXIR AAI certificate subjects sometimes change, which
means that the mapping must be kept up to date. This document describes
how to set up automatic user mapping.

In the following it is assumed that a gridftp endpoint already is
deployed.

Set up sync with PERUN
======================

Install package edg-mkgridmap (present in e.g. EPEL repository):

$ yum install -y edg-mkgridmap

Configure edg-mkgridmap with /etc/edg-mkgridmap.conf::

group "vomss://voms1.grid.cesnet.cz:8443/voms/vo.elixir-europe.org/"
AUTO

gmf\_local /etc/localgridmap.conf

The first line tells the script to obtain a list of certificate subjects
from ELIXIR PERUN. The second line configures where to store local user
mappings that should be present in the grid-mapfile (e.g. if you have
some other user mapping based on e.g. grid certificates).

Configure user mapping
======================

The AUTO keyword tells edg-mkgridmap to execute the local script
/usr/libexec/edg-mkgridmap/local-subject2user when mapping certificates
to local user accounts. The script is called with each user certificate
subject as argument and is expected to write the local username
associated with the user certificate subject to STDOUT.

Here is a sample script that will map a couple of ELIXIR identities to
local user account ‘heartbeat’, and one ELIXIR identity to local user
‘borg’:

#!/bin/bash

# map the following to the heartbeat account:

heartbeaters="Delisa Simonovic\|Amelie Cornelis\|Jinny Chien"

if [[ $1 =~ $heartbeaters ]]

then

echo "heartbeat"

fi

# local user

if [[ $1 =~ 'Mikael Borg' ]]

then

echo "borg"

fi

Note that the script need to be executable.

Testing
=======

It is possible to test the set-up by running edg-mkgridmap without
arguments. The resulting grid-mapfile will then be written to STDOUT.

Note that access to PERUN requires that the server making the connection
has a proper host certificate - letsencrypt certificates are not
accepted. For testing purposes, it is possible to use an ELIXIR proxy
certificate though:

-  Obtain ELIXIR proxy certificate from
       `*CILogon* <https://elixir-cilogon-mp.grid.cesnet.cz/vo-portal/startRequest>`__
       and save to a file, e.g. cert.txt

-  Run edg-mkgridmap in user mode with environment variable
       X509\_USER\_PROXY pointing to your proxy certificate, e.g.:

   -  X509\_USER\_PROXY=$HOME/cert.txt edg-mkgridmap --usermode

Keep mapping updated via cronjob
================================

FInally, in order to keep the user mapping up to date, run edg-mkgridmap
as a cronjob, e.g. add file /etc/cron.d/edg-mkgridmap.cron with content
(as one line):

17 \*/2 \* \* \* /usr/sbin/edg-mkgridmap --conf=/etc/edg-mkgridmap.conf
--output=/etc/grid-security/grid-mapfile --safe --cache --quiet

Acknowledgements
================

Thanks to Michal Procházka for providing necessary information.
