Introduction and Overview
=========================

`ELIXIR <https://www.elixir-europe.org/>`_, the European collaboration for life
science information, is not only funded by the member countries, in the form of
membership fees and in-kind contributions, but also with a H2020 grant from the
European Commission. The grant, called ELIXIR-Excelerate, was awarded in 2012
and is a 4-year project to build up ELIXIR as an infrastructure.

Background
~~~~~~~~~~

ELIXIR is a distributed organization with a hub in Hinxton, England. A large
part of the ELIXIR activities are shaped around 5 different platforms (`Compute
<https://www.elixir-europe.org/platforms/compute>`_, `Interoperability
<https://elixir-europe.org/platforms/interoperability>`_, `Tools
<https://elixir-europe.org/platforms/tools>`_, `Data
<https://elixir-europe.org/platforms/data>`_ and `Training
<https://elixir-europe.org/platforms/training>`_). Each platform develops and
provides services for the life science community. In order to provide
useful services, ELIXIR also has `user communities
<https://elixir-europe.org/communities>`_, which are focused on scientific
areas. In the Excelerate grant, there are four user communities (or use cases,
as they are called in the grant application): marine metagenomics, plant
genotypes, human data and rare disease.

The Compute Platform is divided into working groups responsible for different
subtasks.  In this document, we try to give an overview on how we have been
working on the data transfer task in the ELIXIR Compute
Platform.

Collecting requirements and making plans
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

At the start of the Excelerate project, we started planning the compute
platform by communicating with the use cases about their requirements
for technical services. This information was then compiled and
reorganized into a document, the `ELIXIR Technical Architecture
<http://drive.google.com/file/d/0B0KXZdVao0kqUE9BbXVrc3ZLY1E/view>`_, which is
revised annually. In this document, we have collated and translated common
elements for IT services into so-called technical use cases (TUCs).

Upon reviewing the requirements, and taking the level of funding into
consideration, it became apparent that we could not spend any resources
developing new tools or technologies. Instead, our strategy in the
compute platform is to identify, integrate and deploy best-of-breed
technologies that can fulfill the requirements from the communities.
This is not a bad thing, as this forces us to utilize the efforts from
e-infrastructures, from other scientific disciplines, and from the
commercial sector.

Sub task: Data Storage and Transfers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the sub task "Data Storage and Transfers" we have been asked to develop
services for four TUCs: data transfers, network storage, data distribution, and
a data/PID registry.  Here, we focus on data transfers, which in spite of being
an `old problem <https://xkcd.com/949/>`_, often is a stumble block for large
scale data analysis.

Bulk transfers
^^^^^^^^^^^^^^

When it comes to data transfers, we really mean bulk
transfers of large data volumes. Smaller transfers are really not a
problem, as it can be achieved with common software and protocols thanks
to mature tools, and the fantastic academic networks, coordinated by
`GÉANT <https://www.geant.org/>`_, that we have access to. However, for
transfers on the terra- and peta-byte scale, performance make a substantial
difference in transfer time, and we need to use the best tools available.

Second, due to the heterogeneous and complex landscape of data
producers, providers, centers, processing facilities and scientists, we
need to be technology and protocol agnostic, and also -- in order to
have any chance of uptake -- propose non-invasive solutions that are
reasonably easy to deploy and maintain. We also aim to provide a service
that is as simple as possible (but not simpler); i.e. we aim to provide
a service that can move bits from one place to another reliably, and
with high performance - preferrably using free software.

When it comes to large scale transfers over long distances, http/(s)ftp are
`not viable options
<https://fasterdata.es.net/data-transfer-tools/say-no-to-scp/>`_. While there
are multiple commercial providers
and proprietary tools (e.g. Aspera, Globus Transfer, ...) the *de facto*
standard for bulk transfers in academia is `gridftp
<https://en.wikipedia.org/wiki/GridFTP>`_, which is being used extensibly at
CERN, PRACE, XSEDE, NERSC, etc. The protocol is an
extension of ftp that have some useful features:

-  parallel transfers
-  third-party transfers
-  checksums
-  resumable transfers
-  encryption
-  sync

In 2017, `Globus <https://www.globus.org/>`_ announced that they will no longer
maintain the open source `Globus Toolkit <http://toolkit.globus.org/>`_, which
include server and client software for gridftp-based transfers. Fortunately,
there is now a community effort in place, the `Grid community Forum
<https://gridcf.org/>`_, that will keep the software maintained for the time
being.

Getting high performance using free software used to be something of a
black art, but with the advent of self-tuning tcp parameters in the
Linux network stack, good results can be obtained without virtually any
manual work. In fact, with a high-bandwidth connection, disk I/O often
end up as the bottleneck in the transfer (which of course can be
optimized further by using solid state disks and/or striping on the end
points, but that is a whole other story).

Managed transfers
^^^^^^^^^^^^^^^^^

When working with large scale transfers, it is advantageous to have a service
that can carry out the transfer, confirm integrity using checksums, and keep
logs of the activities. This is useful not only because you can have a central
place to monitor the transfer jobs, but also because it is then possible to
submit a transfer job from your laptop, shut it down and return later.
Furthermore, a dedicated transfer service can at least partly move the task of
optimizing performance from end users to IT staff with expert knowledge.

To this end, we have deployed a pilot instance of `File Transfer Service 3
<http://fts.web.cern.ch/>`_ (fts3), which is developed at CERN, where it is
being used to distribute large volumes of experimental data world-wide.

fts3 is `REST <https://en.wikipedia.org/wiki/Representational_state_transfer>`_
service with support for multiple protocols (e.g. https, gridftp, S3), and no
extra configuration is needed at the storage endpoints. Third-party transfers
are used when possible, otherwise they are relayed by the fts3 server. The fts3
software is well designed, and the code is clean and readable. The architecture
also allows for horizontal scaling, so that more instances can be deployed if
needed.

On top of fts3, we have deployed a pilot `web service
<https://fts3.du2.cesnet.cz/>`_ that provides an easy-to-use interface for
submitting and monitoring the transfers.

AAI integration
^^^^^^^^^^^^^^^

Services that read from and write to storage systems most often need to
have a system of authentication and authorization. In ELIXIR we have a
well-functioning AAI system where users can obtain an ELIXIR identity
that can be connected to identity providers such as `eduGAIN
<https://edugain.org/>`_ or google id. Service providers can then use several
different methods to authenticate users, e.g. SAML and OAuth2.

When it comes to the transfer services that we are testing, the AAI
layer is based on `Public Key Infrastructure
<https://en.wikipedia.org/wiki/Public_key_infrastructure>`_ (PKI). This is a
mature and battle tested technology. However, obtaining and handling
certificates has also proven to be a large hurdle for users -- in particular
the life science community, which is less accustomed to the sometimes archaic
command line utilities involved.

Our approach is to rely on the PKI technology, but to avoid exposing
users to certificates. For this to work, we rely on a credential
translation service (CTS) called `rcauth <http://rcauth.eu/>`_, which is part
of the `AARC project <https://aarc-project.eu/>`_. The CTS allows users to
obtain so-called proxy certificates by authenticating to a web portal. These
proxy certificates, which can be thought of as short-lived tickets, can then be
used when communicating with transfer and storage service.

Current status and the road ahead
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We now have all the building blocks needed for carrying out large scale data
transfers. Future work in this area will to some extent also take place in the
`European Open Science Cloud <https://www.eosc-hub.eu/>`_, where there is a
life science project called `EOSC-Life
<https://elixir-europe.org/news/eosc-life-start>`_.

