# FLiP-Stream2Clickhouse

Stonks. Stocks.   Apache Pulsar - Stream to Altinity Cloud - Clickhouse

## OSACon 2021

OSACON 2021 - Hello Hydrate! From Stream to Clickhouse with Apache Pulsar and Friends


## altinity cloud setup

* Login
* Launch Cluster
* Explore


```


```


## build the environment

```

bin/pulsar-admin sink stop --name solr-sink-energy --namespace default --tenant public
bin/pulsar-admin sinks delete --tenant public --namespace default --name solr-sink-energy
bin/pulsar-admin sinks create --tenant public --namespace default --name solr-sink-energy --sink-type solr --sink-config-file conf/solr-sink-energy.yml --inputs energy
bin/pulsar-admin sinks get --tenant public --namespace default --name solr-sink-energy
bin/pulsar-admin sinks status --tenant public --namespace default --name solr-sink-energy

bin/pulsar-client consume "persistent://public/default/stocks" -s stonks-reader

```
