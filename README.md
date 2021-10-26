# FLiP-Stream2Clickhouse

Stonks. Stocks.   Apache Pulsar - Stream to Altinity Cloud - Clickhouse

## OSACon 2021

OSACON 2021 - Hello Hydrate! From Stream to Clickhouse with Apache Pulsar and Friends


## altinity cloud setup

* Login
* Launch Cluster
* Explore
* Build table


```
CREATE TABLE IF NOT EXISTS stocks ON CLUSTER '{cluster}'
(
    symbol String, 
    uuid String,
    ts Date,
    dt	 String,
   datetime String,
   open String, 
   close String,
   high String,
   volume String,
   low String
) ENGINE = ReplicatedMergeTree('/clickhouse/{cluster}/tables/{shard}/{database}/{table}', '{replica}')
    PARTITION BY toYYYYMM(ts)
    ORDER BY (symbol);

```

## Altinity Cloud / Clickhouse / JDBC Sink Configuration

```
tenant: "public"
namespace: "default"
name: "jdbc-clickhouse-sink"
topicName: "persistent://public/default/stocks"
sinkType: "jdbc-clickhouse"
configs:
    userName: "myusernameisreallycool"
    password: "mypasswordissolongyouwillnotrememberitever123"
    jdbcUrl: "jdbc:clickhouse://streamnative.domain.altinity.cloud:8443?ssl=true"
    tableName: "stocks"
    
```

## Build the Pulsar environment (Or Just click create topic in StreamNative Cloud)

```
bin/pulsar-admin schemas delete stocks
bin/pulsar-admin schemas delete persistent://public/default/stocks

bin/pulsar-admin topics list public/default
bin/pulsar-admin topics create persistent://public/default/stocks

bin/pulsar-admin topics info-internal persistent://public/default/stocks
bin/pulsar-admin topics stats-internal persistent://public/default/stocks

bin/pulsar-admin sinks stop --tenant public --namespace default --name jdbc-clickhouse-sink
bin/pulsar-admin sinks delete --tenant public --namespace default --name jdbc-clickhouse-sink
bin/pulsar-admin sinks restart --tenant public --namespace default --name jdbc-clickhouse-sink

bin/pulsar-admin sinks create --archive ./connectors/pulsar-io-jdbc-clickhouse-2.8.0.nar --inputs stocks --name jdbc-clickhouse-sink --sink-config-file conf/clickhouse.yml --parallelism 1
bin/pulsar-admin sinks list --tenant public --namespace default
bin/pulsar-admin sinks get --tenant public --namespace default --name jdbc-clickhouse-sink
bin/pulsar-admin sinks status --tenant public --namespace default --name jdbc-clickhouse-sink

bin/pulsar-client consume "persistent://public/default/stocks" -s stonks-reader

```


## References

* https://github.com/tspannhw/SmartStocks
* https://github.com/tspannhw/FLiP-SQL/
* https://docs.altinity.com/altinitycloud/quickstartguide/yourfirstqueries/
