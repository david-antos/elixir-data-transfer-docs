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
<https://www.elixir-europe.org/platforms/compute>`_ task 4.3.3 Data Storage and
Transfers has investigated, tested and piloted different methods to move
information between sites. These pages contain findings, recommendations and
instructions for different scenarios related to data transfers.

.. image:: images/elixir.png
   :width: 25%
.. image:: images/excelerate-logo.png
   :width: 25%

.. toctree::
   :maxdepth: 1
   :caption: GridFTP:

   gridftp-installation/gridftp-installation
   gridftp-with-ansible/gridftp-servers-and-clients-ansible
   gridftp-CIlogon-proxy/gridftp-transfer-using-CIlogon-proxy-cert

.. toctree::
   :maxdepth: 1
   :caption: FTS3:

   fts3-tests/testing-fts3-for-transfers

.. toctree::
   :maxdepth: 1
   :caption: AAI aspects and other peculiarities

   gridftp-aai-integration/aai-integration-mapping-users
   prerequisities-for-RCAuth.eu-certificates/prerequisities-for-RCAuth.eu-certificates

.. toctree::
   :maxdepth: 1
   :caption: Documentation on Documentation:

   documentation-notes/documentation-notes

.. Indices and tables
   ==================

.. * :ref:`genindex`
   * :ref:`modindex`
   * :ref:`search`
