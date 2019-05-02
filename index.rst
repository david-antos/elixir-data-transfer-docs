.. ELIXIR Data Transfer documentation master file, created by
   sphinx-quickstart on Mon May 28 20:20:21 2018.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to ELIXIR Data Transfer's documentation!
================================================

`ELIXIR <https://www.elixir-europe.org/>`_ is a distributed infrastructure for
life-science information. As part of the H2020 project `ELIXIR-Excelerate
<https://www.elixir-europe.org/about-us/how-funded/eu-projects/excelerate>`_ 
to accelerate the implementation of ELIXIR, the `ELIXIR Compute Platform
<https://www.elixir-europe.org/platforms/compute>`_ task *4.3.3 Data Storage and
Transfers* has investigated, tested and piloted different methods to move
data between sites, compute facilities and users' local environments. These
pages contain findings, recommendations and instructions for different
scenarios related to data transfers.

.. image:: images/elixir.png
   :width: 25%
.. image:: images/excelerate-logo.png
   :width: 25%

.. toctree::
   :maxdepth: 1
   :caption: Introduction

   overview

Use case-based documentation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. topic:: I want to...

   - deploy a storage endpoint

     - :doc:`gridftp-installation/gridftp-installation`
     - :doc:`gridftp-with-ansible/gridftp-servers-and-clients-ansible`

   - move data

     - ... to a cloud instance (user to cloud)

       - `Docker image for cloud data ingestion <https://github.com/nmb/ready-to-go>`_

     - ... between workstations (user to user)

       - `FilePizza <https://file.pizza/>`_ (WebRTC in browser)
       - `ShareDrop <https://www.sharedrop.io/>`_ (WebRTC in browser)
       - `Magic wormhole <https://magic-wormhole.readthedocs.io/>`_ (command line application)
       - `FileSender <https://filesender.org/>`_ (web service)

     - ... from a central repository to a local resource, and keep the mirror up to date

       - `Reference Data Set Distribution Service <https://docs.google.com/document/d/1DR4YcKVb0HTq-V6r-5lfEhwz8VPRoPl44uAOL4Fzq7Y/edit#heading=h.gbf4xn8cmekk>`_
       - `Dat - a protocol for sharing data between computers <https://docs.datproject.org/docs/intro>`_


.. toctree::
   :hidden:
   :maxdepth: 1
   :caption: GridFTP:

   gridftp-installation/gridftp-installation
   gridftp-with-ansible/gridftp-servers-and-clients-ansible
   gridftp-CIlogon-proxy/gridftp-transfer-using-CIlogon-proxy-cert

.. toctree::
   :hidden:
   :maxdepth: 1
   :caption: FTS3:

   fts3-tests/testing-fts3-for-transfers
   Cloud demonstrator <https://github.com/NBISweden/excelerate-demonstrator-4.3>

.. toctree::
   :hidden:
   :maxdepth: 1
   :caption: AAI aspects and other peculiarities

   gridftp-aai-integration/aai-integration-mapping-users
   prerequisities-for-RCAuth.eu-certificates/prerequisities-for-RCAuth.eu-certificates

.. toctree::
   :hidden:
   :maxdepth: 1
   :caption: Documentation on Documentation:

   documentation-notes/documentation-notes

.. Indices and tables
   ==================

.. * :ref:`genindex`
   * :ref:`modindex`
   * :ref:`search`
