* [Home](index.md)
* [Ansible](ansible.md)
* [Git](git.md)
* [GCP](gcp.md)
* [Docker](docker.md)
* [Azure](azure.md)
* [Terraform](terraform.md)
* [Helm](helm.md)
* </ins>[ElasticSearch](elastic.md)</ins>
* [Kubernetes](k8s.md)
* [Istio](istio.md)


### Check the health of ES:

```bash
curl localhost:9201/_cluster/health?pretty
curl -X GET http://localhost:9201/_cat/nodes?v
curl -X GET http://localhost:9201/_cat/master?v
curl -XGET localhost:9200/_cluster/allocation/explain?pretty
curl -X GET localhost:9200/_cat/shards?h=index,shard,prirep,state,unassigned.reason
curl -X GET localhost:9200/_cat/indices?v
curl -X GET localhost:9200/_cat/nodes?v=true  # return HEAP information
curl http://dc1-elke004.sgdmz.local:9200/_nodes/thread_pool?pretty
curl -XPOST 'localhost:9200/_cluster/reroute?retry_failed' # fixing unassigned shards

GET /_cluster/allocation/explain  and  GET /_cat/indices?v 
```


* From https://www.bluematador.com/docs/troubleshooting/aws-elasticsearch-status

### CircuitBreaker due to JVM (heap)pressure:
```bash
curl -X GET "localhost:9200/_nodes/stats/breaker?pretty"
To calculate the current JVM memory pressure for each node, use the nodes stats API.
GET _nodes/stats?filter_path=nodes.*.jvm.mem.pools.old
Copy as curlView in Console
Use the response to calculate memory pressure as follows:
JVM Memory Pressure = used_in_bytes / max_in_bytes
```

* From https://www.elastic.co/guide/en/elasticsearch/reference/7.15/fix-common-cluster-issues.html#high-jvm-memory-pressure

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