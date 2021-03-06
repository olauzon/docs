.. Project-FiFo documentation master file, created by
   Heinz N. Gies on Fri Aug 15 03:25:49 2014.

*************
Configuration
*************

Configuration file
##################

`Wiggle's <../wiggle.html>`_ configuration file is located in `/opt/local/fifo-wiggle/etc/wiggle.conf`. It is automatically generated on the first install and will not be overwritten on updates. None the less the newest version of the file is always located in `/opt/local/fifo-wiggle/etc/wiggle.conf.example`.

The configuration file is documented in-line but we'll go over go over some more interesting settings here.

``listening_ip``
The TCP IP that mdns messages arrive to, this needs to be set where wiggle can reach sniffle, snarl and howl.

HTTP(S)
*******

``compression``
    Enable or disable compression for http, either ``on`` or ``off``.

``port``
    The port that wiggle listens for HTTP traffic.

``ssl``
    Enable or disable SSL, must be either ``on`` or ``off``.

``ssl.port``
    The port that wiggle listens for HTTPS traffic.

``ssl.cacertfile``
    The SSL CA certificate, this is auto-generated on the installation.

``ssl.certfile``
    The SSL Server certificate, this is auto-generated on the installation.

``ssl.keyfile``
    The SSL Key file, this is auto-generated on the installation.

``acceptors``
    Number of acceptor processes that are kept ready.

Caching
*******

.. attention::

   Caching done by `Wiggle <../wiggle.html>`_ does not affect the other services, requests in `Sniffle <../sniffle.html>`_ / `Snarl <../snarl.html>`_ are always handled with live data! Requests that modify data will never be cached!

`Wiggle <../wiggle.html>`_ has a build in cache. This cache will cache requests that otherwise need to be handled by `Sniffle <../sniffle.html>`_ or `Snarl <../snarl.html>`_ - adding latency and computation time to the request.

However as with most things caching has a trade-off: in exchange for much faster responses the liveliness of data suffers. This means that it is possible that a request will provide stale data. `Wiggle <../wiggle.html>`_ itself tries to invalidate cached elements whenever possible but when more then one `Wiggle <../wiggle.html>`_ service is running it is no longer possible to do this over multiple nodes.

On the other hand most of *FiFo's* data is immutable making it easy to cache since invalidation is only necessary on delete. To take optimal advantage of this fact `Wiggle <../wiggle.html>`_ lets the user configure the cache lifetime on a per datafile level.

When used the cache works in a two tiered model where each tier is defined by a TTL. If the level 1 TTL is hit the cached object is returned without questioning its correctness. If the level 2 TTL (which should be larger then the level 1 TTL) is hit the object is considered suspect but not necessarily wrong - `Wiggle <../wiggle.html>`_ will return the object for this request but then invalidate it and asynchronously update its cache so that further requests get a fresh element.

The two tier model allows short L1 TTLs and longer L2 TTLs making for a good trade-off between liveliness and performance. Nonetheless by **default the cache is disabled** to guarantee no unexpected behavior!

``caching``
    Enables or disables the cache in `Wiggle <../wiggle.html>`_. When turned ``off`` only tokens (to authenticate) are cached with a very short TTL to speed up quick successive requests as they are a very common pattern.

``ttl.element.*.l1``
    Level 1 time to live for an element. When the element is requested again before the L1 TTL is met it is directly served from the cache. The defaults for the L1 TTL are rather short and reflect the chance of which the object might change (i.e. ``packages`` have a much higher TTL then ``vms``).

``ttl.element.*.l2``
   Level 2 time to live for an element. When this element is requested again before the L2 TTL is met the cached value is returned but then invalidated and asynchronously updated. The defaults for the L2 TTL are somewhat high since even when they provide stale data this data is only send once and a simple recheck will provide new data.

``ttl.list.*.l1``
    Analog to ``ttl.element.*.l1`` but for list requests. Since they are much harder to cache and the chance of changes for 'all' elements is much higher then for a single one this TTL is shorter then it's element counterpart.

``ttl.list.*.l2``
    Analog to ``ttl.element.*.l2`` but for list requests. Since they are much harder to cache and the chance of changes for 'all' elements is much higher then for a single one this TTL is shorter then it's element counterpart.
