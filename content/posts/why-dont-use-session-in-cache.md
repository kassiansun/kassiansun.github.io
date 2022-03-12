---
title: "Why You Shouldn't Save Sessions in Redis Anymore"
date: 2022-03-12T17:43:32+08:00
---

### Sessions are important

Security is an important factor of a reliable system, and from a user's perspective,
sessions are where the secure process starts. Sessions are no longer a volatile string saved
in Redis/memcached, but a key to everything owned by users. Expiration time, login IP, login
device, and many other properties of sessions should be tracked and managed properly.

The importance of sessions makes the cache an improper choice: the nature of caching means you shouldn't
put anything important in it. Saving a
business-related data in cache will make the cache a critical path in your business logic,
make the logic fragile and increase the SLA number of cache systems, then make it more expensive
to operate the caching system.

Also, anything saved in the cache should be minimal, save a full-size session record in the cache, (and
there'll be more and more info we want to track along with the session), will reduce the performance
and reliability of the cache.

### It's hard to manage anything saved in cache

Due to the performance requirements of caching databases (Redis and many others), they won't
keep non-necessary informations of the data saved in it, which means you need to track where they're
manually. Nowadays more and more applications provide a portal for users to manage their
all sessions, then you need to track the locations of session anyway. And once you save these
tracking data into database, saving the other informations of sessions alongside them becomes reasonable.

### Sessions don't need that-high performance

A common myth about sessions is that they will be fetched frequently (given that you need to check
whether the session is still valid on every API request), so you should save them somewhere super-performant.
But in real-life applications, a single end-user won't generate super-high QPS, and the overall QPS of
the session query will be a decent number. As relational databases are easy to scale with replications, the
`(read QPS) >> (write QPS)` property of sessions will make good use of it.

### You can add the caching layer back if you really need It

A good design of caching layer should not assume the underlying data layer. Sometimes the
business logic isn't that ideal, and you need to design layers over layers of logic around the caching layer,
to achieve a strong consistency between caching layer and data layer. An important property of sessions is
idempotence, which means you shouldn't change the session data after you generate it. As long as you don't
need to update the session data, the consistency will be easy to achieve, all you need is to make sure the
sessions are invalidated properly.
