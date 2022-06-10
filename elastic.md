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

---

### ElasticSearch

* Index = logical namespace(broken into shards in order to distribute the data and scale) which maps to one ore more primary shards and can have zero or more replica shards.

* Shards = physical data files which are split into chunks and are distributed across the cluster.
  * you cannot delete unassigned shards, an unassigned shard is not a corrupted shard, but a missing replica
  * replica shards = one or more copies of your index's shards
  * primary vs replica = replica is promoted to primary(primary cannot be on the same node as the replica)

* Cluster settings:
  * Transient – Changes that will not persist after a full cluster restart
  * Persistent – Changes that will be saved after a full cluster restart
  * Get settings: `GET /_cluster/settings` 
  ```bash
  {
  "persistent" : { },
  "transient" : { }
  }
    ```

* Elasticsearch contains multiple circuit breakers used to prevent operations from causing on OutOfMemoryError.

  - by default the parent ciruit breaker triggers at 95% JVM memory usage
  - high JVM memory pressure cause circuit breaker errors, each circtuit breaker specifies a limit for how much memory it can use.
  - Elasticsearch will throw a  “CircuitBreakerException” and reject the request rather than risk crashing the entire node

### Check the health of ES:
```bash
curl localhost:9201/_cluster/health?pretty
curl -X GET http://localhost:9201/_cat/nodes?v
curl -X GET http://localhost:9201/_cat/master?v
curl -XGET localhost:9200/_cluster/allocation/explain?pretty
curl -X GET localhost:9200/_cat/shards?h=index,shard,prirep,state,unassigned.reason
curl -X GET localhost:9200/_cat/indices?v
# return HEAP information
curl -X GET localhost:9200/_cat/nodes?v=true
curl http://dc1-elke004.sgdmz.local:9200/_nodes/thread_pool?pretty
```
```bash
# cluster health
GET _cluster/health

# unassigned shards allocation explained
GET /_cluster/allocation/explain

# indices table
GET /_cat/indices?v 

# fixing unassigned shards
curl -XPOST 'localhost:9200/_cluster/reroute?retry_failed' 

# shards stuff
POST _cluster/reroute?retry_failed
GET _cluster/health?filter_path=status,*_shards
GET _cluster/allocation/explain
```

### CircuitBreaker due to JVM (heap)pressure:
  * High JVM memory usage can degrade cluster performance and trigger circuit breaker errors. To prevent this, we recommend taking steps to reduce memory pressure if a node’s JVM memory usage consistently exceeds 85%.

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

---
```bash
# return just indices
curl -X GET "localhost:9200/_nodes/stats/indices?pretty"
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