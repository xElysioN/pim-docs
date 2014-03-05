Step by step installation on developement workstation
=====================================================

This document provides a step by step instruction to install the PIM on development workstations based on CENTOS 6



Prerequisite
-------------
In order to install Akeneo, you will have to download the Akeneo PIM Standard Edition archive file from http://www.akeneo.com/download/


System installation
-------------------

Configure EPEL and IUS
----------------------

.. code-block:: bash 
    :linenos:
    $ wget http://dl.iuscommunity.org/pub/ius/stable/CentOS/6/x86_64/epel-release-6-5.noarch.rpm 
    $ wget http://dl.iuscommunity.org/pub/ius/stable/CentOS/6/x86_64/ius-release-1.0-11.ius.centos6.noarch.rpm
    $ sudo rpm -i epel-release-6-5.noarch.rpm
    $ sudo rpm -i ius-release-1.0-11.ius.centos6.noarch.rpm
    

Installing MySQL
****************


.. code-block:: bash
    :linenos:

    $ sudo yum install mysql-server
    $ sudo chkconfig mysql-server on

Installing Apache
*****************


.. code-block:: bash 
    :linenos:

    $ sudo yum install httpd
    $ chkconfig httpd on

Installing PHP
**************


.. code-block:: bash 
    :linenos:
    $ sudo yum install php54
    $ sudo yum install php54-gd
    $ sudo yum install php54-mysql
    $ sudo yum install php54-intl
    $ sudo yum install php54-mcrypt
    $ sudo yum install php54-soap
    $ sudo yum install php54-xml
    $ sudo yum install java-1.7.0-openjdk 


Installing Java
***************

.. code-block:: bash
    :linenos:

    $ sudo yum install java-1.7.0-openjdk 


Installing PHP opcode and data cache
************************************

.. code-block:: bash 
    :linenos:

    $ sudo yum install php54-pecl-apc


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

    $ sudo gedit /etc/php.ini
    memory_limit = 256M
    date.timezone = Etc/UTC

* Setting up PHP CLI configuration


.. code-block:: bash 
    :linenos:

    $ sudo cp /etc/php.ini /etc/php-cli.ini
    $ sudo gedit /etc/php-cli.ini
    memory_limit = 768M
    date.timezone = Etc/UTC

.. note::
    Use the time zone corresponding to our location, for example *America/Los_Angeles*, *Europe/Berlin*.
    See http://www.php.net/timezones for the list of available timezones.

Apache
******
To avoid spending too much time on rights problems between the installing user and the Apache user, an easy configuration
is to use same user for both processes.


Get your identifiers
^^^^^^^^^^^^^^^^^^^^
**Ubuntu 12.10 & 13.10**

.. code-block:: bash 
    :linenos:

    $ id
    uid=1000(my_user), gid=1000(my_group), ...

In this example, the user is *my_user* and the group is *my_group*.

Use your identifiers for Apache
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
**Ubuntu 12.10 & 13.10**

.. code-block:: bash 
    :linenos:

    $ sudo service httpd stop
    $ sudo gedit /etc/httpd/conf/httpd.conf
    User my_user
    Group my_group


Start Apache
^^^^^^^^^^^^

.. code-block:: bash 
    :linenos:

    $ sudo service httpd start


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
Enabling Apache mod_rewrite
***************************
**Ubuntu 12.10 & Ubuntu 13.10**

.. code-block:: bash 
    :linenos:

    $ sudo a2enmod rewrite

Creating the vhost file
***********************
**Ubuntu 12.10**

.. code-block:: bash 
    :linenos:

    $ sudo gedit /etc/apache2/sites-available/akeneo-pim.local

**Ubuntu 12.10**

.. code-block:: apache
    :linenos:
   
    <VirtualHost *:80>
        ServerName akeneo-pim.local

        DocumentRoot /path/to/pim/root/web/
        <Directory /path/to/pim/root/web/>
            Options Indexes FollowSymLinks MultiViews
            AllowOverride All
            Order allow,deny
            allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/akeneo-pim_error.log

        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/akeneo-pim_access.log combined
    </VirtualHost>

**Ubuntu 13.10**

.. code-block:: bash 
    :linenos:

    $ sudo gedit /etc/apache2/sites-available/akeneo-pim.local.conf


**Ubuntu 13.10**

.. code-block:: apache
    :linenos:
   
    <VirtualHost *:80>
        ServerName akeneo-pim.local

        DocumentRoot /path/to/pim/root/web/
        <Directory /path/to/pim/root/web/>
            Options Indexes FollowSymLinks MultiViews
            AllowOverride All
            Require all granted
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/akeneo-pim_error.log

        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/akeneo-pim_access.log combined
    </VirtualHost>

.. note::

    The difference in Virtual Host configuration between Ubuntu 12.10
    and Ubuntu 13.10 is the result of the switch from Apache 2.2 to
    Apache 2.4. See https://httpd.apache.org/docs/2.4/upgrading.html
    for more explanation.

Enabling the virtualhost
************************
**Ubuntu 12.10 & Ubuntu 13.10**

.. code-block:: bash
    :linenos:

    $ sudo a2ensite akeneo-pim.local
    $ sudo apache2ctl -t
    $ sudo service apache2 restart


Adding the vhost name
*********************
**Ubuntu 12.10 & 13.10**

.. code-block:: bash 
    :linenos:

    $ sudo gedit /etc/hosts
    127.0.0.1    akeneo-pim.local

Testing your installation
-------------------------
Go to http://akeneo-pim.local/ and log in with admin/admin.

If you can see the dashboard, congratulations, you have successfully installed Akeneo PIM !

You can as well access the dev environment on http://akeneo-pim.local/app_dev.php

If you have an error, it means that something went wrong in a previous step. So please check all error output of all instructions.
