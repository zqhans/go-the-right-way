---
title: "Caching"
weight: 120
bookFlatSection: true
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

# Caching

**Notice:** generated with AI (google-labs-jules)

Caching is a vital technique for improving application performance and reducing latency by storing frequently accessed data in a way that allows for faster retrieval. Go applications can leverage various caching strategies, from simple in-memory caches to distributed caching systems.

Go itself is a compiled language, so there's no direct equivalent to "opcode caches" found in interpreted languages like PHP. This section focuses on application-level data caching.

## In-Memory Caching

In-memory caching involves storing data directly within your application's memory space. This is the fastest form of caching but has limitations:
- **Limited Capacity:** The cache size is restricted by the available memory of the application instance.
- **Single Instance:** The cache is local to each application instance. If you have multiple instances of your application running, each will have its own separate cache, which can lead to inconsistencies or redundant data fetching unless a more sophisticated approach is used.
- **Data Loss on Restart:** The cache is volatile and will be lost if the application restarts.

**Common Approaches for In-Memory Caching:**

1.  **Simple Map with Mutex:**
    For basic caching needs, a `map` protected by a `sync.RWMutex` can be used.

    ```go
    import (
    	"sync"
    	"time"
    )

    type CacheItem struct {
    	Value      interface{}
    	Expiration int64
    }

    type InMemoryCache struct {
    	mu    sync.RWMutex
    	items map[string]CacheItem
    }

    func NewInMemoryCache() *InMemoryCache {
    	return &InMemoryCache{
    		items: make(map[string]CacheItem),
    	}
    }

    func (c *InMemoryCache) Set(key string, value interface{}, duration time.Duration) {
    	c.mu.Lock()
    	defer c.mu.Unlock()
    	c.items[key] = CacheItem{
    		Value:      value,
    		Expiration: time.Now().Add(duration).UnixNano(),
    	}
    }

    func (c *InMemoryCache) Get(key string) (interface{}, bool) {
    	c.mu.RLock()
    	defer c.mu.RUnlock()
    	item, found := c.items[key]
    	if !found {
    		return nil, false
    	}
    	if time.Now().UnixNano() > item.Expiration {
    		// Item has expired, can optionally delete it here
    		// delete(c.items, key) // Requires upgrading to a Write Lock or handling deletion separately
    		return nil, false
    	}
    	return item.Value, true
    }

    // Example Usage:
    // cache := NewInMemoryCache()
    // cache.Set("mykey", "myvalue", 5*time.Minute)
    // val, found := cache.Get("mykey")
    ```

2.  **Third-Party Libraries:**
    Several libraries provide more sophisticated in-memory caching solutions with features like:
    - Size limits (LRU, LFU eviction policies)
    - Automatic expiration
    - Concurrent access safety

    Popular libraries:
    - **[patrickmn/go-cache](https://github.com/patrickmn/go-cache):** A widely used in-memory cache library, similar to the example above but more feature-rich.
    - **[allegro/bigcache](https://github.com/allegro/bigcache):** Optimized for high-volume, concurrent access and large datasets, focusing on minimizing GC overhead.
    - **[hashicorp/golang-lru](https://github.com/hashicorp/golang-lru):** Provides LRU (Least Recently Used) and ARC (Adaptive Replacement Cache) implementations.

## Distributed Caching

For applications running on multiple servers or requiring a shared cache with greater durability and scalability, distributed caching systems are preferred. These systems run as separate services.

**Common Distributed Cache Systems:**

-   **Redis ([redis.io](https://redis.io/)):**
    An extremely popular open-source, in-memory data structure store, used as a database, cache, and message broker. Redis offers various data structures (strings, hashes, lists, sets, sorted sets), persistence options, and high availability features.
    - **Go Clients for Redis:**
        - **[go-redis/redis](https://github.com/go-redis/redis):** A widely used, full-featured Redis client for Go.
        - **[gomodule/redigo](https://github.com/gomodule/redigo):** Another popular Go client for Redis.

    ```go
    // Example with go-redis/redis
    // import "github.com/go-redis/redis/v8"
    // import "context"
    // var ctx = context.Background()

    // rdb := redis.NewClient(&redis.Options{ Addr: "localhost:6379", })
    // err := rdb.Set(ctx, "mykey", "myvalue", 10*time.Minute).Err()
    // val, err := rdb.Get(ctx, "mykey").Result()
    ```

-   **Memcached ([memcached.org](https://memcached.org/)):**
    A high-performance, distributed memory object caching system. It's simpler than Redis, primarily offering a key-value store (strings).
    - **Go Clients for Memcached:**
        - **[bradfitz/gomemcache](https://github.com/bradfitz/gomemcache):** A commonly used Memcached client for Go.

    ```go
    // Example with bradfitz/gomemcache
    // import "github.com/bradfitz/gomemcache/memcache"

    // mc := memcache.New("localhost:11211")
    // mc.Set(&memcache.Item{Key: "mykey", Value: []byte("myvalue"), Expiration: 600})
    // item, err := mc.Get("mykey")
    ```

**Benefits of Distributed Caching:**
- **Shared Cache:** Accessible by multiple application instances.
- **Scalability:** Can be scaled independently of the application.
- **Durability (for some systems like Redis):** Can offer data persistence options.
- **Reduced Load on Primary Datastores:** Offloads read traffic from databases.

## Cache Eviction Policies

When a cache reaches its capacity, it needs to decide which items to remove to make space for new ones. Common eviction policies include:
- **LRU (Least Recently Used):** Discards the least recently used items first.
- **LFU (Least Frequently Used):** Discards the least frequently used items first.
- **FIFO (First-In, First-Out):** Discards the oldest items first.
- **ARC (Adaptive Replacement Cache):** More complex, aims to improve upon LRU.

The choice of eviction policy depends on the access patterns of your data.

## Cache Patterns

- **Cache-Aside (Lazy Loading):**
    1. Application requests data.
    2. Check if data is in the cache.
    3. If yes, return data from cache.
    4. If no, fetch data from the datastore, store it in the cache, and then return it.
- **Read-Through:** The cache itself is responsible for fetching data from the datastore if it's not found in the cache. The application always talks to the cache.
- **Write-Through:** Data is written to the cache and the datastore simultaneously. Ensures consistency but can have higher latency on writes.
- **Write-Back (Write-Behind):** Data is written to the cache first. The cache then asynchronously writes the data to the datastore after a delay. Faster writes, but a risk of data loss if the cache fails before data is persisted.
- **Write-Around:** Data is written directly to the datastore, bypassing the cache. Only data that is read makes it into the cache.

Choosing the right caching strategy, system, and patterns depends heavily on your application's specific requirements, access patterns, and scalability needs.
