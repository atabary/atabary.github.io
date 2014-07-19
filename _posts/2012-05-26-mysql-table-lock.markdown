---
layout: post
title: "MySQL table lock with Django"
date: 2012-05-26 17:30:00
---

When operating with a relational database like MySQL or PostgreSQL it is sometimes required to use table locks, usually when performing a transaction susceptible to concurrency problems.

As a rule of thumb:

* a WRITE LOCK on a table is needed when writing to that table while performing a transaction susceptible to concurrency issues,
* a READ LOCK on a table is needed when reading from that table while performing a transaction susceptible to concurrency issues,
* when a lock is acquired, all the tables used in the transaction must be locked,
* all locks must be released when a database transaction is completed.

Django's ORM doesn't have support for table locks, which is quite understandable as table locking is database specific.

I wrote a small context manager that can be used to lock tables with MySQL:

{% highlight python %}
import contextlib

from django.db import connection


@contextlib.contextmanager
def acquire_table_lock(read, write):
    '''Acquire read & write locks on tables.

    Usage example:
    from polls.models import Poll, Choice
    with acquire_table_lock(read=[Poll], write=[Choice]):
        pass
    '''
    cursor = lock_table(read, write)
    try:
        yield cursor
    finally:
        unlock_table(cursor)


def lock_table(read, write):
    '''Acquire read & write locks on tables.'''
    # MySQL
    if connection.settings_dict['ENGINE'] == 'django.db.backends.mysql':
        # Get the actual table names
        write_tables = [model._meta.db_table for model in write]
        read_tables = [model._meta.db_table for model in read]
        # Statements
        write_statement = ', '.join(['%s WRITE' % table for table in write_tables])
        read_statement = ', '.join(['%s READ' % table for table in read_tables])
        statement = 'LOCK TABLES %s' % ', '.join([write_statement, read_statement])
        # Acquire the lock
        cursor = connection.cursor()
        cursor.execute(statement)
        return cursor
    # Other databases: not supported
    else:
        raise Exception('This backend is not supported: %s' %
                        connection.settings_dict['ENGINE'])


def unlock_table(cursor):
    '''Release all acquired locks.'''
    # MySQL
    if connection.settings_dict['ENGINE'] == 'django.db.backends.mysql':
        cursor.execute("UNLOCK TABLES")
    # Other databases: not supported
    else:
        raise Exception('This backend is not supported: %s' %
                        connection.settings_dict['ENGINE'])

{% endhighlight %}

It works with the models declared in your django application, by simply providing two lists:

* the list of models to lock for read purposes, and
* the list of models to lock for write purposes.

For instance, using django tutorial's models, you would just call the context manager like this:

{% highlight python %}
with acquire_table_lock(read=[models.Poll], write=[models.Choice]):
    pass  # Do something here
{% endhighlight %}
