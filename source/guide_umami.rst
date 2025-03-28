.. highlight:: console

.. author:: Thomas Johnson <https://johnson.tj/>
.. author:: Andreas Fuchs <https://anfuchs.de/>

.. tag:: lang-nodejs
.. tag:: web
.. tag:: analytics

.. sidebar:: Logo

  .. image:: _static/images/umami.svg
      :align: center

#####
umami
#####

.. tag_list::

umami_ is a simple, easy to use, self-hosted web analytics solution. The goal is to provide you with a friendlier, privacy-focused alternative to Google Analytics and a free, open-sourced alternative to paid solutions. Umami collects only the metrics you care about and everything fits on a single page.

----

.. error::

  This guide seems to be **broken** for the current versions of Umami, we would be
  happy if you want to work on a solution and create a Pull Request.
  See also the related issue: https://github.com/Uberspace/lab/issues/1882

.. note:: For this guide you should be familiar with the basic concepts of

  * :manual:`Node.js <lang-nodejs>` and its package manager :manual_anchor:`npm <lang-nodejs.html#npm>`
  * :manual:`MySQL <database-mysql>`
  * :manual:`supervisord <daemons-supervisord>`
  * :manual:`domains <web-domains>`

Prerequisites
=============

Node and npm
------------

We're using :manual:`Node.js <lang-nodejs>` version 20:

.. code-block:: console

 [isabell@stardust ~]$ uberspace tools version show node
 Using 'Node.js' version: '20'
 [isabell@stardust ~]$

.. include:: includes/my-print-defaults.rst

Your website domain needs to be set up:

.. include:: includes/web-domain-list.rst

Installation
============

We clone the repository to our home directory and install the application.

.. code-block:: console

 [isabell@stardust ~]$ git clone https://github.com/umami-software/umami.git
 [isabell@stardust ~]$ cd umami
 [isabell@stardust umami]$ yarn install
 (...)
 [isabell@stardust umami]$


Configuration
=============

After the installation you need to create the database.

.. code-block:: console

 [isabell@stardust umami]$ mysql -e "CREATE DATABASE ${USER}_umami"
 [isabell@stardust umami]$

Use your favorite editor to create ``~/umami/.env`` with the following content:

.. code-block:: ini

  DATABASE_URL=mysql://isabell:mypassword@localhost:3306/isabell_umami

.. warning:: Currently there seems to be a migration issue that leads to a fatal error. If you encounter an `Error: P3018` on the command `prisma migrate deploy` you need to apply a fix. Run the following command inside your `umami/`-directory: ``npx prisma migrate resolve --applied "05_add_visit_id"``

Now you can create the production build:

.. code-block:: console

 [isabell@stardust umami]$ yarn build
 [isabell@stardust umami]$

.. warning:: In newer versions, sometimes the build process fails without any errors in the `next build` stage. This is due to Uberspace killing the process for needing to much memory. If this happens, you will not be able to start the app – it will say `Error: Could not find a production build in the '/home/isabell/umami/.next [...]' directory`. Try running the build process via ``NODE_OPTIONS=--max_old_space_size=512 npm run build --debug`` to limit the RAM usage and build the app successfully.

The first time the build is run successfully, it will create all the required database tables in your database.


Setup daemon
------------

Create ``~/etc/services.d/umami.ini`` with the following content:

.. code-block:: ini

 [program:umami]
 directory=%(ENV_HOME)s/umami
 command=yarn start
 autostart=yes
 autorestart=yes

.. include:: includes/supervisord.rst

Configure web server
--------------------

.. note::

    umami is running on port 3000.

.. include:: includes/web-backend.rst

Configure application
---------------------

Your Umami installation will create a default administrator account with the username ``admin`` and the password ``umami``.

.. warning:: The first thing you will want to do is log in and change your password.

For more information check the umami_ documentation.

Updates
-------

.. note:: Check the git repository_ regularly to stay informed about changes.

To update the application, stop the daemon and repeat the installation step.

.. _umami: https://umami.is/docs/install
.. _repository: https://github.com/umami-software/umami

----

Tested with umami 2.13.2 and Uberspace 7.16.2

.. author_list::
