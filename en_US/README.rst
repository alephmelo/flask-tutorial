.. contents:: :local:

Introducing Flaskr
==================

We will call our blogging application flaskr, but feel free to choose your own
less Web-2.0-ish name ;)  Essentially, we want it to do the following things:

1. Let the user sign in and out with credentials specified in the
   configuration.  Only one user is supported.
2. When the user is logged in, they can add new entries to the page
   consisting of a text-only title and some HTML for the text.  This HTML
   is not sanitized because we trust the user here.
3. The index page shows all entries so far in reverse chronological order
   (newest on top) and the user can add new ones from there if logged in.

We will be using SQLite3 directly for this application because it's good
enough for an application of this size.  For larger applications, however,
it makes a lot of sense to use `SQLAlchemy`_, as it handles database
connections in a more intelligent way, allowing you to target different
relational databases at once and more.  You might also want to consider
one of the popular NoSQL databases if your data is more suited for those.

Here a screenshot of the final application:

.. image:: ../static/flaskr.png
   :align: center
   :class: screenshot
   :alt: screenshot of the final application

.. _SQLAlchemy: http://www.sqlalchemy.org/

Creating The Folders
============================

Before we get started, let's create the folders needed for this
application::

    /flaskr
        /static
        /templates

The ``flaskr`` folder is not a Python package, but just something where we
drop our files. Later on, we will put our database schema as well as main
module into this folder. It is done in the following way. The files inside
the *static* folder are available to users of the application via HTTP.
This is the place where CSS and Javascript files go.  Inside the
*templates* folder, Flask will look for `Jinja2`_ templates.  The
templates you create later on in the tutorial will go in this directory.

.. _Jinja2: http://jinja.pocoo.org/

Database Schema
=======================

First, we want to create the database schema. Only a single table is needed
for this application and we only want to support SQLite, so creating the
database schema is quite easy. Just put the following contents into a file
named *schema.sql* in the just created *flaskr* folder:

.. sourcecode:: sql

    drop table if exists entries;
    create table entries (
      id integer primary key autoincrement,
      title text not null,
      'text' text not null
    );

This schema consists of a single table called ``entries``. Each row in
this table has an ``id``, a ``title``, and a ``text``.  The ``id`` is an
automatically incrementing integer and a primary key, the other two are
strings that must not be null.

Application Setup Code
==============================

Now that we have the schema in place, we can create the application module.
Let's call it ``flaskr.py``. We will place this file inside the ``flaskr``
folder. We will begin by adding the imports we need and by adding the config
section.  For small applications, it is possible to drop the configuration
directly into the module, and this is what we will be doing here. However,
a cleaner solution would be to create a separate ``.ini`` or ``.py`` file,
load that, and import the values from there.

First, we add the imports in *flaskr.py*:

.. sourcecode:: python

    # all the imports
    import os
    import sqlite3
    from flask import Flask, request, session, g, redirect, url_for, abort, \
         render_template, flash

Next, we can create our actual application and initialize it with the
config from the same file in *flaskr.py*:

.. sourcecode:: python

    # create our little application :)
    app = Flask(__name__)
    app.config.from_object(__name__)

    # Load default config and override config from an environment variable
    app.config.update(dict(
        DATABASE=os.path.join(app.root_path, 'flaskr.db'),
        SECRET_KEY='development key',
        USERNAME='admin',
        PASSWORD='default'
    ))
    app.config.from_envvar('FLASKR_SETTINGS', silent=True)

The *flask.Config* object works similarly to a dictionary so we
can update it with new values.

**Database Path**

    Operating systems know the concept of a current working directory for
    each process.  Unfortunately, you cannot depend on this in web
    applications because you might have more than one application in the
    same process.

    For this reason the ``app.root_path`` attribute can be used to
    get the path to the application.  Together with the ``os.path`` module,
    files can then easily be found.  In this example, we place the
    database right next to it.

    For a real-world application, it's recommended to use
    ``instance-folders`` instead.

Usually, it is a good idea to load a separate, environment-specific
configuration file.  Flask allows you to import multiple configurations and it
will use the setting defined in the last import. This enables robust
configuration setups. ``flask.Config.from_envvar`` can help achieve this.

.. code-block:: python

   app.config.from_envvar('FLASKR_SETTINGS', silent=True)

Simply define the environment variable ``FLASKR_SETTINGS`` that points to
a config file to be loaded.  The silent switch just tells Flask to not complain
if no such environment key is set.

In addition to that, you can use the ``flask.Config.from_object``
method on the config object and provide it with an import name of a
module.  Flask will then initialize the variable from that module.  Note
that in all cases, only variable names that are uppercase are considered.

The ``SECRET_KEY`` is needed to keep the client-side sessions secure.
Choose that key wisely and as hard to guess and complex as possible.

We will also add a method that allows for easy connections to the
specified database. This can be used to open a connection on request and
also from the interactive Python shell or a script.  This will come in
handy later.  We create a simple database connection through SQLite and
then tell it to use the ``sqlite3.Row`` object to represent rows.
This allows us to treat the rows as if they were dictionaries instead of
tuples.

.. sourcecode:: python

    def connect_db():
        """Connects to the specific database."""
        rv = sqlite3.connect(app.config['DATABASE'])
        rv.row_factory = sqlite3.Row
        return rv

With that out of the way, you should be able to start up the application
without problems.  Do this with the following command:

    flask --app=flaskr --debug run

The ``--debug`` flag enables or disables the interactive debugger.  *Never
leave debug mode activated in a production system*, because it will allow
users to execute code on the server!

You will see a message telling you that server has started along with
the address at which you can access it.

When you head over to the server in your browser, you will get a 404 error
because we don't have any views yet.  We will focus on that a little later,
but first, we should get the database working.