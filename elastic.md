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

---

### General

* OpenSearch splits **indices** into **shards**. Each shard stores a subset of all **documents** in an index.

* Indices Health:

```bash
# cluster general health
GET _cluster/health
GET _cluster/stats
GET _cat/nodes?v=true

# shard status: no of shards in the cluster
GET _cluster/health?filter_path=status,*_shards

# indices table
GET /_cat/indices?v
GET _cat/indices?v&pretty

# sort indices by size or date
GET _cat/indices?v&s=store.size:desc
GET _cat/indices?v&s=creation.date:desc

# Get a specific INDEX and return all of the documents in an index using a "match_all" qu
GET <index>/_search
{
    "query": {
        "match_all": {}
    }
}

# Get the INDEX settings
GET <index>/_settings

# get indices - Most Elasticsearch APIs accept an alias in place of a data stream or index name
GET _aliases/?pretty=true
```

### Shards

* Shard size matters because it impacts both search latency and write performance, too many small shards will exhaust the memory (JVM Heap) too few large shards prevent OpenSearch from properly distributing requests, a good rule of thumb is to keep shard size between 10–50 GB.

* When writing or searching data within an index, that index it’s in an open state, the longer you keep the indices open the more shards you use.

* Very important red indexes cause red shards, and red shards cause red clusters.

* Unassigned shards cannot be deleted, an unassigned shard is not a corrupted shard, but a missing replica.

```bash
# check the global cluster status for shards
GET _cluster/health?filter_path=status,*_shards
GET _cluster/health?level=shards

# get shards for specific index (check shard size)
GET _cat/shards/<index>?v

# unassigned shards allocation explained and unassigned reason
GET _cluster/allocation/explain
GET _cluster/allocation/explain?pretty
GET _cat/shards?h=index,shards,state,prirep,unassigned.reason

# shards retry allocation
POST _cluster/reroute?retry_failed

# some with curl
curl -X GET http://localhost:9200/_cluster/allocation/explain?pretty
curl -X GET http://localhost:9200/_cat/shards?h=index,shards,state,prirep,unassigned.reason
```

* To identify the indexes causing the red cluster status:
![alt text](https://github.com/dejanu/cheetcity/blob/gh-pages/src/shards.PNG?raw=true)

* There is a limit on how many shards a node can handle. Each node can accomodate a no of shards, Check how many shards a node can accomodate and search check `max_shards_per_node` setting 

```bash
# Check how many shards a node can accomodate and search 
# cluster.max_shards_per_node setting. integer: Limits the total number of primary and replica shards for the cluster
GET /_cluster/settings?include_defaults=true

# update the no of shards per node
curl -X PUT localhost:9200/_cluster/settings -H "Content-Type: application/json" -d '{ "persistent": { "cluster.max_shards_per_node": "3000" } }'

PUT _cluster/settings
{
  "persistent": {
    "cluster.max_shards_per_node": 1100
  }
}
```

* Cluster settings:
  * Transient – Changes that will not persist after a full cluster restart
  * Persistent – Changes that will be saved after a full cluster restart
  * Get settings: `GET /_cluster/settings`

```bash

  # include default settings
  GET /_cluster/settings?include_defaults=true
  
  {
  "persistent" : { },
  "transient" : { }
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
  * High JVM memory usage can degrade cluster performance and trigger circuit breaker errors. To prevent this, we recommend taking steps to reduce memory pressure if a node’s JVM memory usage consistently exceeds 85%.
  * Every shard uses memory, a small set of large shards uses fewer resources than many small shards - check health and shards status allocation

```bash
curl -X GET "localhost:9200/_nodes/stats/breaker?pretty"

# each instance displays a JVM memory pressure indicator
GET _nodes/stats?filter_path=nodes.*.jvm.mem.pools.old

# Use the response to calculate memory pressure as follows:
# JVM Memory Pressure = used_in_bytes / max_in_bytes

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