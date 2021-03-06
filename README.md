Introduction
============

smeuhsocial is the app that powers [My.Smeuh](https://my.smeuh.org), the
community platform of [Smeuh.Org](http://smeuh.org).  It's based on
[Pinax](http://pinaxproject.com) which is itself based on
[Django](http://djangoproject.com).  This document is written in
[Markdown](http://daringfireball.net/projects/markdown/syntax).

Instructions presented here are incomplete and poorly tested so don't hesitate to
ask for help if you run into trouble.

Developement Setup
==================


Requirements
------------

 - Python version 2.5.x, because this is the version we have on smeuh's server.
   Using the same version for development will prevent you from writing
   incompatible code. If you don't have this version currently installed on
   your system, the easiest and safest way to get version 2.5 is probably to
   install it from source in your home directory.
 - [virtualenv](http://pypi.python.org/pypi/virtualenv). This is used to create
   an isolated Python environement containing the dependencies necessary to run
   the project, without affecting your global Python installation. Make sure you
   have a version of virtualenv working against your version of Python 2.5.x
 - Development packages for Python, LDAP, SASL and SSL. For Debian-like systems:

    $ sudo apt-get install python-dev libldap-dev libsasl2-dev libssl-dev

 - [OpenLDAP](http://www.openldap.org/) server. OpenLDAP server is necessary
   only if you want to work on OpenLDAP authentication. You might get into
   trouble with a pre-packaged version so just grab the official source tarball
   and install it following instructions at
   <http://www.openldap.org/doc/admin24/quickstart.html>. See section "Setup
   LDAP directory" below for more info on this.



Installation
------------

Fork the git repository https://github.com/amarandon/smeuhsocial and clone
it with:

    $ git clone git@github.com:<USERNAME>/smeuhsocial.git

Create your virtual environment:

    $ virtualenv-2.5 smeuhsocial_env

(Note that the virtual environement can be located anywhere on your filesystem.
It does not need to contain the source code of the app.)

Activate your virtual environement:

    $ . ./smeuhsocial_env/bin/activate

Switch to the directory containing the source code:

    $ cd smeuhsocial

Install required Python packages dependancies:

    $ pip install -r requirements/project.txt

Create a file local_settings.py with this content:

    MEDIA_URL = "/site_media/"

(This is because static media location is a bit odd on the live server right now)


Initialize the database
-----------------------

Default database backend is sqlite that will store data in db/dev.db

If the "db" sub directory does not exist, create it:

    $ mkdir db

Then load the schema:

    $ python manage.py syncdb


Start development server
------------------------

    $ python manage.py runserver

and point your browser at <http://127.0.0.1:8000/>


Setup LDAP directory
--------------------

I had a lot of trouble trying to get smeuh's directory working with the
pre-packaged OpenLDAP from Ubuntu, so I ended up installing from source.  This
procedure worked for me:

* ask smeuh admins for a dump of smeuh ldap db (or al for another version)
* install openldap from source
* edit slapd.conf
  - make sure these schema are included:
        include     /usr/local/etc/openldap/schema/core.schema
        include     /usr/local/etc/openldap/schema/cosine.schema
        include     /usr/local/etc/openldap/schema/inetorgperson.schema
        include     /usr/local/etc/openldap/schema/nis.schema
        include     /usr/local/etc/openldap/schema/misc.schema
  - and configure database like this:
        database    bdb
        suffix      "o=loc"
        rootdn      "cn=admin,o=loc"
        rootpw      aBkdo38u3
        directory   /usr/local/var/openldap-data
        index   objectClass eq
* the raw dump as provided by smeuh's admin contains entries which prevent from
  restoring it in your own OpenLDAP installation. We provide a sed script
  (scripts/cleanup_ldap_dump.sed) to remove unwanted entries from the dump. It
  can be used like this:

    $ sed -f scripts/cleanup_ldap_dump.sed < ldap-dump-from-admin.ldif > clean-dump-clean.ldif

* import the data with:

    $ ldapadd -x -D "cn=admin,o=loc" -W -f ldap-dump-clean.ldif

* then finally run the slapd server with a command such as:

    $ sudo /usr/local/libexec/slapd

you should now be able to log into the app with your smeuh credentials.

If you need to edit the content of your ldap database, you can use ldapvi with a
command such as:

    $ ldapvi -D 'cn=admin,o=loc'  --discover

To add an entry, you can copy an existing entry, change the attributes you wish
to change, replace the sequence number at the beginning of its first line by
the word 'add'. To delete an entry, just remove it.


Hacking Pinax
-------------

If you've followed the setup presented here, you should have a clone of Pinax
Git repository in smeuhsocial_env/src/pinax. You can hack in there and
potentially send patches or you could fork Pinax code and change
requirements/project.txt to point to your own Pinax repository by changing this
line:

    -e git+ssh://git@github.com/amarandon/pinax.git#egg=Pinax

to point to your own fork of Pinax.
