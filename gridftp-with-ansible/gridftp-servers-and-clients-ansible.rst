========================================
GridFTP Servers and Clients with Ansible
========================================

We have created two Ansible roles and an example playbook for the
deployment of GridFTP servers and/or clients. The role will:

- Install globus software and dependencies as needed

- Configure gridftp servers

  - Main config in ``/etc/gridftp.conf``
  - Directory access restrictions in ``/etc/gridftp.d``
  - Host cert/key in ``/etc/grid-security``
  - CA certificates in ``/etc/grid-security/certificates``
  - Complete management of mappings in
    ``/etc/grid-security/grid-mapfile``

- Start the service and enable it at boot
- Open firewall ports if firewalld is detected
- Install fetch-crl and cron jobs on servers to maintain revocation lists
- Install UberFTP of clients

This playbook will also transfer SimpleCA certificates from the server
to all the clients, a step that is only needed if you don’t have access
to a real CA certificate and/or a real host certificate/key, ie when
using Vagrant.

Additionally, a Vagrantfile is provided to launch VirtualBox machines on
a local setup for testing purposes.

Most of the work is based on notes and conversations with Mikael Borg
and Harri Salminen:

- `Brief notes on gridftp transfers using proxy certificates from the
  ELIXIR CILogon service
  <https://docs.google.com/document/d/1vDhPU3hgG8xgzf_YrJmza9mbo2FbsjaobxlYWesqY9M>`__

- `GridFTP installation instructions for ELIXIR use
  <https://docs.google.com/document/d/1IogT-n3nKYCcs03CF1gTKW2jDmF09DK468fUOZplCwU>`__

Github repositories
===================

- `https://github.com/EMBL-EBI-TSI/ansible-playbook-gridftp <https://github.com/EMBL-EBI-TSI/ansible-playbook-gridftp>`__ *(version 1.2.0)*

- `https://github.com/EMBL-EBI-TSI/ansible-gridftp <https://github.com/EMBL-EBI-TSI/ansible-gridftp>`__ *(version 1.2.0)*

- `https://github.com/EMBL-EBI-TSI/ansible-simpleca <https://github.com/EMBL-EBI-TSI/ansible-simpleca>`__ (*version 1.0.0)*

Running the playbook
====================

General instructions can be found in the `playbook
repository <https://github.com/EMBL-EBI-TSI/ansible-playbook-gridftp>`__
and further documentation of the roles can be found in their respective
repositories. In this document we will complete that setup specifically
for ELIXIR with CILogon. We’ll go through three examples:

-  Full setup in Vagrant/VirtualBox
-  Setting up a GridFTP server
-  Adding GridFTP clients

In the following sections we assume you have a working Ansible
environment (see below), cloned the
`ansible-playbook-gridftp <https://github.com/EMBL-EBI-TSI/ansible-playbook-gridftp>`__
repository and installed the role dependencies::

  git clone https://github.com/EMBL-EBI-TSI/ansible-playbook-gridftp.git
  cd ansible-playbook-gridftp
  ansible-galaxy install -r requirements.yml

Adding ELIXIR specifics to the playbook
---------------------------------------

Before launching servers, to make GridFTP works with ELIXIR’s CA we
first need to add the CA certificate and user mappings to the playbook.

Elixir AAI integration with CILogon
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

While the integration is still work in progress at Elixir AAI, the
interim solution is to:

-  Install ``rcauth-pilot-ica-g1`` and ``DDCA-Root-G1-CA`` certificates in
   servers and clients. These are already included in the playbook
   (``group_vars/all/main.yml`` and
   ``group_vars/gridftp-servers/main.yml``).

-  Users should get their certificate manually by:

   a. Sign up for an ELIXIR account if you haven’t already at `https://www.elixir-europe.org/intranet <https://www.elixir-europe.org/intranet>`__.

   b. Obtain proxy certificate from `https://elixir-cilogon-mp.grid.cesnet.cz/vo-portal/ <https://elixir-cilogon-mp.grid.cesnet.cz/vo-portal/>`__ by pressing Get Proxy button.

   c. Copy the certificate to ``/tmp/x509up_u1000`` with permissions ``0600`` on the gridftp client host, where ``1000`` is your user id (``id -u``).

   d. Check the proxy certificate with ``grid-proxy-info``.

Issues with CESNET’s certificate at globus.du3.cesnet.cz
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We have noticed some authentication problems when connecting to
``globus.du3.cesnet.cz``. The following workaround is necessary only on
gridftp clients and is already included in the playbook
(``group_vars/gridftp-clients/main.yml``):

- Add “TERENA SSL CA 3” certificate with the modified signing policy. `Download <https://www.terena.org/activities/tcs/repository-g3/TERENA_SSL_CA_3.pem>`__ the certificate and save it in ``files/TERENA-SSL-CA-3.pem``.

- Add “TERENA SSL CA 3” DN to the signing policy of the root DigiCert certificate.

Mappings
~~~~~~~~

We need to let the GridFTP server know how to map DNs to local users.
This is done by listing your mappings in
``group_vars/gridftp-servers/main.yml``. For example::

  gridftp_mappings:
  - ln: nobody
  dn: /DC=eu/DC=rcauth/DC=rcauth-clients/O=elixir-europe.org/CN=your identity

You can find your DN using ``grid-proxy-info`` on a GridFTP client. You can
add as many as needed, but remember to run ansible-playbook on every
change.

Full setup in Vagrant/VirtualBox
--------------------------------

Assuming Vagrant and Virtualbox are also installed on the system,
running vagrant up
should bring up a fully functional gridftp server and gridftp client.
The only piece missing is a valid user certificate or proxy. For testing
purposes the Elixir proxy should be sufficient (see “Elixir AAI
integration with CILogon” above). Once that is setup, you can try
copying a file::

  Vagrant ssh gridftp-client.local
  echo hello > /tmp/hello.txt
  globus-url-copy -nodcau file:///tmp/hello.txt
  gsiftp://gridftp-server.local/tmp/yeah.txt

Assuming your DN has been added to other gridftp servers, you can run
the script ``test_endpoints.sh`` to test bidirectional transfers from each
endpoint listed in the script from your gridftp client::

  Vagrant ssh gridftp-client.local
  /vagrant/test_endpoints.sh hx-gridftp-test.ebi.ac.uk/tmp/luisg \
    test-gridftp.csc.fi/mnt/gridftp/testaaja/luisg \
    gridftp.bio.nic.funet.fi/home/test/luisg \
    gridftp.bils.se/home/amelie/luisg \
    globus.du3.cesnet.cz/exports/home/luisg

Setting up a GridFTP server
---------------------------

In this section we will deploy a real GridFTP server, which is what most
of you came here to for. We need to do the following changes:

-  Setup our inventory
-  Simplify playbook by removing plays that only work on gridftp-clients hosts, simpleca, or vagrant.
-  Add mappings between DNs and local users (see above).
-  Add the host certificate and key

Inventory
~~~~~~~~~

We need to tell ansible which machines to target and that is best done
with an inventory. Create a file called ``production`` in the current
directory with the following contents (referring your own FQDN)::

  [gridftp-servers]
  my-gridftp.server.com

Simplify playbook
~~~~~~~~~~~~~~~~~

The following plays in site.yml should be enough (with the first play
just there for best practices)::

  ---
  - name: Gather all facts
    hosts: all
    tasks: []
  - name: gridftp servers
    hosts: gridftp-servers
    roles:
      - {role: gridftp, gridftp\_mode: server}

Host certificate and key
~~~~~~~~~~~~~~~~~~~~~~~~

On a real GridFTP server you will need a valid host certificate/key
pair. You should ask your local CA how to obtain these. Once you have
them you might need to manipulate them to convert them to PEM format and
remove the password from the host key. Now they can be referenced in the
variables found in ``group_vars/gridftp-servers/main.yml`` and
``group_vars/gridftp-servers/vault.yml`` (see below on how to create this
file). In the following example we will use ansible’s vault to keep the
host key secured. Note that if you don’t want to bother with the vault
at the moment, you can input the value of the host key directly in this
file, but do not push this to any repository because your key would be
compromised.

We start with ``group_vars/gridftp-servers/main.yml``::

  gridftp_host_cert: |
    -----BEGIN CERTIFICATE-----
    .... contents of your certificate ....
    .... contents of your certificate ....
    -----END CERTIFICATE-----
  gridftp_host_key: '{{vault_gridftp_host_key}}'

The last line will set the value of the ``host_key`` to the one we input in
the secured file. Now we create the vault at
``group_vars/gridftp-servers/vault.yml``::

  ansible-vault --ask-vault-pass create group\_vars/gridftp-servers/vault.yml

And enter the following content::

  vault_gridftp_host_key: |
    -----BEGIN RSA PRIVATE KEY-----
    .... contents of your key ....
    .... contents of your key ....
    -----END RSA PRIVATE KEY-----

Running the playbook
~~~~~~~~~~~~~~~~~~~~

Now that all variables are in place, it is time to run ansible::

  ansible-playbook -i production -u root site.yml

Ansible will ask you for the password to access your target machine as
root and also the password to access the vault. All this can be
automated by providing paths to files containing a private ssh key that
pairs with a public key deployed to the target machine and another file
that contains the password (in plain text) for the vault. Note to keep
both files secured if you follow this route. For example::

  ansible-playbook --private-key=/path/to/ssh.key
  --vault-password-file=/path/to/vault/pass -u root site.yml

Adding GridFTP clients
----------------------

If you need to bootstrap one or more GridFTP clients you can just add
the gridftp clients play in ``site.yml``::

  - name: gridftp clients
    hosts: gridftp-clients
    roles:
      - {role: gridftp, gridftp\_mode: client}

And if you also want to use the SimpleCA certificates generated by
globus upon install of gridftp, just leave the full ``site.yml`` file
intact.

Update the inventory with your gridftp clients::

  [gridftp-clients]
  my-gridftp.client1.com
  my-gridftp.client2.com

Of course you now need to run ansible-playbook (see above).

Installing Ansible, Vagrant and VirtualBox
==========================================

Vagrant and VirtualBox are better installed using your package manager.
For Ansible, you can also use your package manage, or alternatively I
recommend just cloning from git (remember to source ``env-setup`` before
running ansible)::

  export PROVISION=~/provision
  mkdir $PROVISION
  cd $PROVISION
  git clone git://github.com/ansible/ansible.git --recursive
  source $PROVISION/ansible/hacking/env-setup

It is also worth taking the time to configure ansible in a custom
``ansible.cfg``::

  export ANSIBLE_CONFIG=$PROVISION/ansible.cfg
  cat <<EOF >$ANSIBLE_CONFIG
  [defaults]
  vault_password_file = /path/to/vault/pass/file
  private_key_file = /path/to/private/key/file
  roles_path = vendor/roles:/path/to/ansible/roles
  EOF

Note that
`ansible-playbook-gridftp <https://github.com/EMBL-EBI-TSI/ansible-playbook-gridftp>`__
already includes the minimal ``ansible.cfg`` configuration to make it work
with this document.

Changes
=======

1.0.0 (10 May 2016)
  Initial version
1.1.0 (14 June 2016)
  - Playbook:
    - Workaround for CESNET's CA issues
  - Gridftp role:
    - Support certificates from file
1.2.0 (16 June 2016)
  - Playbook:
    - Include certs needed by Elixir in the repository
    - Add script to test endpoints
  - Gridftp role:
    - Restrict directories in server
    - Update revocation lists with fetch-crl
    - Install UberFTP on clients

