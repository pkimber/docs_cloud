Amazon
******

.. highlight:: bash

Thank you to the authors of the following articles for their help:

- `An Introduction to the AWS Command Line Tool Part 2`_
- `An Introduction to the AWS Command Line Tool`_
- `Using Security Groups`_

Install
=======

Create your Access Keys by clicking on your user name in the web console and
selecting *Security Credentials*

Install the command line tool and then configure::

  pip install awscli
  aws configure

Enter your access key and secret key::

  AWS Access Key ID [None]:
  AWS Secret Access Key [None]:
  Default region name [None]: eu-west-1
  Default output format [None]: table

I chose ``eu-west-1`` and ``table`` for the output format.

The following two commands will check the command line tool is running::

  aws ec2 describe-regions
  aws ec2 describe-availability-zones

Security Groups
===============

.. note:: The security groups need to be set-up once.  After setting up the
          security groups, we just assign them to instances.

Graphite
--------

Graphite carbon server needs to listen to messages on port 2003::

  aws ec2 create-security-group \
      --group-name graphite \
      --description "Graphite Security Group"
  aws ec2 authorize-security-group-ingress \
      --group-name graphite \
      --protocol tcp \
      --cidr 0.0.0.0/0 \
      --port 2003

Web
---

Create security groups::

  aws ec2 create-security-group \
      --group-name web \
      --description "Web Security Group"
  # ssh
  aws ec2 authorize-security-group-ingress \
      --group-name web \
      --protocol tcp \
      --cidr 0.0.0.0/0 \
      --port 22
  # http
  aws ec2 authorize-security-group-ingress \
      --group-name web \
      --protocol tcp \
      --cidr 0.0.0.0/0 \
      --port 80
  # https
  aws ec2 authorize-security-group-ingress \
      --group-name web \
      --protocol tcp \
      --cidr 0.0.0.0/0 \
      --port 443

Salt Master
-----------

To allow inbound connections to a Salt master...

Create the security group::

  aws ec2 create-security-group \
      --group-name master \
      --description "Salt Master Security Group"
  aws ec2 authorize-security-group-ingress \
      --group-name master \
      --protocol tcp \
      --cidr 0.0.0.0/0 \
      --port 4505
  aws ec2 authorize-security-group-ingress \
      --group-name master \
      --protocol tcp \
      --cidr 0.0.0.0/0 \
      --port 4506

Database
--------

:doc:`amazon-database`

Salt Cloud
==========

Key
---

Create a private and public SSH key (replace ``my_salt_cloud_key`` with a key
name of your choice)::

  sudo ssh-keygen -f /etc/salt/my_salt_cloud_key -t rsa -b 4096
  aws ec2 import-key-pair --key-name my_salt_cloud_key \
        --public-key-material file:///etc/salt/my_salt_cloud_key.pub

To list key pairs::

  aws ec2 describe-key-pairs

To remove a key pair::

  aws ec2 delete-key-pair --key-name my_salt_cloud_key

Provider
--------

Add a provider to ``~/repo/dev/module/deploy/salt-cloud/cloud.providers`` e.g:

.. code-block:: yaml

  kb_eu_west_1_public_ips:
    minion:
      master: master.pkimber.net
    ssh_interface: public_ips
    id: your-amazon-id
    key: 'your-amazon-key'
    keyname: my_salt_cloud_key
    private_key: /etc/salt/my_salt_cloud_key
    securitygroup: web
    location: eu-west-1
    availability_zone: eu-west-1a
    size: Micro Instance
    del_root_vol_on_destroy: True
    ssh_username: ubuntu
    rename_on_destroy: True
    provider: ec2

- Replace ``your-amazon-id`` with your *AWS Access Key ID* (see above)
- Replace ``your-amazon-key`` with your *AWS Secret Access Key* (see above)
- Update the ``keyname`` and ``private_key`` so they match the details for your
  own key.
- Find the ``availability_zone`` for your ``location`` by running
  ``aws ec2 describe-availability-zones``
- Make sure the ``securitygroup`` matches the name you chose.

.. note:: For information on the above settings, see
          http://salt-cloud.readthedocs.org/en/latest/topics/aws.html

Profile
-------

Add an image to ``~/repo/dev/module/deploy/salt-cloud/cloud.profiles`` e.g:

.. code-block:: yaml

  base_ec2_private:
    provider: kb_eu_west_1_private_ips
      image: ami-ff498688

- I chose ``ami-ff498688`` from
  http://cloud-images.ubuntu.com/releases/14.04/release/ (which I hope is a 32
  bit micro instance).
- The ``provider`` is the name of the section in ``cloud.providers``

Usage
=====

Create a test server::

  sudo -i
  salt-cloud \
    --profiles=/home/patrick/repo/dev/module/deploy/salt-cloud/cloud.profiles \
    --providers-config=/home/patrick/repo/dev/module/deploy/salt-cloud/cloud.providers \
    --profile base_ec2_private \
    test-ec2

- Replace ``patrick`` with your user name on the workstation.
- Replace ``test-ec2`` with the name of the server you want to create.

Make a note of the ``publicIp`` and ``instanceId``.  If you need to find the
instance ID later::

  aws ec2 describe-instances --filter Name=tag:Name,Values=test-ec2

- replace ``test-ec2`` with the name of the server you are looing for.

Log into your new server::

  sudo -i
  eval `ssh-agent`
  ssh-add /etc/salt/my_salt_cloud_key
  ssh ubuntu@54.77.12.170

.. note:: The IP address of the new server is the ``publicIp`` (see above).

To get root access (on this Ubuntu server)::

  sudo -i

Security Groups
---------------

For a web server, we need to add the ``db`` security group.  Make a note of the
``GroupId`` for the ``web`` and ``db`` security groups::

  aws ec2 describe-security-groups --group-names db
  aws ec2 describe-security-groups --group-names web

Add these two security groups to the web server you just created::

  aws ec2 modify-instance-attribute \
    --instance-id <instance id> \
    --groups <security group id> <db security group id>

- Replace ``<instance id>`` with the ``InstanceId`` of the server you just
  created.
- Replace ``<security group id>`` with the ID of the ``web`` security group
  (see ``awscli``).
- Replace ``<db security group id>`` with the ID of the ``db`` security group.

Database
--------

To find the *Endpoint* *Address* for your database instance::

  aws rds describe-db-instances

You should be able to connect to your database instance using ``psql``::

  psql \
    --host=my-db-instance.cmf1ips9eg9s.eu-west-1.rds.amazonaws.com \
    --username=postgres postgres

- Enter the master user password when prompted (see ``apg`` in
  `RDS, Create database`_).


.. _`An Introduction to the AWS Command Line Tool Part 2`: http://www.linux.com/news/featured-blogs/206-rene-cunningham/764536-an-introduction-to-the-aws-command-line-tool-part-2
.. _`An Introduction to the AWS Command Line Tool`: http://www.linux.com/learn/tutorials/761430-an-introduction-to-the-aws-command-line-tool
.. _`RDS, Create database`: https://www.pkimber.net/howto/amazon/rds.html#create-database
.. _`Using Security Groups`: http://docs.aws.amazon.com/cli/latest/userguide/cli-ec2-sg.html
