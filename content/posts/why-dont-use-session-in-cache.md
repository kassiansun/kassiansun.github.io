---
title: "Why You Shouldn't Save Sessions in Redis Anymore"
date: 2022-03-12T17:43:32+08:00
---

### Sessions Are Important

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

### It's Hard to Manage Everything Saved in Cache

Due to the performance requirements of caching databases (Redis and many others), they won't
keep non-necessary informations of the data saved in it, which means you need to track where they're
manually. Nowadays more and more applications provide a portal for users to manage their
all sessions, so you need to track the location of these sessions anyway. And once you save these
tracking data into database, saving the other informations of sessions alongside them becomes reasonable.

### Sessions Don't Need That-High Performance

A common myth about sessions is that they will be fetched frequently (given that you need to check
whether the session is still valid on every API request), so you need to save them somewhere super-performant.
But in real-life applications, a single end-user won't generate super-high QPS, and the overall QPS of
the session query will be a decent number. As relational databases are easy to scale with replications, the
read QPS >> write QPS property of sessions will make good use of it.

### You Can Add The Caching Layer Anytime You Need It

A good design of caching layer should not assume the underlying data layer. Sometimes the
business logic isn't that ideal, so you need to design layers over layers of logic around the caching layer,
to achieve a strong consistency between caching layer and data layer. An important property of sessions is
idempotence, which means you shouldn't change the session data after you generate it. As long as you don't
need to update the session data, the consistency will be easy to achieve, all you need to make sure that
session info is invalidated properly.
