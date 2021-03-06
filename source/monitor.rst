Monitor
*******

.. highlight:: bash

You can use https://opbeat.com/ (or you could use our own Graphite monitoring
solution - which doesn't really work properly yet)!

Opbeat
======

Requirements:

.. code-block:: text

  # requirements/base.txt
  opbeat

.. tip:: Find the version number in :doc:`dev-requirements`

In ``settings/base.py`` for the project::

  THIRD_PARTY_APPS = (
      'opbeat.contrib.django',

  MIDDLEWARE_CLASSES = (
      'opbeat.contrib.django.middleware.OpbeatAPMMiddleware',

  OPBEAT = {
      'ORGANIZATION_ID': get_env_variable('OPBEAT_ORGANIZATION_ID'),
      'APP_ID': get_env_variable('OPBEAT_APP_ID'),
      'SECRET_TOKEN': get_env_variable('OPBEAT_SECRET_TOKEN'),
  }

.. note:: The ``OpbeatAPMMiddleware`` in the ``MIDDLEWARE_CLASSES`` must come
          first in the list.

.. note:: I thought about using the Opbeat logging config, but it doesn't have
          the same file handlers and has a ``mysite`` section (which I don't
          understand).

In ``settings/local.py``::

  OPBEAT['APP_ID'] = None

The Opbeat monitor is configured in the pillar file for the site e.g:

.. code-block:: yaml

  hatherleigh_info:
    profile: django
    domain: hatherleigh.info
    opbeat: 1234

.. note:: Find the ``APP_ID``, ``ORGANIZATION_ID`` and ``SECRET_TOKEN`` on the
          Opbeat app set-up wizard.

.. note:: Add the ``APP_ID`` to the site config.

Add the ``ORGANIZATION_ID`` and ``SECRET_TOKEN`` to the ``config/opbeat.sls``
file in the pillar e.g.

.. code-block:: yaml

  opbeat:
    organization_id: 1234
    secret_token: 1234

Graphite
========

.. _monitor_server:

Server
------

To create a monitor (Graphite) server, start by adding a ``monitor`` section to
the ``pillar``:

.. code-block:: yaml

  monitor:
    uwsgi_port: 3032
    db_pass: your-db-password
    secret_key: my-secret-key-generated-by-django
    domain: monitor.hatherleigh.info

Note:

- We normally install a monitor onto a separate server because our apps use
  python 3 and Graphite uses python 2 (not sure if they will work together).
- To generate a unique secret key, see :ref:`generate_secret_key`
- The ``domain`` is used to fill in the Django ``ALLOWED_HOSTS`` field.  You
  will probably want to copy this domain to the ``django`` pillar file (see
  below).

::

  psql -X -U postgres -c "CREATE ROLE graphite WITH PASSWORD '<your-db-password>' NOSUPERUSER CREATEDB NOCREATEROLE LOGIN;"
  psql -X -U postgres -c "CREATE DATABASE graphite WITH OWNER=graphite TEMPLATE=template0 ENCODING='utf-8';"

Diagnostics
-----------

Check storage schema::

  /opt/graphite/bin/validate-storage-schemas.py

Client
------

The monitor client is configured in the ``django`` pillar file e.g:

.. code-block:: yaml

  django:
    monitor: monitor.hatherleigh.info

.. note:: This will probably be the same as the domain name configured in the
          server (see :ref:`monitor_server` above).

Diagnostics
-----------

To run ``statsd`` without ``supervisord``::

  /usr/bin/nodejs /opt/statsd/stats.js /opt/statsd/localConfig.js

To view the messages received by ``statd``, edit ``/opt/statsd/localConfig.js``
and add ``dumpMessages: true`` e.g::

  {
      graphitePort: 2003,
      graphiteHost: "monitor.hatherleigh.info",
      port: 8125,
      dumpMessages: true,
      backends: [ "./backends/graphite" ]
  }

.. tip:: Don't forget to stop ``statsd`` in ``supervisorctl`` if running from
         the command line.

From `Looking Under the Covers of StatsD`_

To see the statistics from the management interface::

   echo "stats" | nc localhost 8126

   (echo "timers" | nc localhost 8126)
   (echo "counters" | nc localhost 8126)

To see if the monitor server is accepting connections::

   telnet monitor.hatherleigh.info 2003

To send some data to ``statsd``::

  echo "foo:1|c" | nc -u -w0 127.0.0.1 8125


.. _`Looking Under the Covers of StatsD`: http://blog.johngoulah.com/2012/10/looking-under-the-covers-of-statsd/
