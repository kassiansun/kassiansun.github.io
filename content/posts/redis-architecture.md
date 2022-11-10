---
title: 'Redis Architecture'
date: 2022-11-10T15:21:10+08:00
tags:
  - redis
---

## Start-Up Procedures

- `spt_init`: This procedure initializes the process's name and deep copy command-line arguments & environments.
- `tzset`.
- `zmalloc_set_oom_handler`
- `init_genrand64` with timestamp and pid number. Code Path: `mt19937-64.c`
- `crc64_init`. Code Path: `crc64.c`
- `umask`
- `dictSetHashFunctionSeed` with a random 16-byte seed. Code Path: `dict.c`
- `initServerConfig`
- `ACLInit`. Code Path: `acl.c`
- `moduleInitModulesSystem`. Code Path: `module.c`
- `connTypeInitialize`. Code Path: `connection.c`
- `initSentinelConfig` and `initSentinel` if the server was started in sentinel mode. Code Path: `sentinel.c`
- Run `redis_check_aof_main` or `redis_check_rdb_main` and exit. Code Path: `redis-check-aof.c`, `redis-check-rdb.c`
- Parse command-line options and `loadServerConfig`. Code Path: `config.c`.
- `sentinelCheckConfigFile` if it's in sentinel mode. Code Path: `sentinel.c`
- Check whether the server is supervised by `upstart` or `systemd`.
- `daemonize` if required.
- `initServer`. It initializes the global `redisServer` structure.
- `createPidFile` if required.
- `redisSetProcTitle` if required.
- `checkTcpBacklogSettings`.
- `clusterInit` if it's in cluster mode. Code Path: `cluster.c`
- `moduleLoadFromQueue`. Code Path: `module.c`.
- `ACLLoadUsersAtStartup`, Code path: `acl.c`.
- `initListeners`. Redis supports multiple types of listeners, and each one has its own accept handling logic. The listeners will be registered by `aeCreateFileEvent`
- `clusterInitListeners` if it's in cluster mode. Code Path: `cluster.c`
- `bioInit` initializes the redis's background I/O. Code Path: `bio.c`
- `initThreadedIO` initializes the threaded I/O. Code Path: `networking.c`
- `set_jemalloc_bg_thread`.
- If it's not sentinel mode:
  - `aofLoadManifestFromDisk`. Code Path: `aof.c`
  - `loadDataFromDisk`. It will either load the AOF file with `loadAppendOnlyFiles`, or load the RDB file with `rdbLoad` and initialize the replication backlog with `createReplicationBacklog`.
    - RDB Code Path: `rdb.c`
  - `aofOpenIfNeededOnServerStart` to open the AOF file on disk. Code Path: `aof.c`
  - `aofDelHistoryFiles` to clear the `history_aof_list`. Code Path: `aof.c`
  - `verifyClusterConfigWithData` check the cluster slots. Note that cluster mode only allows data in db 0. Code Path: `cluster.c`
- Else, do `sentinelIsRunning`.
- `redisSetCpuAffinity`
- `setOOMScoreAdj` will update the value in `/proc/self/oom_score_adj`
- `aeMain` starts the event loop. `aeDeleteEventLoop` after the event loop is finished. Code Path: `ae.c`

## Event Loop

`server.el` got initialized during `initServer`. The main event loop is created, `serverCron` and `module_pipe` events are registered.
`beforeSleep` and `afterSleep` are also configured on the event loop.

## `connSocketAcceptHandler`

- Code Path: `socket.c`

`connSocketAcceptHandler` is the accept handler for the default `CT_Socket` type.

- `anetTcpAccept` returns a file descriptor for the accepted connection. Code Path: `anet.c`
- `connCreateAcceptedSocket` returns a new `connection` for the accepted file descriptor.
- `acceptCommonHandler` checks the max connection number, creates a new `client` and links it on the global client list with `createClient`,
  and executes `connSocketAccept` to fire the module events.
  - `createClient` will also register `readQueryFromClient` as the connection's read handler, and register on the event loop.

## `readQueryFromClient`

`readQueryFromClient` is the main handler to handle a pending query on a client.

## What's Next

- ACL implementation
- Event loop implementation
- AOF implementation
- Blocking operation.
- Clustering/replication/sentinel implementation
- Database operations (scan, query, etc.).
- Active memory defragmentation
- Memory eviction policies
- Key expiration implementation.
- Multi command implementation
- Pub/Sub
- Data structures implementations:
  - Hash
  - List
  - Set
  - Stream
  - String
  - ZSet
  - Hyperloglog
  - listpack and quicklist
  - Radix tree
  - Dynamic String
  - etc.
