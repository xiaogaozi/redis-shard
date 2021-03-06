Redis Shard
###########

A Redis sharding implementation.

.. image:: https://travis-ci.org/zhihu/redis-shard.svg?branch=master
   :target: https://travis-ci.org/zhihu/redis-shard
   :alt: Build Status

Redis is great. It's fast, lightweight and easy to use. But when we want to store
a mass of data into one single instance, we may encounter some problems such as performance
degradation and slow recovery and we need to scale it.

Usage
=====

First, Create an RedisShardAPI instance with multiple nodes, node ``name`` **must be unique**::

    from redis_shard.shard import RedisShardAPI
    
    servers = [
        {'name':'server1', 'host':'127.0.0.1', 'port':10000, 'db':0},
        {'name':'server2', 'host':'127.0.0.1', 'port':11000, 'db':0},
        {'name':'server3', 'host':'127.0.0.1', 'port':12000, 'db':0},
    ]
    
    client = RedisShardAPI(servers)

Then, you can access the Redis cluster as you use `redis-py <https://github.com/andymccurdy/redis-py>`_::

    client.set('test', 1)
    client.get('test')  # 1
    client.zadd('testset', 'first', 1)
    client.zadd('testset', 'second', 2)
    client.zrange('testset', 0, -1)  # [first, second]


Hash Tags
---------

If you want to store specific keys on one node for some reason (such as you prefer single instance pipeline, or
you want to use multi-keys command such as ``sinter``), you should use Hash Tags::

    client.set('foo', 2)
    client.set('a{foo}', 5)
    client.set('b{foo}', 5)
    client.set('{foo}d', 5)
    client.set('d{foo}e', 5)

    client.get_server_name('foo') == client.get_server_name('a{foo}') == client.get_server_name('{foo}d') \
            == client.get_server_name('d{foo}e')  # True

The string in a braces of a key is the Hash Tag of the key. The hash of a Hash Tag will be treated the hash of the key.
So, keys ``foo``, ``bar{foo}`` and ``b{foo}ar`` will be sotred in the same node.


Config Format
-------------

`RedisShardAPI` supports 3 kinds of config format:

- list::

    servers = [
        {'name':'node1','host':'127.0.0.1','port':10000,'db':0},
        {'name':'node2','host':'127.0.0.1','port':11000,'db':0},
        {'name':'node3','host':'127.0.0.1','port':12000,'db':0},
    ]

- dict::

    servers = {
        'node1': {'host':'127.0.0.1','port':10000,'db':0},
        'node2': {'host':'127.0.0.1','port':11000,'db':0},
        'node3': {'host':'127.0.0.1','port':12000,'db':0},
    }

- url_schema::

    servers = [
        'redis://127.0.0.1:10000/0?name=node1',
        'redis://127.0.0.1:11000/0?name=node2',
        'redis://127.0.0.1:12000/0?name=node3'
    ]


Limitations
===========

* Redis Shard dose not support all Redis commands.
* As mentioned above, Redis Shard does not support multi-keys command crossing different nodes,
  you have to use Hash Tag to work with those commands.
* Redis Shard does not have any replication mechanism.


How it Works
============

Redis Shard is basically inspired by `this article <http://oldblog.antirez.com/post/redis-presharding.html>`_.
