# Redis — In-Memory Data Store

## Table of Contents

- What is Redis?
- Data Structures
- Where Does Redis Data Live?
- Why Redis is Fast
- Network & Latency
- Cache Attack Defence
- Quick Start


# What is Redis?

Redis is an in-memory data store — it keeps all data in RAM instead of on disk. This makes operations blazing fast (microseconds vs milliseconds for a traditional DB). It is primarily used as a cache, session store, queue, message broker, and real-time analytics engine — usually alongside a primary database like PostgreSQL, not instead of it.

Think of it this way: your PostgreSQL database is like a library — it stores everything permanently, but fetching a book takes time. Redis is like your desk — the books you use all the time are right there, instantly accessible.


## The Core Problem Redis Solves

Your app goes viral. 10,000 users hit your server. Every request triggers a DB query. 9,000 of those requests are asking for the exact same data — the top products, the homepage feed, the user profile. You're querying the database 9,000 times for data that hasn't changed in 5 seconds.


## Common Use Cases

Store expensive DB query results. Set a TTL — Redis auto-expires them. Database gets a fraction of the traffic.

Store sessionId → user data in Redis. Fast auth on every request. Works across multiple server instances.

Track request counts per IP using INCR + EXPIRE . Block abusive clients without touching your DB.

Sorted Sets let you maintain a real-time ranked leaderboard with atomic score updates. No complex SQL needed.

Push jobs to a Redis list. Worker processes pop and execute them. Simple, fast background processing.

Publish events to channels. Subscribers receive them instantly. Real-time notifications, live feeds, chat systems.

Redis vs Database: Redis is not a replacement for PostgreSQL. Redis is your fast layer for hot, frequently-accessed data. PostgreSQL is your source of truth. They work together — Redis in front, Postgres behind.


# Data Structures


## TTL — Time To Live

Every Redis key can have an expiry. Redis automatically deletes the key after the TTL expires. This is how you prevent stale cache and manage memory.


```javascript

// Set with TTL (seconds)
await redis.set('user:42', JSON.stringify(userData), 'EX', 3600);
// ↑ expires in 1 hour automatically

// Get
const cached = await redis.get('user:42');
const user = cached ? JSON.parse(cached) : await db.findUser(42);

// Counter with expiry (rate limiting pattern)
await redis.incr(`requests:${ip}`);
await redis.expire(`requests:${ip}`, 60); // reset every 60s

```


# Where Does Redis Data Live?

Redis always uses the RAM of the machine it runs on. The question is — which machine? If you host Redis yourself, it's your server's RAM. If you use a cloud Redis service, it's their server's RAM, and you talk to it over a network.

Cloud Redis isn't "internet Redis". When your app and Redis are in the same AWS region/VPC, traffic travels through private AWS networking — not the public internet. Think of it as a super-fast computer sitting next door in the same datacenter, not some random server across the world.

Always colocate. Never put your app server in Mumbai and your Redis in Virginia. Cross-continent Redis completely defeats the purpose — you'll get 100–250ms latency on every cache hit.


# Why Redis is Extremely Fast


## RAM vs Disk — The Fundamental Difference

Redis itself is almost infinitely fast — it executes operations in microseconds. The actual bottleneck in a cloud setup is network latency , not Redis. So the real engineering question is: how do you minimize round trips to Redis?


## Why Single-Threaded Isn't a Problem

Redis uses a single-threaded event loop. You might think "isn't that slow?" — but it's actually a design advantage. No mutex locks, no thread contention, no context switching. Redis processes commands in a queue, one by one, each in microseconds. At 100,000+ operations per second, this is not the bottleneck.

Redis 6+ added I/O threading — network reads/writes happen on multiple threads, but command execution stays single-threaded. Best of both worlds.


# Network & Latency

The total time for a Redis operation = Redis execution time + network round trip . Redis execution is microseconds. The network is where the variance comes in.


## Layers of Network Latency


## How to Reduce Round Trips


## Connection Pooling

Never create a new Redis connection per request. Create one shared connection (or pool) when your app starts and reuse it. ioredis handles this automatically — one import, one client, used everywhere.


```javascript

const user    = await redis.get('user:42');     // round trip 1
const profile = await redis.get('profile:42');  // round trip 2
const posts   = await redis.get('posts:42');    // round trip 3

```


```javascript

const [user, profile, posts] = await redis.mget(
  'user:42',
  'profile:42',
  'posts:42'
); // single round trip — 3× faster

```


```javascript

const pipeline = redis.pipeline();
pipeline.get('user:42');
pipeline.incr('visits:42');
pipeline.expire('visits:42', 3600);
const results = await pipeline.exec(); // all 3 in one trip

```


```javascript

import Redis from 'ioredis';

const redis = new Redis({
  host: process.env.REDIS_HOST,
  port: 6379,
  keepAlive: 30000,       // keep TCP connection alive
  maxRetriesPerRequest: 3
});

export default redis; // import this everywhere — one shared connection

```


# Cache Attack Defence

Cache penetration is when attackers intentionally request keys that don't exist — /user/999999999 where no such user exists. Redis misses, so every request falls through to the database. At scale, this overwhelms your DB. The cache layer is bypassed entirely.


## Defence 1 — Negative Caching

When a key genuinely doesn't exist in the DB, cache that "not found" result in Redis with a short TTL. Future requests for the same invalid key hit Redis (fast miss) instead of the DB (expensive miss).


## Defence 2 — Bloom Filters

A Bloom Filter is a probabilistic data structure. Before hitting Redis or the DB, ask the Bloom Filter: "could this key exist?" If it says NO, the key definitely doesn't exist — return immediately. If it says YES, it might exist — proceed normally. False positives are possible, false negatives are not.


## Defence 3 — Cache Stampede Prevention

When a popular cached key expires, thousands of requests simultaneously miss the cache and all try to query the DB at once. This is a cache stampede (also called thundering herd).

Solution — Mutex/Lock: When a cache miss happens, one request acquires a Redis lock and queries the DB. All other requests wait briefly and re-check the cache. Only one DB query happens.


## Defence Summary


```javascript

async function getUser(id) {
  const cached = await redis.get(`user:${id}`);
  if (cached === 'NULL') return null; // cached miss — don't hit DB
  if (cached) return JSON.parse(cached);

  const user = await db.findUser(id);

  if (!user) {
    await redis.set(`user:${id}`, 'NULL', 'EX', 60); // cache the miss
    return null;
  }

  await redis.set(`user:${id}`, JSON.stringify(user), 'EX', 3600);
  return user;
}

```


```javascript

async function getWithLock(key, fetchFn) {
  let cached = await redis.get(key);
  if (cached) return JSON.parse(cached);

  const lockKey = `lock:${key}`;
  const locked = await redis.set(lockKey, 1, 'NX', 'EX', 5);

  if (locked) {
    const data = await fetchFn(); // only this request hits DB
    await redis.set(key, JSON.stringify(data), 'EX', 3600);
    await redis.del(lockKey);
    return data;
  }

  // Others wait 100ms and retry from cache
  await new Promise(r => setTimeout(r, 100));
  return JSON.parse(await redis.get(key));
}

```


# Quick Start

Never use KEYS * in production. It blocks Redis while it scans every key. Use SCAN instead for production key inspection.


```javascript

# Using Docker (recommended)
docker run --name myredis -p 6379:6379 -d redis

# Or install locally (Ubuntu/Debian)
sudo apt install redis-server
redis-server

```


```javascript

npm install ioredis

```


```javascript

import express from 'express';
import Redis from 'ioredis';

const app   = express();
const redis = new Redis();

// Cache middleware
const cache = (ttl) => async (req, res, next) => {
  const key    = `cache:${req.originalUrl}`;
  const cached = await redis.get(key);
  if (cached) return res.json(JSON.parse(cached));
  res.sendResponse = res.json.bind(res);
  res.json = (data) => {
    redis.set(key, JSON.stringify(data), 'EX', ttl);
    res.sendResponse(data);
  };
  next();
};

// Apply to routes
app.get('/products', cache(3600), async (req, res) => {
  const products = await db.getProducts(); // only runs on cache miss
  res.json(products);
});

```


```javascript

redis-cli
# Inside the CLI:
PING                          # should return PONG
SET test "hello" EX 60
GET test                        # returns "hello"
TTL test                        # returns seconds remaining
KEYS *                          # list all keys (dev only!)

```
