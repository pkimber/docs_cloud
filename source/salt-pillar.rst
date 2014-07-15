Pillar
******

.. highlight:: yaml

.. warning::

  The Salt pillar data must be kept secure.  Do not push to a public repository
  such as GitHub or BitBucket

Sites
=====

To set-up a new site (or sites) on a server, create or edit a file in the
pillar, ``sites`` folder e.g. ``sites/mysites.sls``.  The file should contain
details of the sites to be deployed onto this server e.g::

  sites:
    pkimber_net:
      profile: django
      db_pass: password
      db_type: psql
      domain: pkimber.net
      secret_key: 'my-secret-key-generated-by-django'
      ssl: False
      uwsgi_port: 3035
    another_site:
      profile: django
      db_pass: password2
      db_type: psql
      domain: hatherleigh.net
      secret_key: 'another-secret-key-generated-by-django'
      ssl: True
      uwsgi_port: 3036
      ftp: True
      ftp_password: "generated-using-mkpasswd-see-ftp-notes"

FTP
---

:doc:`ftp`

LAN
---

If you want to install a site to your local area network, then add the
``lan`` option to your site configuration e.g::

  sites:
    my_site:
      db_pass: password
      domain: pkimber.net
      lan: True
      secret_key: 'my-secret-key-generated-by-django'
      ssl: False
      uwsgi_port: 3038

.. note::

  If you enable the ``lan`` option then you (currently) cannot use ``ssl``.

  nginx will be configured with an empty server name so only one site can be
  installed on the server.

.. warning::

  If you enable the ``lan`` option, Django site will set ``ALLOWED_HOSTS`` to
  ``*``   This is a security risk for public web sites.

Mail
----

:doc:`app-mail`

pip and devpi
-------------

:doc:`devpi`

Secret Key
----------

To generate a new secret key, use the Django extensions application::

  pip install django-extensions

.. code-block:: python

  THIRD_PARTY_APPS = (
      'django_extensions',

::

  django-admin.py generate_secret_key

Database
========

The fabric :doc:`fabric-release` task uses a ``prefix`` parameter for
identifying your modules.  This ``prefix`` is also used to lookup the
database IP address for your site when running the :doc:`fabric-deploy`
command.  So, for example, if your prefix is ``pkimber``, you should have a
file in your pillar called::

  db/settings.sls

This file should contain the IP address of your server (or ``localhost`` if
your database is installed on the same server as your site) e.g::

  postgres_settings:
    listen_address: localhost

Users
=====

To create users on your server, add a ``users`` section to your pillar in the
following format::

  users:
    patrick:
      uid: 7501
      fullname: Patrick Kimber
      password: "abc"
      sudo: True
      keys:
        - ssh-rsa AAAAB3...patrick@hamm
        - ssh-rsa AAAAB3...patrick@rex
    greg:
      uid: 7504
      fullname: Greg Smith
      password: "xyz"
      sudo: True
      keys:
        - ssh-rsa AAAAB3...greg@buzz

To create the password hash (where ``<password>`` is your password)::

  mkpasswd -m sha-512 <password>

The ``keys`` are a list of public ssh keys.

Validate
========

To validate the pillar files, use the fabric ``valid`` task e.g::

  cd fabric
  fab valid:server_name=drop-temp,site_name=hatherleigh_net