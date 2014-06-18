Search
******

.. highlight:: bash

:doc:`solr`

ElasticSearch
=============

In your ``requirements/base.txt``, add the following::

  django-haystack==2.1.0
  pyelasticsearch==0.6.1

Maintenance
-----------

The flush process of an index basically frees memory::

  curl localhost:9200/_flush

Test
----

To check the install::

  curl -X GET 'http://localhost:9200'

SOLR
----

If SOLR was previously installed on a server, you will want to remove OpenJDK
and Tomcat before installing ElasticSearch::

  apt-get remove openjdk-7-jre-headless
  apt-get remove tomcat7