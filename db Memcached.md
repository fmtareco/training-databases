# Memcached database

* Memcached is a 
	* high-performance, 
	* distributed memory
	* object caching system. 
* Originally developed in 2003 for LiveJournal, it is 
	* designed to speed up dynamic web applications 
	* by alleviating database load. 

----

# Purpose and Core Concept

* Memcached is a 
	* volatile, 
	* in-memory key-value store. 
	
* primary purpose is 
	* to store small chunks of arbitrary data (strings, objects) 
	* resulting from 
		* database calls, 
		* API calls, or 
		* page rendering. 

The Goal 
--
* Reduce the number of times a "heavy" data source (like a SQL database) must be read 
* by keeping the most frequently accessed data in RAM. 

Memcached Overview
--
* Free & open source, 
	* high-performance, 
	* distributed memory object caching system, 
	* generic in nature, but intended for use in speeding up dynamic web applications by alleviating database load.

* Memcached is an 
	* in-memory 
	* key-value store 
		* for small arbitrary data (strings, objects) 
		* from results of database calls, API calls, or page rendering.

* Memcached is simple yet powerful. 
	* simple design promotes 
		 * quick deployment, 
		 * ease of development, and 
		 * solves many problems facing large data caches. 
	* API is available for most popular languages.


What is it Made Up Of? 
--
* Client software, which is given a list of available memcached servers.
* client-based hashing algorithm, which chooses a server based on the “key”.
* Server software, which stores values with their keys into an internal hash table.
* LRU, which determine when to throw out old data (if out of memory), or reuse memory.

Design Philosophy 
--
* Simple Key/Value Store 
	* server does not care what your data looks like. 
	* Items are made up of a key, an expiration time, optional flags, and raw data. 
	* does **not understand data structures**; you must upload **data that is pre-serialized**. 
	* Some commands (***incr***/***decr***) may operate on the underlying data, but in a simple manner.

* Logic Half in Client, Half in Server 
	* A “***memcached implementation***” is partially in a client, and partially in a server. 
	* Clients understand 
		* how to choose which server to read or write to for an item, 
		* what to do when it cannot contact a server.
	 servers understand 
		* how to store and fetch items. 
		* also manage when to evict or reuse memory.

* Servers are Disconnected From Each Other 
	* servers are unaware of each other. 
	* There is no crosstalk, no synchronization, no broadcasting, no replication. 
	* Adding servers increases the available memory. 
	* Cache invalidation is simplified, as clients delete or overwrite data on the server which owns it directly.

O(1) 
--
	* All commands are implemented to be as fast and lock-friendly as possible. 
		* allows near-deterministic query speeds for all use cases.

	* Queries on slow machines should run in well under 1ms. 
		* High end servers can serve millions of keys per second in throughput.

	* Forgetting is a Feature 
		* Memcached is, by default, a ***Least Recently Used*** cache. 
		* Items expire after a specified amount of time. 
		* Both of these are elegant solutions to many problems; 
			* Expire items after a minute to limit stale data being returned, or 
			* flush unused data in an effort to retain frequently requested information.

	* No “pauses” waiting for a garbage collector 
		* ensures low latency, and 
		* free space is lazily reclaimed.

	* Cache Invalidation 
		* Rather than broadcasting changes to all available hosts, 
			* clients directly address the server holding the data to be invalidated.



----


# Most Relevant Technical Aspects

* Simplicity: 
	* uses a simple client-server architecture with a 
	* minimal set of commands (get, set, add, replace, delete).
* Multi-threaded: 
	* Unlike the **original Redis**, 
	* is natively multi-threaded, 
	* allowing it to scale linearly on multi-core machines.
* Memory Management (Slabs): 
	* uses "***Slab Allocation***" to prevent memory fragmentation. 
	* pre-allocates memory in chunks of specific sizes.
* LRU (Least Recently Used): 
	* cache is full, automatically discards the oldest, least-used data to make room for new data.
* Stateless/Distributed: 
	* "***distributed***" part happens at the client level. 
	* client knows which server holds which key using a hashing algorithm (like ***Consistent Hashing***).

3. Relevance Today
--
* Redis has taken much of the market share, 
* Memcached remains highly relevant for:
	* Large-scale simple caching: 
		* Facebook, Netflix, and Wikipedia for 
		* massive, high-throughput caching layers where 
		* complex data structures aren't needed.
	* Cloud Infrastructure: 
		* AWS (***ElastiCache***) and 
		* Google Cloud offer 
		* Memcached as a **managed service** because of its 
		* predictable performance and low overhead. 

4. Memcached vs. Others (Comparison)
--
| Feature  | Memcached | Redis |
|---|---|---|
| Data Types | Simple Strings/Blobs only. | Strings, Lists, Sets, Hashes, Bitmaps. |
| Persistence | None. (If it restarts, data is gone). | Optional (RDB/AOF snapshots to disk). |
| Threading | Multi-threaded (Scales on CPU cores). | Mostly Single-threaded (Event-loop). |
| Architecture | Simple, distributed via client-side. | Complex, built-in Cluster/Sentinel support. |
| Best Use Case | Pure, high-speed caching. | Caching, Message Broker, Leaderboards. |


5. Practice 
--
* Consistent Hashing: 
	* Understand how clients distribute keys across multiple nodes so that adding a new node doesn't invalidate the entire cache.
* Cache Invalidation Strategies: 
	* Learn the ***"Cache-Aside" pattern*** 
		* (check cache -> if miss, read DB -> update cache).
* The ***"Thundering Herd"*** Problem: 
	* What happens when a very popular key expires and 10,000 requests hit the DB at once? (
		* Practice using Locks or Leases).
* ***Slab Eviction***: 
	* Understand what happens when you store many items that are slightly larger than the slab size (internal fragmentation).

6. Recommended Resources
--
* Official Documentation: memcached.org 
	* (The "About" and "Tutorial" sections are gold).
	* A Story of Caching: https://docs.memcached.org/tutorialcachingstory/	
* AWS Documentation: 
	* Search for "Best Practices for ElastiCache Memcached"
	* it contains great architectural advice.
* Docker: 
	* Run docker run --name my-memcache -p 11211:11211 -d memcached and 
	* use a Java client like ***SpyMemcached*** or ***Lettuce*** to experiment.
* System Design Primer (GitHub): 
	* Look for the "Caching" section in the 
	* donnemartin/system-design-primer repository.

----


#  Memcached in a Spring Boot application

* we typically use a provider like 
	* ***XMemcached*** or ***SpyMemcached***, as 
	* Spring doesn't have a "starter" for it as it does for Redis.
	
Implementation of the Cache-Aside pattern (***Look-aside caching***).
--
* Dependency (Maven)
	* Add the XMemcached client to your pom.xml:
---
<dependency>
    <groupId>com.googlecode.xmemcached</groupId>
    <artifactId>xmemcached</artifactId>
    <version>2.4.7</version>
</dependency>
---

* Configuration
	* Create a Bean to manage the connection to your Memcached server 
		* (usually running on port 11211).
---
@Configuration
public class MemcachedConfig {
    @Bean
    public MemcachedClient memcachedClient() throws IOException {
        // Points to localhost for dev; use environment variables for Prod
        return new XMemcachedClientBuilder("127.0.0.1:11211").build();
    }
}
---

* Service with Cache-Aside Logic
	* In this pattern, the application code is responsible for checking the cache first.
---
	@Service
	public class ProductService {

		@Autowired
		private ProductRepository repository;

		@Autowired
		private MemcachedClient cache;

		public Product getProduct(Long id) throws Exception {
			String key = "product:" + id;

			// 1. Try to get from Memcached
			Product cachedProduct = cache.get(key);

			if (cachedProduct != null) {
				System.out.println("Cache Hit!");
				return cachedProduct;
			}

			// 2. Cache Miss: Get from Database
			System.out.println("Cache Miss! Fetching from DB...");
			Product dbProduct = repository.findById(id)
					.orElseThrow(() -> new RuntimeException("Not Found"));

			// 3. Put in Cache for next time (Expires in 3600 seconds/1 hour)
			cache.set(key, 3600, dbProduct);

			return dbProduct;
		}
	}
---


4. Key Performance Strategies for Interviews
--
* Lazy Loading: 
	* Data is only cached when requested. 
	* This saves memory but adds a small latency penalty on the first request (the "miss").
* Expiration (TTL): 
	* Always set a Time To Live. 
	* Memcached is volatile; 
		* if you don't set a TTL, 
		* your data might become "stale" (outdated) compared to the database.
* Serialization: 
	* Memcached stores data as byte[]. 
	* Your Product class **must implement Serializable** or be **converted to JSON** before storing.
* Avoid Large Objects: 
	* Memcached has a default limit of 1MB per item. 
	* Do not try to cache entire database tables; 
		* cache specific rows or DTOs.

* Common Question: What if Memcached goes down?
	* Since Memcached is a cache and not a primary database, 
		* the application should gracefully fall back to the database. 
	* It will be slower (high latency), 
		* but the system remains functional. 
	* This is called Graceful Degradation.

----

# Cache Stampede (or "Thundering Herd") 

* happens when a very popular cached key (like homepage_data) expires. 
* Suddenly, thousands of concurrent requests see a cache miss and all hit your database at the same time to recalculate the data. 
	* This can crash your DB.
	
* The three most common ways to solve this in a Spring Boot environment:
	* 1. Simple Locking (Synchronization) - ***Double-checked locking***
		* easiest way is to ensure **only one thread per application instance** 
			* is allowed to fetch from the DB for a specific key.
---
	public Product getProduct(Long id) throws Exception {
		String key = "product:" + id;
		Product p = cache.get(key);

		if (p == null) {
			// Only one thread enters this block per JVM
			synchronized (this) {
				// Check again! (Double-checked locking)
				p = cache.get(key);
				if (p == null) {
					p = repository.findById(id).orElseThrow();
					cache.set(key, 3600, p);
				}
			}
		}
		return p;
	}
---

	* 2. Distributed Locking (Using Memcached add)
		* In a microservices environment, synchronized only works for one instance. 
		* To stop all instances from hitting the DB, 
			* use the Memcached add command as a "mutex" (lock). 
			* add only succeeds if the key does not exist.
---
	public Product getProductSecurely(Long id) throws Exception {
		String key = "product:" + id;
		String lockKey = "lock:" + key;
		Product p = cache.get(key);

		if (p == null) {
			// Try to acquire a 5-second lock
			if (cache.add(lockKey, 5, "LOCKED")) {
				try {
					p = repository.findById(id).orElseThrow();
					cache.set(key, 3600, p);
				} finally {
					cache.delete(lockKey); // Release lock
				}
			} else {
				// Someone else is fetching. Wait a bit and retry the cache.
				Thread.sleep(100);
				return getProductSecurely(id);
			}
		}
		return p;
	}
---

	* 3. Probabilistic Early Recomputation (X-Fetch)
		* Instead of waiting for the key to expire, 
			* you recompute it just before it dies.
		* How: 
			* You store the expiration timestamp inside the cached object.
		* Logic: 
			* As the time gets closer to expiry, there is a random chance a thread will decide to refresh the cache early while the old data is still there for others to use.

How to prevent a Cache Stampede:
--
* use ***Double-Checked Locking*** for single-instance apps 
* or ***Distributed Mutexes** using Memcached's atomic add operation for microservices. 
* For extremely high-traffic keys, implement ***Background Refreshing*** or ***Probabilistic Expiry** 
	* to ensure the cache is never truly empty."

Resources to Deep Dive:

* Paper: "Optimal Probabilistic Cache Expiry" (The math behind X-Fetch).
* Pattern: External Locking (using a tool like Redis or Memcached to coordinate).

Would you like to move on to database sharding or explore Redis Cluster vs. Memcached Distribution?

