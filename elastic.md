## Cheatsheet collection

* [Home](index.md)
* [Ansible](ansible.md)
* [Git](git.md)
* [GCP](gcp.md)
* [Docker](docker.md)
* [Azure](azure.md)
* [Terraform](terraform.md)
* [Helm](helm.md)
* <ins>[ElasticSearch](elastic.md)<ins>
* [Kubernetes](k8s.md)
* [Istio](istio.md)
* [OIDC](openID.md)
* [PostgreSQL](postgres.md)
* [Terraform](terraform.md)

---

### General

* OpenSearch splits **indices** into **shards**. Each shard stores a subset of all **documents** in an index.

* Cluster Health:

```bash
GET _cluster/health
GET _cluster/stats

# human and pretty
GET _cluster/health?pretty=true
GET _cluster/health?human=true

GET _cat/nodes?v=true
GET _cluster/stats/nodes/opensearch-cluster-data-0 #update opensearch-cluster-data-0 node name

# nodes table view for heam percent disk used and cpu
GET _cat/nodes?v&s=master,name&h=name,master,node.role,heap.percent,disk.used_percent,cpu
GET _cat/nodes?v=true&h=heap.current,name

# nodes JVM status (GC is triggered by JVM, i.e. when the heap is getting full GC starts)
GET _nodes/stats?filter_path=nodes.*.jvm.mem.pools.old
```

* Indices Health:

```bash

# indices table
GET _cat/indices?v&pretty

# sort indices by size or date
GET _cat/indices?v&s=store.size:desc
GET _cat/indices?v&s=creation.date:desc

# use aliases i.e. k8s-* for indices like k8s-logging, k8s-events
GET _cat/indices/k8s-*?v&s=creation.date:desc

# return all of the DOCUMENTS in a specific INDEX using a "match_all" query
GET <index>/_search
{
    "query": {
        "match_all": {}
    }
}

# get the INDEX settings and stats
GET <index>/_settings
GET /<index>/_stats

# get no of documents
GET <index>/_count

# get he number of docs in each and health index
GET _cat/indices?h=index,creation.date,docs.count,health&format=json

# get indices - Most Elasticsearch APIs accept an alias in place of a data stream or index name
GET _aliases/?pretty=true


# close index
POST /<index>/_close

# reduce replica shards to 0
PUT /<index>/_settings
{
  "index" : {
	"number_of_replicas" : 0
  }
}

DELETE <INDEX>
```

### Shards

* Shard size matters because it impacts both search latency and write performance:

  * For fast indexing (ingestion), you need as many shards as possible; for fast searching, it is better to have as few shards as possible.

  * A small set of large shards uses fewer resources than many small shards (too many small shards will exhaust the memory - JVM Heap), however, on the other side, too few large shards prevent OpenSearch from properly distributing requests.

* When writing or searching data within an index, that index it’s in an open state, the longer you keep the indices open the more shards you use.

* Very important red indexes cause red shards, and red shards cause red clusters.

* Unassigned shards cannot be deleted, an unassigned shard is not a corrupted shard, but a missing replica.

```bash
# shard status: no of shards in the cluster
GET _cluster/health?filter_path=status,*_shards
GET _cluster/health?level=shards

# table view for index and shards
GET _cat/shards?v=true&h=index,prirep,shard,store&s=prirep,store&bytes=gb

# sort shards
GET _cat/shards?v&s=store:desc

# node shard distribution 
GET _cat/shards?v=true&s=store:desc&h=index,node,store

# get shards for specific index (check shard size)
GET _cat/shards/<index>?v

# unassigned shards allocation explained and unassigned reason
GET _cluster/allocation/explain
GET _cat/shards?h=index,shards,state,prirep,unassigned.reason

# shards retry allocation
POST _cluster/reroute?retry_failed

# some with curl
curl -X GET http://localhost:9200/_cluster/allocation/explain?pretty
curl -X GET http://localhost:9200/_cat/shards?h=index,shards,state,prirep,unassigned.reason
```
* Manually move shard (reroute) from one node to another:
```json
POST _cluster/reroute
{
 "commands": [
   {
     "move": {
       "index": "jaeger-span-2024-11-29", "shard": 0,
       "from_node": "opensearch-cluster-data-5", "to_node": "opensearch-cluster-data-7"
     }
   }
 ]
}
```

* To identify the indexes causing the red cluster status:
![alt text](https://github.com/dejanu/cheetcity/blob/gh-pages/src/shards.PNG?raw=true)


### Cluster settings

* Transient – Changes that will not persist after a full cluster restart
* Persistent – Changes that will be saved after a full cluster restart
* Get settings: `GET /_cluster/settings`

* There is a limit on how many shards a node can handle. Each node can accomodate a no of shards, Check how many shards a node can accomodate and search check `max_shards_per_node` setting 

```bash
# Check how many shards a node can accomodate and search 
# cluster.max_shards_per_node setting. integer: Limits the total number of primary and replica shards for the cluster
GET /_cluster/settings?include_defaults=true

# update the no of shards per node
curl -X PUT localhost:9200/_cluster/settings -H "Content-Type: application/json" -d '{ "persistent": { "cluster.max_shards_per_node": "1100" } }'

PUT _cluster/settings
{
  "persistent": {
    "cluster.max_shards_per_node": 1100
  }
}
```

* Elasticsearch contains multiple circuit breakers used to prevent operations from causing on OutOfMemoryError.

  - by default the parent ciruit breaker triggers at 95% JVM memory usage
  - high JVM memory pressure cause circuit breaker errors, each circtuit breaker specifies a limit for how much memory it can use.
  - Elasticsearch will throw a  “CircuitBreakerException” and reject the request rather than risk crashing the entire node

### Check the health of ES

```bash
# cURL usage with auth
curl -XGET -k -i -u 'user:password' "{OPENSEARCH_URL}:9200/{YOUR_INDEX}/_search"

curl -X GET http://localhost:9200/_cat/shards?h=index,shard,prirep,state,unassigned.reason
curl -X GET http://localhost:9200/_cat/indices?v

curl -X GET http://localhost:9200/_cluster/health?pretty

curl -X GET http://localhost:9200/_cat/nodes?v
curl -X GET http://localhost:9200/_cat/master?v


# return HEAP information
curl -X GET http://localhost:9200/_cat/nodes?v=true
curl http://dc1-elke004.sgdmz.local:9200/_nodes/thread_pool?pretty
 
# fixing unassigned shards
curl -XPOST 'localhost:9200/_cluster/reroute?retry_failed' 
```

### CircuitBreaker due to JVM (heap)pressure:

* The heap size is the amount of RAM allocated to the JVM.

* High JVM memory usage can degrade cluster performance and trigger circuit breaker errors. To prevent this, we recommend taking steps to reduce memory pressure if a node’s JVM memory usage consistently exceeds 85%.
  
* Every shard uses memory, a small set of large shards uses fewer resources than many small shards - check health and shards status allocation

```bash

# check percentage of memory that is currently used by the heap
GET _cat/nodes?v=true&h=name,heap.current,heap.percent

# each instance displays a JVM memory pressure indicator
GET _nodes/stats?filter_path=nodes.*.jvm.mem.pools.old

# Use the response to calculate memory pressure as follows:
# JVM Memory Pressure = used_in_bytes / max_in_bytes

curl -X GET "localhost:9200/_nodes/stats/breaker?pretty"
# update parent CircuitBreaker as a transient setting
PUT _cluster/settings
{
  "transient": {"indices.breaker.total.limit":"500mb" }
}

```

### cURL stuff

```bash
# return just indices
curl -X GET "localhost:9200/_nodes/stats/indices?pretty"

# return sorted indices by size
curl -X GET "http://localhost:9200/_cat/indices?pretty&s=store.size:desc"

# return sorted indices by date
curl -X  GET "http://localhost:9200/_cat/indices?pretty&s=creation.date"

# allocation of disk space for indices and the number of shards on each node.
curl -X GET _cat/allocation?v

# get the current shard size
curl -X GET _cat/shards?v=true&h=index,prirep,shard,store&s=prirep,store&bytes=gb

# return just os and process
curl -X GET "localhost:9200/_nodes/stats/os,process?pretty"
# return just process for node with IP address 10.0.0.1
curl -X GET "localhost:9200/_nodes/10.0.0.1/stats/process?pretty"

# Fielddata summarized by node
curl -X GET "localhost:9200/_nodes/stats/indices/fielddata?fields=field1,field2&pretty"
# Fielddata summarized by node and index
curl -X GET "localhost:9200/_nodes/stats/indices/fielddata?level=indices&fields=field1,field2&pretty"
# Fielddata summarized by node, index, and shard
curl -X GET "localhost:9200/_nodes/stats/indices/fielddata?level=shards&fields=field1,field2&pretty"
# You can use wildcards for field names
curl -X GET "localhost:9200/_nodes/stats/indices/fielddata?fields=field*&pretty"

# All groups with all stats
curl -X GET "localhost:9200/_nodes/stats?groups=_all&pretty"
# Some groups from just the indices stats
curl -X GET "localhost:9200/_nodes/stats/indices?groups=foo,bar&pretty"

# Retrive Ingest stats only https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-nodes-stats.html#cluster-nodes-stats-ingest-ex
curl -X GET "localhost:9200/_nodes/stats/ingest?filter_path=nodes.*.ingest&pretty"
curl -X GET "localhost:9200/_nodes/stats?metric=ingest&filter_path=nodes.*.ingest&pretty"
curl -X GET "localhost:9200/_nodes/stats?metric=ingest&filter_path=nodes.*.ingest.pipelines&pretty"
```
---

### Links:

* [How To Return All Documents From An Index In Elasticsearch](https://kb.objectrocket.com/elasticsearch/how-to-return-all-documents-from-an-index-in-elasticsearch)
* [Elasticsearch Concepts](https://logz.io/blog/10-elasticsearch-concepts/)
* [OpenSearch CLI](https://opensearch.org/docs/1.2/clients/cli/)
* [Sizing indices](https://opensearch.org/blog/optimize-opensearch-index-shard-size/)
* [Error handling](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/handling-errors.html)
 
```bash
                    ___ _____
                   /\ (_)    \
                  /  \      (_,
                 _)  _\   _    \
                /   (_)\_( )____\
                \_     /    _  _/
                  ) /\/  _ (o)(
                  \ \_) (o)   /
                   \/________/         @dejanualex
```