A *production-ready* image for Odoo 7 & 8 & 9alpha
==================================================

This image weighs just over 1Gb. Keep in mind that Odoo is a very extensive suite of business applications written in Python. We designed this image with built-in external dependencies and almost nothing useless. It is used from development to production on version 7.0 and 8.0 with various community addons.

!!!Latest changes!!!
====================
This image has been "applified" and behaves just like a binary application with several options.

Try running ```docker run --rm xcgd/odoo``` and see what happens :)

Check the [BitBucket project page][2] for contributing, discussing and reporting issues.
This README is updated with regards to your questions. Thank you for your help!

Recent changes: 

- when binding the volume /opt/odoo/etc, the default `odoo.conf` file is provided if none is found in the host folder
- fixed some permission issues when binding volumes

Odoo version
============

This docker builds with a *tested* version of Odoo (formerly OpenERP) AND related dependencies. We do not intend to *follow the git*. The packed versions of Odoo have always been tested against our CI chain and are considered as *production grade* (apart from version 9 which is still is alpha stage). We update the revision pretty often, though :)

This is important to do in this way (as opposed to a nightly build) because we want to ensure reliability and keep control of external dependencies.

You may use your own sources simply by binding your local Odoo folder to /opt/odoo/sources/odoo/

Here are the current revisions from https://github.com/odoo/odoo for each docker tag

    # production grade
    xcgd/odoo:7.0	a0484cbe45abb5dc393d3229ee1c6d24a4dfae23 (branch 7.0)
    xcgd/odoo:8.0	d18d606a55330a6b9fd19596d8b12c4e9e0a4b57 (branch 8.0)

    # playing only
    xcgd/odoo:latest	7d3cda215a85dd81571a5dde3753fd2f954f6ccf (branch master/9alpha)

The tar is built with:

    tar czvf ../odoo${version}.tgz *

Prerequisites
=============

xcgd/postgresql
---------------

you'll need [xcgd/postgresql][1] docker image or any other PostgreSQL image of your choice that you link with Odoo under the name `db`:

    $ docker run --name="pg93" xcgd/postgresql

Note: read the instructions on how to use this image with data persistance. You also may use a distant server and change the configuration file accordingly.

Start Odoo
----------

Run Odoo in the background, assuming you named your PostgreSQL container `pg93` and target Odoo version 7.0:

    $ docker run -p 8069:8069 --rm --name="xcgd.odoo" --link pg93:db xcgd/odoo:7.0 start


WARNING: note that we aliased the PostgreSQL as `db`. This is ARBITRARY since we use this alias in the configuration files.

If docker starts without issues, just open your favorite browser and point it to http://localhost:8069	

The default admin password is `somesuperstrongadminpasswdYOUNEEDTOCHANGEBEFORERUNNING`

If you want to change the Odoo configuration with your own file you can bind it easily like so: 

    # let's pretend your configuration is located under /opt/odoo/instance1/etc/ on your host machine, you can run it by

    $ docker run --name="xcgd.odoo" -v /opt/odoo/instance1/etc:/opt/odoo/etc -p 8069:8069 --link pg93:db -d xcgd/odoo start

If you want to use a specific configuration file, you can use the environment variable ODOO_CONF:

    $ docker run -e ODOO_CONF=/etc/openerp.conf --name="xcgd.odoo" -v /opt/odoo/instance1/etc:/opt/odoo/etc -p 8069:8069 --link pg93:db -d xcgd/odoo start

By default the image does not chown the file in /opt/odoo anymore, if you need to run it, set the environment variable `ODOO_CHOWN` to `true`.

Security Notes
==============

You'll note that we did not open ports to the outside world on the PostgreSQL container. This is for security reasons, NEVER RUN your PostgreSQL container with ports open to the outside world... Just `--link` the Odoo container (single host) or use an ambassador pattern (cluster).

This is really important to understand. PostgreSQL is configured to trust everyone so better keep it firewalled. And before yelling madness please consider this: If someone gains access to your host and is able to launch a container and open a port for himself he's got your data anyways... he's on your machine. So keep that port closed and secure your host. Your database is as safe as your host is, no more.

To prevent any data corruption during an image build, we use SHA256 algorithm to check file integrity of odoo archive and python requirement packages hosted on our repository.


  [1]: https://registry.hub.docker.com/u/xcgd/postgresql/
  [2]: https://bitbucket.org/xcgd/odoo
