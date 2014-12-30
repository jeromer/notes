Notes on using ETS tables in erlang
===================================

Introduction
------------

I think I did all possible stupid mistakes with ETS tables.
So below are a few notes in order to avoid doing the same mistakes again.


Interesting links
-----------------

- Official documentation : http://www.erlang.org/doc/man/ets.html
- On the Scalability of the Erlang Term Storage : http://winsh.me/papers/erlang_workshop_2013.pdf
- More Scalable Ordered Set for ETS Using Adaptation : http://winsh.me/papers/erlang_workshop_2014.pdf
- Bears, ETS, Beets : http://learnyousomeerlang.com/ets

C implementation
----------------

ETS tables can be of type ``set``, ``ordered_set``, ``bag`` and
``duplicate_bag``.

The C implementation of ETS table is available here:

- https://github.com/erlang/otp/blob/maint/erts/emulator/beam/erl_db.c
- https://github.com/erlang/otp/blob/maint/erts/emulator/beam/erl_db_hash.c
  (for ``bag`` and ``duplicate_bag``)
- https://github.com/erlang/otp/blob/maint/erts/emulator/beam/erl_db_tree.c
  (for ``set`` and ``ordered_set``)

``set`` is implemented using `AVL trees <https://en.wikipedia.org/wiki/AVL_tree>`_

Not loosing ETS tables
----------------------

An ETS table is lost if the owner process crashes, this is explained in
http://steve.vinoski.net/blog/2011/03/23/dont-lose-your-ets-tables/.

At $DAYJOB, we decided to use a process whose only job is to create ETS tables
which avoids completely the risk of crashes.

The implementation worked so well that we extended the process to handle
almost all ETS tables in our adserver and is responsible for creating, deleting
et emptying tables if needed and we never looked back.

Best practices
--------------

Store only what is absolutely needed
++++++++++++++++++++++++++++++++++++

If you only need a few fields from a record, store only them. If you store
un-necessary data and/or too big terms (binaries for example). Not only your
memory usage will grow but also you will get lower performances. This is because
you have more data to serialize/deserialize when storing/fetching data to/from
a table. After keeping only what is necessary in our ETS table we notice a
really huger performance boost.

Prefer sets to bags
+++++++++++++++++++

If you can afford it, use tables of type ``set`` as it is much much faster.

Avoid ets:select/2 if possible
------------------------------

Using `ets:select/2 <http://www.erlang.org/doc/man/ets.html#select-2>`_
triggers a full table scan, which is obviously a lot slower than a key/value
lookup with a ``set``.
