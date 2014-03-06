Step by step server installation
================================

This document provides a step by step instruction to install the PIM on servers based on CENTOS 6


Prerequisite
-------------

In order to install Akeneo, you will have to download the Akeneo PIM Standard Edition archive file from http://www.akeneo.com/download/
You can also use an archive supplied by your integrator.

An akeneo user and an akeneo group must be created to run crontabs and PIM commands. 

By convention, all commands preceded by a # should be issued with root, and all commands preceded by a $ should be 
issued with the akeneo user.


Configure EPEL and IUS
----------------------


.. code-block:: bash 
    :linenos:

    $ wget http://dl.iuscommunity.org/pub/ius/stable/CentOS/6/x86_64/epel-release-6-5.noarch.rpm 
    $ wget http://dl.iuscommunity.org/pub/ius/stable/CentOS/6/x86_64/ius-release-1.0-11.ius.centos6.noarch.rpm
    # rpm -i epel-release-6-5.noarch.rpm
    # rpm -i ius-release-1.0-11.ius.centos6.noarch.rpm
    

Installing MySQL
****************


.. code-block:: bash
    :linenos:

    # yum install mysql-server
    # chkconfig mysql-server on


Installing Apache
*****************


.. code-block:: bash 
    :linenos:

    # yum install httpd
    # chkconfig httpd on

Installing PHP
**************


.. code-block:: bash 
    :linenos:

    # yum install php54
    # yum install php54-gd
    # yum install php54-mysql
    # yum install php54-intl
    # yum install php54-mcrypt
    # yum install php54-soap
    # yum install php54-xml
    # yum install java-1.7.0-openjdk 


Installing Java
***************

.. code-block:: bash
    :linenos:

    # yum install java-1.7.0-openjdk 


Installing PHP opcode and data cache
************************************

.. code-block:: bash 
    :linenos:

    # yum install php54-pecl-apc


System configuration
--------------------
MySQL
*****

* Creating a MySQL database and user for the application


.. code-block:: bash 
    :linenos:

    $ mysql -u root
    mysql> CREATE DATABASE akeneo_pim;
    mysql> GRANT ALL PRIVILEGES ON akeneo_pim.* TO akeneo_pim@localhost IDENTIFIED BY 'akeneo_pim';
    mysql> EXIT

PHP
***
* Setting up PHP Apache configuration


.. code-block:: bash 
    :linenos:

    # vi /etc/php.ini
    memory_limit = 256M
    date.timezone = Etc/UTC

* Setting up PHP CLI configuration


.. code-block:: bash 
    :linenos:

    # cp /etc/php.ini /etc/php-cli.ini
    # vi /etc/php-cli.ini
    memory_limit = 768M
    date.timezone = Etc/UTC

.. note::
    Use the time zone corresponding to our location, for example *America/Los_Angeles*, *Europe/Berlin*.
    See http://www.php.net/timezones for the list of available timezones.

Apache
******
To avoid spending too much time on rights problems between the akeneo user and the Apache user, an easy configuration
is to use same user for both processes.


Use your identifiers for Apache
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash 
    :linenos:

    # service httpd stop
    # vi /etc/httpd/conf/httpd.conf
    User akeneo
    Group akeneo

Change permissions for PHP session files
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash 
    :linenos:

    # chown -R akeneo:akeneo /var/lib/php/session


Installing Akeneo PIM
---------------------

Extracting the archive
**********************
.. code-block:: bash 
    :linenos:

    $ cd /path/to/installation
    $ tar -xvzf /path/to/pim-community-standard-version.tar.gz

.. note::
    Replace the */path/to/installation* by the path to directory where you want to install the PIM.

    Replace the */path/to/pim-community-standard-version.tar.gz* by the location and name of the archive
    you have downloaded from http://www.akeneo.com/download.

.. warning::

    After the extraction, a new directory usually called *pim-community-standard-version* is created
    inside the */path/to/installation* directory.

    It will be our PIM root directory and will be refered as */path/to/pim/root* in the following instructions.

Installing Akeneo
*****************
.. code-block:: bash 
    :linenos:

    $ cd /path/to/pim/root
    $ php app/console pim:install --env=prod
    $ php app/console cache:clear --env=prod

Configuring the virtualhost
---------------------------

Add a vhost to your Apache config
*********************************


Adapt and add the following content to your /etc/httpd/conf/httpd.conf file

.. code-block:: bash
    :linenos:

    NameVirtualHost *:80
    <VirtualHost *:80>
        ServerName akeneo-pim.local

        DocumentRoot /path/to/pim/root/web/
        <Directory /path/to/pim/root/web/>
            Options Indexes FollowSymLinks MultiViews
            AllowOverride All
            Order allow,deny
            allow from all
        </Directory>
        ErrorLog /var/log/httpd/akeneo-pim_error.log

        LogLevel warn
        CustomLog /var/log/httpd/akeneo-pim_access.log combined
    </VirtualHost>



Restart Apache
**************


.. code-block:: bash 
    :linenos:

    # service httpd restart


Configuring the crontab
***********************


.. code-block:: bash 
    :linenos:

    $ crontab -e
    0 * * * *  /usr/bin/php /path/to/pim/root/app/console pim:completeness:calculate