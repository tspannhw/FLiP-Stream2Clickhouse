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
drop table stocks ON CLUSTER '{cluster}';
drop table stocks_local ON CLUSTER '{cluster}';

alter table stocks_local  delete where ts = 0;

CREATE TABLE IF NOT EXISTS stocks_local ON CLUSTER '{cluster}'
(
    symbol String, 
    uuid String,
    ts Int32,
    dt Int32,
    datetime String,
    open String, 
    close String,
    high String,
    volume String,
    low String
) ENGINE = MergeTree()
  PARTITION BY symbol
  ORDER BY (symbol);
  
CREATE TABLE stocks ON CLUSTER '{cluster}' AS stocks_local
ENGINE = Distributed('{cluster}', default, stocks_local, rand());

 INSERT INTO stocks VALUES('IBM', '6bec81c6', 1634912880810, 1611327960000,  '2021/01/22 10:06:00', '340.83099','341.38000','341.38000','2198','340.83099');
 
 drop table iotjetsonjson ON CLUSTER '{cluster}';
drop table iotjetsonjson_local ON CLUSTER '{cluster}';

CREATE TABLE iotjetsonjson_local
(
	uuid String, 
	camera String,
	ipaddress String,  
	networktime String, 
        top1pct String, 
	top1 String, 
	cputemp String, 
	gputemp String,
        gputempf String,
	cputempf String, 
	runtime String,
	host String,
	filename String,  
	host_name String, 
        macaddress String, 
	te String, 
	systemtime String,
	cpu String,
        diskusage String,
	memory String, 
	imageinput String
)
ENGINE = MergeTree()
  PARTITION BY uuid
  ORDER BY (uuid);
  
  
CREATE TABLE iotjetsonjson ON CLUSTER '{cluster}' AS iotjetsonjson_local
ENGINE = Distributed('{cluster}', default, iotjetsonjson_local, rand());


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
    jdbcUrl: "jdbc:clickhouse://streamdomaininthe.cloud:8443?ssl=true"
    tableName: "stocks_local"
    
    
tenant: "public"
namespace: "default"
name: "jdbc-clickhouse-sink-iot"
topicName: "persistent://public/default/iotjetsonjson"
sinkType: "jdbc-clickhouse"
configs:
    userName: "youradminname"
    password: "somepasswordthatiscool"
    jdbcUrl: "jdbc:clickhouse://mydomainiscool.cloud:8443/default?ssl=true"
    tableName: "iotjetsonjson_local"

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


bin/pulsar-admin sinks stop --tenant public --namespace default --name jdbc-clickhouse-sink-iot
bin/pulsar-admin sinks delete --tenant public --namespace default --name jdbc-clickhouse-sink-iot
bin/pulsar-admin sinks restart --tenant public --namespace default --name jdbc-clickhouse-sink-iot

bin/pulsar-admin sinks create --archive ./connectors/pulsar-io-jdbc-clickhouse-2.8.0.nar --inputs iotjetsonjson --name jdbc-clickhouse-sink-iot --sink-config-file conf/clickhouseiot.yml --parallelism 1
bin/pulsar-admin sinks list --tenant public --namespace default
bin/pulsar-admin sinks get --tenant public --namespace default --name jdbc-clickhouse-sink-iot
bin/pulsar-admin sinks status --tenant public --namespace default --name jdbc-clickhouse-sink-iot

bin/pulsar-client consume "persistent://public/default/iotjetsonjson" -s iotjetsonjson-reader


```
## Example Python3 Script

```
pip3 install clickhouse-driver

from clickhouse_driver import Client
client = Client('server.server.altinity.cloud', user='admin', password='password123456', port=9440, secure='y', verify=False)

client.execute("INSERT INTO stocks VALUES('ABC', '6bec81343434c6', 1634912880810, 1611327960000, '2021/05/22 10:06:00', '333.83099','355.38000','366.38000','2198','377.83099')")

```

## References

* https://github.com/tspannhw/SmartStocks
* https://github.com/tspannhw/FLiP-SQL/
* https://docs.altinity.com/altinitycloud/quickstartguide/yourfirstqueries/
* https://clickhouse.com/docs/en/sql-reference/functions/date-time-functions/
* https://clickhouse.com/docs/en/sql-reference/functions/type-conversion-functions/
* https://github.com/tspannhw/FLiP-CloudIngest
* https://github.com/tspannhw/StreamingAnalyticsUsingFlinkSQL
