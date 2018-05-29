GridFTP installation instructions for ELIXIR use
================================================

ELIXIR Excelerate WP4

*Harri Salminen/CSC, 2016*

This document covers an example installation of a standard globus
GridFTP server with ELIXIR Demo CA on a Scientific Linux 7 server. For
official and more comprehensive generic installation instructions please
see the `*gtadmin
manual* <http://toolkit.globus.org/toolkit/docs/6.0/admin/install/>`__

Software installation
---------------------

First you need to have or install the Scientific Linux 7 server or
similar (Centos/RHEL). This example uses SL7.2 set up as an
infrastructure server. If you have a debian based distribution like
Ubuntu the principles are same but commands differ (yum -> apt-get,
firewall-cmd -> ufw or whatever fw you use etc.). It’s assumed that you
have a running server with standard unix development and management
tools with which you are familiar with.

To start the server installation please go to
`*http://toolkit.globus.org/toolkit/* <http://toolkit.globus.org/toolkit/>`__
and check for the

latest stable release. In this case it was GT6.0 and it’s download page
listed repositories for various operating systems (Centos/RHEL/SL,
Ubuntu, SuSE,Mac OS X,Windows etc.).

Install the chosen repo with

wget
http://toolkit.globus.org/ftppub/gt6/installers/repo/globus-toolkit-repo-latest.noarch.rpm

rpm -i globus-toolkit-repo-latest.noarch.rpm

Then install Globus packages e.g.:

yum install globus-gridftp globus-gsi globus-data-management-server

You may also want a client tools for in the server or your client host
to test the server with

yum install globus-data-management-client

Firewall configuration
----------------------

Open ports for GridFTP control and data connections

firewall-cmd --add-port=50000-51000/tcp

firewall-cmd --add-port=2811/tcp

Check results:

firewall-cmd --zone=public --list-all

public (default, active)

interfaces: eth0

sources:

services: dhcpv6-client ssh

ports: 2811/tcp 50000-51000/tcp

masquerade: no

forward-ports:

icmp-blocks:

rich rules:

If your public interface is eth1 instead of eth0 change it with

firewall-cmd --zone=public --change-interface=eth1

You’ll also need to configure the port range to the environment of the
GridFTP server in its startup script. (e.g.
/etc/init.d/globus-gridftp-server)

export GLOBUS\_TCP\_PORT\_RANGE=50000,51000

or if you use xinetd in /etc/xinet.d/gridftp

env += GLOBUS\_TCP\_PORT\_RANGE=50000,51000

If you also need to restrict source ports there’s variable called

GLOBUS\_TCP\_SOURCE\_RANGE for that purpose.

Access Control
--------------

For secure transfers GridFTP uses the Grid Security Infrastructure. It
requires that the client has a valid X.509 certificate which is signed
by a Certificate Authority (CA). Both sides must be able to validate the
certificate via a chain of trust. Who you trust, determines with whom
you can communicate. In this example we trust both the `*Interoperable
Global Trust Federation (IGTF)* <https://www.igtf.net/>`__ used for
building distributed research infrastructures and an ELIXIR demo CA used
for demonstrating the possibilities of the ELIXIR AAI pilot.

The server does also support password and ssh authentication which are
beyond the scope of this document but of course documented in the
manuals if you need them. It even has anonymous mode which is not
recommended. If you need to allow anonymous public access, I’d recommend
you to look at pureftpd and rsync instead.

After verifying the client certificate the server uses the DN in the
certificate to map the user to a local user using a grid map file. What
that entitles the user to do, is up to the configuration of each
particular installation.

Host certificate 
~~~~~~~~~~~~~~~~~

Every server needs its own server certificate and key which should be
signed by a Certificate Authority accepted by the clients. How you get
certificates depends on your organization. Many european sites can get
the certificate service via their National Research Network. Like FUNET,
most of them, but not all, are partners in the `*Terena Certificate
Service* <http://www.geant.org/Services/Trust_identity_and_security/Pages/TCS.aspx>`__.

If your organization is not a member in that, check you can get IGTF
recognized certificates via some other trusted source. If not, you may
have to acquire it from some other CA or act as your own CA. The
standard installation does create by default a globus-simple-ca and
associated certificates which are mainly intended for testing and
development. In that case you’ll have to convince all your clients to
install and trust your CA certificates as well which may work in a
limited internal setup you can control but not very well globally
between organizations.

In all cases the procedure is similar. First you create a certificate
request for your server with openssl preferably on the same server. The
-subj parameter is filled according to your local instructions and
identifies your server globally.

openssl req -newkey rsa:4096 -sha256 -nodes -subj
"/C=FI/ST=Uusimaa/L=Espoo/O=CSC/CN=gridftp.bio.nic.funet.fi" -out
gridftp.bio.nic.funet.fi.req -keyout gridftp.bio.nic.funet.fi.key

Then you send the resulting .req file to your local certificate
provider. E.g. an authorized person in your organization that can verify
you and issue the certificate while you wait.

After you get your public host certificate, you should place it in a
file called /etc/grid-security/hostcert.pem

The private .key file should be named as /etc/grid-security/hostkey.pem
which should be protected from access by others than root. (0600)

You should of course have the domain name in the request configured in
the DNS. Bear in mind that reverse DNS record (PTR) is checked by some
applications (e.g. uberftp) but not all (e.g. globus-url-copy) so try to
keep it synced with the certificate or some applications may not work.
If you plan to use a CNAME for the server you could include the name
mentioned in the reverse record as an alternative name.

Subject: DC=org, DC=terena, DC=tcs, C=FI, ST=Uusimaa, L=Espoo, O=CSC -
Tieteen tietotekniikan keskus Oy, CN=gridftp.bio.nic.funet.fi

...

X509v3 Subject Alternative Name:

DNS:gridftp.bio.nic.funet.fi,DNS:valine.nic.funet.fi

This approach works only if you have authority to request certificates
for all the domains in question from the CA. Usually server certificates
are issued for a period of 1-3 years and you should be the owner of the
domains for that period. For servers with a short lifetime you may need
to figure out a solution by getting your own subdomain (with reverse
DNS) and apply for a wildcard certficate e.g.
\*.vm.yourproject.somewhere.net.

IGTF CA certificate list installation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you use common eScience certificates from terena or other IGTF
compatible certificates Grid Community usually uses you should add trust
anchors which means CA certificate and signing policy files usually kept
in the directory /etc/grid-security/certificates. The easiest way is to
install them from the IGTF repository so they will be automatically kept
up to date. You should also install the Certificate Revocation List
update scripts in cron to be able to revoke certificates that may have
been compromised or for some other reason. Please note that there’s
multiple different IGTF repo versions. Please pick the comprehensive one
detailed under heading Installation which should also contain the CA for
eScience Personal Certificates you probably may wish to use. For more
information please go to:

`*https://wiki.egi.eu/wiki/EGI\_IGTF\_Release* <https://wiki.egi.eu/wiki/EGI_IGTF_Release>`__

ELIXIR CILogon integration
~~~~~~~~~~~~~~~~~~~~~~~~~~

The `*ELIXIR AAI
integration* <https://docs.google.com/document/d/1ihb0hH2YJqSCPZS0syVpvAOeQP1HTxdf_XMsZZLe_W0/edit>`__
is still in R&D phase and is the process of being included in the
official IGTF repository. To enable it at the moment you should do the
following:

cd /etc/grid-security/certificates

wget --no-check-certificate -O
/etc/grid-security/certificates/rcauth.eu.pem
`*http://rcauth.eu/pilot/g1/ca/cacert.pem* <http://rcauth.eu/pilot/g1/ca/cacert.pem>`__

wget --no-check-certificate -O
/etc/grid-security/certificates/rcauth.eu.signing\_policy

`*https://rcauth.eu/pilot/rcauth-pilot-ica-g1.signing\_polic* <https://rcauth.eu/pilot/rcauth-pilot-ica-g1.signing_policy>`__\ y

export HASH=\`openssl x509 -in
/etc/grid-security/certificates/rcauth.eu.pem -noout -hash\`

ln -s rcauth.eu.pem $HASH.0

ln -s rcauth.eu.signing\_policy $HASH.signing\_policy

wget --no-check-certificate -O
/etc/grid-security/certificates/dcaroot.pem
`*https://ca.dutchgrid.nl/dcaroot/g1/ca/cacert.pem* <https://ca.dutchgrid.nl/dcaroot/g1/ca/cacert.pem>`__

wget --no-check-certificate -O
/etc/grid-security/certificates/dcaroot.signing\_policy
`*https://ca.dutchgrid.nl/dcaroot/g1/dca-root-g1.signing\_policy* <https://ca.dutchgrid.nl/dcaroot/g1/dca-root-g1.signing_policy>`__

export HASH=\`openssl x509 -in
/etc/grid-security/certificates/dcaroot.pem -noout -hash\`

ln -s dcaroot.pem $HASH.0

ln -s dcaroot.signing\_policy $HASH.signing\_policy

Obtaining ELIXIR CILogon VO proxy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To test and demonstrate the ELIXIR AAI infrastructure there’s a portal
via one can get a temporary proxy certificate that validates against the
rcauth.eu CA. First you need to register yourself with your name to the
`*elixir intranet* <https://www.elixir-europe.org/intranet>`__ to get an
ELIXIR id. There’s also a growing number of alternative authenticators
available on the `*vo
proxy* <https://elixir-cilogon-mp.grid.cesnet.cz/vo-portal/>`__ login
page.

After that you can go to the demo portal, press the vo-proxy button and
you should end up in

a page that has the temporary proxy certificate and key behind a
show/hide link.

Please cut/paste a copy of the encoded certificates and private key to
your client machine in a file /tmp/x509up\_uUID with permissions 0600.
UID would be the user under which you are going to run the GridFTP
clients.

For real use this step is planned to be automated under the hood in one
way or other during the course of the ELIXIR EXCELERATE WP4. This is
just a proof of concept.

Adding users to Gridmap file
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Also take note of the identity line in your certificate which you’ll
need to put in the gridmap file at the server to map the identity to a
local user (here a fictitious Joe User).

grid-mapfile-add-entry -dn
‘/DC=eu/DC=rcauth/DC=rcauth-clients/O=elixir-europe.org/CN=Joe Use
ABCDEFG1234567’ -ln test

For eScience certificates there’s usually at least C, O and CN
attributes and rest vary locally.

After the gridmap procedure the holders of the certificate mentioned in
the file have the same file access rights as the local user it’s mapped
to including usually the root directory.

Limiting access
~~~~~~~~~~~~~~~

If you want to further limit access to certain directories you’ll need
additional gridftp configuration options which can be placed in
/etc/gridftp.conf. If you wish, more complex configurations may be split
to files without type (a dot in a filename) to the /etc/gridftp.d/
directory which you must create first yourself.

e.g. to restrict access to only certain directory trees use a setting
like

“restrict\_paths /pub,/home”

If you wish to limit users to their home directories set
“use\_home\_dirs 1”

Chrooting the gridftp server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you wish you can optionally chroot the whole gridftp server to
particular directory tree

as follows:

mkdir /mnt/gridftp

cd /mnt/gridftp/

globus-gridftp-server-setup-chroot -r /mnt/gridftp/

and add in the gridftp config “chroot\_path /mnt/gridftp”

After that you should have /mnt/gridftp/etc/grid-security/certificates
and other necessary files and directories copied under /mnt/gridftp. You
can the mount whatever data directories you need. And you can use
restrict paths to hide the /etc, /dev and /tmp directories if you wish.
Note that /etc/passwd contains paths for users home directories that are
now relative to the chroot root directory.

NOTE: the server still runs under user root by default even though it
changes to the user mentioned in the gridmap file. If you want to avoid
that and prefer a non-root public server, you can set up a split
configuration where the frontend is running under some other userid and
the data transfer backend nodes as root. See the admin manual for advice
on how to set up split and striped configurations. You’ll need to create
a new user id with suitable environment, write custom startup scripts,
copy keys, assign backend ports etc. so the default configuration for a
single server setup described here is not enough.

Testing the server
------------------

After you have set up the server it’s time to test it.

First try to start it either as a standalone daemon

service globus-gridftp-server start

If it fails, start debugging and reading the manual until it succeeds.
You may want to set a

config parameter “debug 1” after which the server doesn’t disconnect but
stays at foreground in debug mode until one request has been served.

The default installation includes a xinetd configuration file in
/etc/xinet.d/gridftp which you

can enable by changing the disable parameter in it to no and reloading
xinetd config. However in that mode you can’t debug it in foreground
mode.

Third option which is also suited for debugging is to run the server
directly from command line. For command line options try
globus-gridftp-server -h

If you have the certificates for the CA set up in your local workstation
and grid proxy initialized either by grid-proxy-init (the normal way) or
by copying the above mentioned proxy

to /tmp directory (the demo way) you should be ready to start.

For testing the simple globus-url-copy tool is used here

Syntax: globus-url-copy [-help] [-vb] [-dbg] [-r] [-rst] [-s <subject>]

[-p <parallelism>] [-tcp-bs <size>] [-bs <size>]

-f <filename> \| <sourceURL> <destURL>

If something fails, add -dbg flag to see where it fails.

List directories
^^^^^^^^^^^^^^^^

globus-url-copy -list gsiftp://gridftp.bio.nic.funet.fi/

gsiftp://gridftp.bio.nic.funet.fi/

home/

pub/

globus-url-copy -list
gsiftp://gridftp.bio.nic.funet.fi/pub/mirrors/ftp.ebi.ac.uk/pub/databases/ensembl/mysql/83/xiphophorus\_maculatus\_rnaseq\_83\_1/

gsiftp://gridftp.bio.nic.funet.fi/pub/mirrors/ftp.ebi.ac.uk/pub/databases/ensembl/mysql/83/xiphophorus\_maculatus\_rnaseq\_83\_1/

alt\_allele.MYD

alt\_allele.MYI

…

Copying files
^^^^^^^^^^^^^

globus-url-copy
gsiftp://gridftp.bio.nic.funet.fi/home/test/RandomMegabyte.bin
Random.bin

mkdir /tmp/test; globus-url-copy -vb
gsiftp://gridftp.bio.nic.funet.fi/pub/mirrors/ftp.ebi.ac.uk/pub/databases/ensembl/mysql/83/xiphophorus\_maculatus\_rnaseq\_83\_1/
/tmp/test/

with -vb flag you can get some performance statistics. With -tcp-bs you
can try to increase your TCP buffers if they don’t scale enough
automatically within few seconds.

-p option specifies how many parallel data connections should be used
which may help when window scaling or tcp tuning isn’t a solution.

The -p option seems to automatically use active mode FTP which means
that the server tries to open the data connections to the client and not
vice versa. For that to succeed the destination should have it’s
firewall opened to a range of ports for incoming TCP-connections. So you
may need to set the GLOBUS\_TCP\_PORT\_RANGE=start,end also in the
client and open all firewalls for the range. Finally there’s a -cc
switch which means that you can specify how many parallel ftp clients
for transferring different files you may want to use. Excessive amounts
are not a good idea since you might block out others, try just a few to
start and tune your TCP first if possible.

There’s also a large number of other parameters that are documented in
the man page.

If your transfer speed is not fast enough you should check if it’s
limited by source or destination server and its associated storage or
the network in between.

If you are experiencing problems, you could debug the by adding the -dbg
flag to your clobus-url-copy command

globus-url-copy -list -dbg gsiftp://gridftp.bio.nic.funet.fi/

The are also 3rd party transfer tools such as UberfFTP and
`*gtransfer* <https://github.com/fr4nk5ch31n3r/gtransfer>`__ with
multiple functions and a debug option. You should note that UberFTP is
significantly slower than globus-url-copy with smaller files since it
doesn’t seem to reuse the data connections.

uberftp -debug 3 globus.du3.cesnet.cz

If you can’t get the authetication to success, you can check for
possible certificate issues with commands

grid-cert-diagnostics -g globus.du3.cesnet.cz

grid-proxy-init -verify -debug

Many problems seem to stem from the fact that the certificate your
client is not signed by a CA that is in the
/etc/grid-security/certificates on both ends. Other common issues are
missing or expired CRLs or certificates.

Network performance analysis
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For checking the network connection I recommend in addition to the basic
ping and traceroute tools the iperf performance testing tool against
some suitable iperf servers. The command line iperf too can act either
as server or client so it would relatively easy to set up servers at
each site so that one could measure, analyze and tune network issues
independent of storage and gridftp server issues. You can get it along
with documentation from `*https://iperf.fr/* <https://iperf.fr/>`__
Version 3 is recommended, it’s easier to use and has more features. Use
version 2 only if there’s no version 3 available. They are not
compatible and use different default ports.

A basic iperf v3 server is started simply with command iperf3 -s. You
only need to open port 5201 for TCP and UDP both in IPv4 and IPv6 (if
you use it). Funet has a dedicated iperf server with a 10Gbit/s link
called iperf.funet.fi. Also iperf-delay50 and iperf-delay150.funet.fi
are available with simulated extra 50ms (trans european) or 150ms
(transatlantic) delay.

Following is an example of a basic performance test between the `*Cray
XC40 supercomputer called
sisu* <https://research.csc.fi/sisu-supercomputer>`__ in the CSC Kajaani
data center which is 7,7 ms away from the FUNET iperf server in Espoo.
Link speed is 10Gbit/s shared with other users as is usually the case.
You may want to repeat tests at different times of day or different
days.

hks@sisu-login3:~> ping iperf.funet.fi

PING iperf.funet.fi (193.166.255.193) 56(84) bytes of data.

64 bytes from iperf.funet.fi (193.166.255.193): icmp\_seq=1 ttl=60
time=7.73 ms

64 bytes from iperf.funet.fi (193.166.255.193): icmp\_seq=2 ttl=60
time=7.74 ms

hks@sisu-login3:~> traceroute iperf.funet.fi

traceroute to iperf.funet.fi (193.166.255.193), 30 hops max, 40 byte
packets using UDP

1 compnet-gw2.csc.fi (86.50.166.3) 0.407 ms 0.325 ms 0.299 ms

2 rr2-lsc2.csc.fi (86.50.160.14) 7.943 ms 7.808 ms 7.806 ms

3 lsc2-lsc1.csc.fi (86.50.160.4) 7.898 ms 7.828 ms 7.742 ms

4 lsc1-csc6.csc.fi (86.50.160.0) 7.690 ms 7.660 ms 7.630 ms

5 iperf.funet.fi (193.166.255.193)(N!) 7.692 ms (N!) 7.745 ms (N!) 7.663
ms

| hks@sisu-login3:~> iperf3 -c iperf.funet.fi
| Connecting to host iperf.funet.fi, port 5201
| [ 4] local 86.50.166.23 port 51444 connected to 193.166.255.193 port
  5201
| [ ID] Interval Transfer Bandwidth Retr Cwnd
| [ 4] 0.00-1.00 sec 12.9 MBytes 108 Mbits/sec 0 242 KBytes
| [ 4] 1.00-2.00 sec 96.2 MBytes 807 Mbits/sec 0 1.80 MBytes
| [ 4] 2.00-3.00 sec 381 MBytes 3.20 Gbits/sec 0 3.82 MBytes
| [ 4] 3.00-4.00 sec 414 MBytes 3.47 Gbits/sec 0 3.87 MBytes
| [ 4] 4.00-5.00 sec 421 MBytes 3.53 Gbits/sec 0 3.87 MBytes
| [ 4] 5.00-6.00 sec 420 MBytes 3.52 Gbits/sec 0 3.89 MBytes
| [ 4] 6.00-7.00 sec 421 MBytes 3.53 Gbits/sec 0 3.90 MBytes
| [ 4] 7.00-8.00 sec 425 MBytes 3.57 Gbits/sec 0 3.90 MBytes
| [ 4] 8.00-9.00 sec 424 MBytes 3.55 Gbits/sec 0 3.90 MBytes
| [ 4] 9.00-10.00 sec 421 MBytes 3.53 Gbits/sec 0 3.91 MBytes
| - - - - - - - - - - - - - - - - - - - - - - - - -
| [ ID] Interval Transfer Bandwidth Retr
| [ 4] 0.00-10.00 sec 3.36 GBytes 2.88 Gbits/sec 0 sender
| [ 4] 0.00-10.00 sec 3.36 GBytes 2.88 Gbits/sec receiver
| iperf Done.

Note the scaling up of the TCP windows and buffers in the first few
seconds. Scaling should be on by default, you can check it on linux with

cat /proc/sys/net/ipv4/tcp\_window\_scaling

1

You can also check the TCP buffer settings and possibly tune them within
reasonable limits. Too large maximum buffers may cause problems with
your other connections. In this case the sisu supercomputer has
reasonable write max of 4MB and receive max of 6MB

cat /proc/sys/net/ipv4/tcp\_wmem

4096 16384 4194304

cat /proc/sys/net/ipv4/tcp\_rmem

4096 87380 6291456

If you are familiar with the `*TCP
protocol* <https://en.wikipedia.org/wiki/Transmission_Control_Protocol>`__
and its extensions then you could also use some suitable network
analyzer like tcpdump or wireshark to look deeper in the problem at the
protocol and packet level too.

Storage performance analysis
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If network seems to be much faster that your transfers, then the problem
might be in your storage infrastructure. The simple way to check is to
transfer to/from /dev/null or ramdisk like /dev/shm and compare it to
the results from real storage. To test the storage performance you could
first simply use a large enough file and copy it to /dev/null or ramdisk
with for example dd which may give an idea of sequential transfer
speeds. Please note that there may be a huge difference in performance
depending if the file is already in the memory buffers of the server,
disk cache (ram/ssd) or only in real spinning disks possibly used by
others too. To rule that out, in general you should use transfers that
are several times bigger than the largest caches along the way which may
be hard to achieve in some environments.

Also the access patterns, record sizes, number of parallel threads, disk
type etc. affects the performance so for more specialized testing tools
designed for the purpose should be used.

One such tool for unix/linux is the open source
`*iozone* <http://www.iozone.org/>`__.

GridFTP server tuning
~~~~~~~~~~~~~~~~~~~~~

The transfers do use server resources like memory, CPU and I/O
especially at high speeds over long latencies. So you may want to limit
the number of concurrent GridFTP connections.

A rule of thumb is that you might need 16MB of memory for network
buffers in both directions of a long delay connection + 2MB for the
server which would set the maximum number of connections available
memory/34. You may need to change the kernel limits too.

To set the limit use the max\_connections parameter in the config file
if you are running the server as an independent daemon as in this
example. However, if you are using xinetd, to start the server you can
set the variable instance = <max instances> in the xinetd config.

Another option for tuning is to separate the frontend and backend
processes so that you can have multiple backends for the actual transfer
and only a lightweight nonroot frontend process open to the internet.
This may give a performance boost especially in clusters that can use
multiple different I/O nodes in a striped configuration as well as
increase security.

For more information please refer to the `*Gridftp admin
guide* <http://toolkit.globus.org/toolkit/docs/latest-stable/gridftp/gridftp.pdf>`__.
