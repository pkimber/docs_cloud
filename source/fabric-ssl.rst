SSL Certificate
***************

Configuration
=============

The following task expects the certificate to be in a folder for the domain
within a folder called ``ssl-cert`` which is placed beside the ``salt-pillar``
folder e.g:

::

  ├── salt-pillar
  ├── ssl-cert
  │   ├── hatherleigh.info
  │   │   ├── server.key
  │   │   └── ssl-unified.crt
  │   ├── pkimber.net
  │   │   ├── server.key
  │   │   └── ssl-unified.crt

Make sure the Salt pillar is configured correctly for the site.  You need to
set ``ssl`` to ``True`` and the ``domain`` must match the certificate e.g:

::

  domain: hatherleigh.info
  ssl: True

Usage
=====

To copy an SSL certificate to a web server:

::

  fab -H server ssl_cert:hatherleigh_net

.. note:: The task will ask you to enter the ``sudo`` password for the user on
  the server.

.. note:: This task expects the ``/srv/`` folder to exist on the server.  I
  don't know what creates the folder... so I am currently not sure how it gets
  there!