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
