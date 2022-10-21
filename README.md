# scrapy-redis cluster version

![PyPI](https://img.shields.io/pypi/v/scrapy-redis-sentinel)
![PyPI - License](https://img.shields.io/pypi/l/scrapy-redis-sentinel)
![GitHub last commit](https://img.shields.io/github/last-commit/crawlmap/scrapy-redis-sentinel)
![PyPI - Downloads](https://img.shields.io/pypi/dw/scrapy-redis-sentinel)

This project is based on the original project [scrapy-redis-sentinel](https://github.com/crawlaio/scrapy-redis-sentinel)

Make changes, the changes are as follows:

1. Added Redis Sentinel, there is support for 2 password connections
2. Support Python3.8+ (introduction of collection.abc)
3. Fill in the missing "dupefilter/filtered" stats in `dupefilter.py`, which is helpful for crawler progress data analysis
4. Automatically add track_id: "make request from data" and "get request from next_request"
5. Added task loss prevention: every time the last task is backed up and the crawler is started, the task will return to the top of the queue. `defaults.LATEST_QUEUE_KEY`
6. Add task scheduling using shield: `MQ_USED`


This project is based on the original project [scrapy-redis](https://github.com/rmax/scrapy-redis)

Make changes, the changes are as follows:

1. Added `Redis` sentinel connection support
2. Added `Redis` cluster connection support
3. Added `Bloomfilter` deduplication

## Install

```bash
python3 -m pip install scrapy-redis-sentinel --user
```

## Configuration example

All configurations of the original scrapy-redis are supported, priority: Sentinel mode > Cluster mode > Stand-alone mode

```python
# ----------------------------------------Bloomfilter configuration------- ------------------------------
# Number of hash functions to use, default is 6
BLOOMFILTER_HASH_NUMBER = 6

# Redis memory bit used by Bloomfilter, 30 means 2 ^ 30 u003d 128MB, default is 30 (2 ^ 22 u003d 1MB can deduplicate 130W URL)
BLOOMFILTER_BIT = 30

# Whether to enable deduplication debugging mode default is False off
DUPEFILTER_DEBUG = False

# -----------------------------------------Redis stand-alone mode------ -------------------------------
# Redis stand-alone address
REDIS_HOST = "172.25.2.25"
REDIS_PORT = 6379

# REDIS stand-alone mode configuration parameters
REDIS_PARAMS = {
  "password": "password",
  "db": 0
}

# ----------------------------------------Redis Sentinel Mode------ -------------------------------

# Redis sentinel address
REDIS_SENTINELS = [
  ('172.25.2.25', 26379),
  ('172.25.2.26', 26379),
  ('172.25.2.27', 26379)
]

# SENTINEL_KWARGS optional parameter, you can set sentinel password, refer to https://github.com/redis/redis-py/issues/1219
SENTINEL_KWARGS u003d {'password': 'sentinel_password'}

# REDIS_SENTINEL_PARAMS Sentinel mode configuration parameters.
REDIS_SENTINEL_PARAMS u003d {
  "service_name": "mymaster",
  "password": "password",
  "db": 0
}

# -------------------------------------- Redis cluster mode------ -------------------------------

# Redis cluster address
REDIS_STARTUP_NODES = [
  {"host": "172.25.2.25", "port": "6379"},
  {"host": "172.25.2.26", "port": "6379"},
  {"host": "172.25.2.27", "port": "6379"},
]

# REDIS_CLUSTER_PARAMS cluster mode configuration parameters
REDIS_CLUSTER_PARAMS u003d {
  "password": "password"
}

# -----------------------------------------Scrapy other parameters------ -------------------------------

# Keep the various queues used by scrapy-redis in redis to allow pause and resume after pause, that is, do not clean up redis queues
SCHEDULER_PERSIST = True
# dispatch queue
SCHEDULER = "mob_scrapy_redis_sentinel.scheduler.Scheduler"
# base deduplication
DUPEFILTER_CLASS = "mob_scrapy_redis_sentinel.dupefilter.RedisDupeFilter"
#BloomFilter
# DUPEFILTER_CLASS = "mob_scrapy_redis_sentinel.dupefilter.RedisBloomFilter"

# Enable Redis based statistics
STATS_CLASS = "mob_scrapy_redis_sentinel.stats.RedisStatsCollector"

# Specify the queue to use when sorting and crawling addresses
# Default sorting by priority (Scrapy default), a non-FIFO, LIFO way implemented by sorted set.
# SCHEDULER_QUEUE_CLASS = 'mob_scrapy_redis_sentinel.queue.SpiderPriorityQueue'
# optional first in first out (FIFO)
# SCHEDULER_QUEUE_CLASS = 'mob_scrapy_redis_sentinel.queue.SpiderStack'
# optional last in first out (LIFO)
# SCHEDULER_QUEUE_CLASS = 'mob_scrapy_redis_sentinel.queue.SpiderStack'
```

> Note: Single machine does not take effect when using cluster

## spiders use

**Modify the introduction method of RedisSpider**

How to use the original version of `scrapy-redis`

```python
from scrapy_redis.spiders import RedisSpider


class Spider(RedisSpider):
...

```

How to use `scrapy-redis-sentinel`

```python
from scrapy_redis_sentinel.spiders import RedisSpider


class Spider(RedisSpider):
...

```

