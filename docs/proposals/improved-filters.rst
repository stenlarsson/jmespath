==================
Filter Expressions
==================

:JEP: 9
:Author: James Saryerwinnie
:Status: proposed
:Created: 07-July-2014


Abstract
========

JEP 7 introduced filter expressions, which is a mechanism to allow
list elements to be selected based on matching an expression against
each list element.  While this concept is useful, the actual comparator
expressions were not sufficiently capable to accomodate a number of common
queries.  This JEP expands on filter expressions by proposing support for
``and-expressions``, ``not-expression``, ``paren-expressions``, and
``unary-expressions``.  With these additions, the capabilities of a filter
expression now allow for sufficiently powerful queries to handle the majority
of queries.


Motivation
==========

JEP 7 introduced filter queries, that essentially look like this::

    foo[?lhs omparator rhs]

where the left hand side (lhs)  and the right hand side (rhs)
are both an ``expression``, and comparator is one of
``==, !=, <, <=, >, >=``.

This added a useful feature to JMESPath: the ability to filter
a list based on evaluating an expression against each element in a list.

In the time since JEP 7 has been part of JMESPath, a number of cases have been
pointed out in which filter expressions cannot solve.  Below are examples of
each type of missing features.


Or Expressions
--------------

First, users want the ability to filter based on matching one or more
expressions.  For example, given::

    {
      "cities": [
        {"name": "Seattle", "state": "WA"},
        {"name": "Los Angeles", "state": "CA"},
        {"name": "Bellevue", "state": "WA"},
        {"name": "New York", "state": "NY"},
        {"name": "San Antonio", "state": "TX"},
        {"name": "Portland", "state": "OR"}
      ]
    }

a user might want to select locations on the west coast, which in
this specific example means cities in either ``WA``, ``OR``, or
``CA``.  It's not possible to express this as a filter expression
given the grammar of ``expression comparator expression``.  Ideally
a user should be able to use::

    cities[?state == `WA` || state == `OR` || state == `CA`]

JMESPath already supports Or expressions, just not in the context
of filter expressions.

And Expressions
---------------

The next missing feature of filter expressions is support for And
expressions.  It's actually somewhat odd that JMESPath has support
for Or expressions, but not for And expressions.  For example,
given a list of user accounts with permissions::

    {
      "users": [
        {"name": "user1", "type": "normal"", "allowed_hosts": ["a", "b"]},
        {"name": "user2", "type": "admin", "allowed_hosts": ["a", "b"]},
        {"name": "user3", "type": "normal", "allowed_hosts": ["c", "d"]},
        {"name": "user4", "type": "admin", "allowed_hosts": ["c", "d"]},
        {"name": "user5", "type": "normal", "allowed_hosts": ["c", "d"]},
        {"name": "user6", "type": "normal", "allowed_hosts": ["c", "d"]}
      ]
    }

We'd like to find admin users that have permissions to the host named
``c``.  Ideally, the filter expression would be::

    users[?type == `admin` && contains(allowed_hosts, `c`)]


Unary Expressions
-----------------

Think of an if statement in a language such as C or Java.  While you can write
an if statement that looks like::

    if (foo == bar) { ... }

You can also use a unary expression such as::

    if (allowed_access) { ... }

or::

    if (!allowed_access) { ... }

Adding support for unary expressions brings a natural syntax when filtering
against boolean values.  Instead of::

    foo[?boolean_var == `true`]

a user could instead use::

    foo[?boolean_var]

As a more realistic example, given a slightly different structure
for the ``users`` data above::

    {
      "users": [
        {"name": "user1", "is_admin": false, "disabled": false},
        {"name": "user2", "is_admin": true, "disabled": true},
        {"name": "user3", "is_admin": false, "disabled": false},
        {"name": "user4", "is_admin": true, "disabled": false},
        {"name": "user5", "is_admin": false, "disabled": true},
        {"name": "user6", "is_admin": false, "disabled": false}
      ]
    }

If we want to get the names of all admin users whose account is enabled, we
could either say::

    users[?is_admin == `true` && disabled == `false]

but it's more natural and succinct to instead say::

    users[?is_admin && !disabled]

A case can be made that this syntax is not strictly necessary.  This is true.
However, the main reason for adding support for unary expressions in a filter
expression is users expect this syntax, and are surprised when this is not
a supported syntax.  Especially now that we are basically anchoring to
a C-like syntax for filtering in this JEP, users will expect unary expressions
even more.


Specification
=============

There are several updates to the grammar::

    and-expression         = expression "&&" expression
    not-expression         = "!" expression
    paren-expression       = "(" expression ")"


Additionally, the ``filter-expression`` rule is updated
to be more general::

    bracket-specifier      =/ "[?" expression "]"

And the ``list-filter-expression`` is now a more general
``comparator-expression``::

    comparator-expression  = expression comparator expression



Rationale
=========

