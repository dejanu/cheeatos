* index/type/document == bookstore_db/book_table/1

* Index = logical namespace(broken into shards in order to distribute the data and scale) which maps to one ore more primary shards and can have zero or more replica shards.

* Shards = physical data files which are split into chunks and are distributed across the cluster.

* Elasticsearch contains multiple circuit breakers used to prevent operations from causing on OutOfMemoryError.

  - by default the parent ciruit breaker triggers at 95% JVM memory usage
  - high JVM memory pressure cause circuit breaker errors, each circtuit breaker specifies a limit for how much memory it can use.
  - Elasticsearch will throw a  “CircuitBreakerException” and reject the request rather than risk crashing the entire node
  - 50% of memory on an Elasticsearch node is generally used for the JVM  heap, while the other half of the memory is used for other requirements such as cache.

Setting Changes
* Transient – Changes that will not persist after a full cluster restart
* Persistent – Changes that will be saved after a full cluster restart

# Every shard uses memory, a small set of large shards uses fewer resources than many small shards - check health and shards status allocation

GET _cluster/health
GET _cluster/health?filter_path=status,*_shards
GET _cluster/allocation/explain?pretty

# retry to decrease unassigned_shards
POST _cluster/reroute?retry_failed


# get cluster settings (default and new/modified) e.g. parent circuit breaker total.limit
GET /_cluster/settings?include_defaults=true
GET /_cluster/settings


# Check JVM memory pressure => JVM Memory Pressure = used_in_bytes / max_in_bytes
GET _nodes/stats?filter_path=nodes.*.jvm.mem.pools.old


#Statistics about the field data circuit breaker
GET _nodes/stats/breaker

#JVM stats, memory pool information, garbage collection, buffer pools, number of loaded/unloaded classes.
GET/_nodes/stats/jvm

# "reason" : "[parent] Data too large, data for [cluster:monitor/nodes/stats[n]] would be [528680448/504.1mb], which is larger than the limit of [524288000/500mb], 
real usage: [528679104/504.1mb], new bytes reserved: [1344/1.3kb], usages [request=0/0b, fielddata=0/0b, in_flight_requests=34224/33.4kb

# update parent CircuitBreaker
PUT _cluster/settings
{
  "transient": {"indices.breaker.total.limit":"500mb" }
}


GET _cat/nodes?v=true&h=name,node*,heap*
GET _nodes/stats?filter_path=nodes.*.jvm.mem.pools.old
GET /_cat/shards

GET _cat/shards?v=true&h=index,prirep,shard,store&s=prirep,store&bytes=gb

 #relevant files. (open your synergy-config and find them on the path given by the properties values.)
 bundleConfig.logging.opensearchDataCustomization 
 bundleConfig.logging.opensearchMasterCustomization 
 
 



"parent" : {
          "limit_size_in_bytes" : 510027366,
          "limit_size" : "486.3mb",
          "estimated_size_in_bytes" : 314815296,
          "estimated_size" : "300.2mb",
          "overhead" : 1.0,
          "tripped" : 0
		  
 "parent" : {
          "limit_size_in_bytes" : 510027366,
          "limit_size" : "486.3mb",
          "estimated_size_in_bytes" : 483634776,
          "estimated_size" : "461.2mb",
          "overhead" : 1.0,
          "tripped" : 485504
		  
		  

# get all indices and check the number of replicas of each index
GET _cat/indices?v


# replica shards = one or more copies of your index's shards 



explanation" : "a copy of this shard is already allocated to this node 
[[.opendistro-ism-managed-index-history-2022.04.24-000079][0], node[UnlCU7hkSJKXK7dfK6pcwg], [P], s[STARTED], a[id=bvStxrLmTpSYW3PHycIhvA]]"

PUT /jaeger-service-2022-04-01/_settings
{
  "index" : {
    "number_of_replicas" : 0
  }
}
