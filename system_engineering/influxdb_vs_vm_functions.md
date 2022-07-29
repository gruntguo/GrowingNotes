

influxdb与victoriametrics的功能对比。



### 一.写入



均支持行协议的写入：

```
foo,tag1=value1,tag2=value2 field1=12,field2=40
bar,tag1=value3,tag2=value4 field1=34,field2=60
```



influxdb行协议写入：

```
curl -i -XPOST 'http://localhost:8086/write?db=prometheus&precision=ms' --data-binary \
'foo,tag1=value1,tag2=value2 field1=12,field2=40
bar,tag1=value3,tag2=value4 field1=34,field2=60'
```



vm行协议写入：

```
curl -i -XPOST 'http://localhost:8480/insert/0/influx/write' --data-binary \
'foo,tag1=value1,tag2=value2 field1=12,field2=40
bar,tag1=value3,tag2=value4 field1=34,field2=60'
```

vm写入后，将被转成如下的datapoints:

```
foo_field1{tag1="value1", tag2="value2"} 12
foo_field2{tag1="value1", tag2="value2"} 40
```



vm除了支持行协议写入，还支持remote-write、opentsdb格式、json、csv等格式。



### 二.查询



influxdb仅支持influxQL查询语法，不支持PromQL:

```
> precision rfc3339
> use prometheus;
Using database prometheus
>
> select * from up order by time desc limit 10;
name: up
time                      job                     namespace                     pod                                   value
----                      ---                     ---------                     ---                                   -----
2022-07-28T02:03:57.22Z   alertmanager-main       kubesphere-monitoring-system  alertmanager-main-0                   1
2022-07-28T02:03:55.899Z  kube-scheduler          kube-system                   kube-scheduler-node2                  1
2022-07-28T02:03:55.16Z   prometheus-operator     kubesphere-monitoring-system  prometheus-operator-5cd568cc6c-27xhw  1
2022-07-28T02:03:50.572Z  kube-controller-manager kube-system                   kube-controller-manager-node2         1
```

```
curl -G 'http://localhost:8086/query' --data-urlencode "db=prometheus" --data-urlencode "q=SELECT * FROM \"up\" order by time desc limit 10"
```



vm不支持influxQL查询语法，支持兼容PromQL:

```
# instant query
curl http://localhost:8481/select/0/prometheus/api/v1/query?query=up

# range query
curl http://localhost:8481/select/0/prometheus/api/v1/query_range?query=up&start=1659061714.753&end=1659063514.753&step=3.75
```



### 三. 保留时间



influxdb支持为database配置retention policy：

* retention policy中配置了保留时间，副本数；
* retention policy可以通过HTTP API进行更改；

```
> show retention policies;
name     duration shardGroupDuration replicaN default
----     -------- ------------------ -------- -------
one_week 168h0m0s 24h0m0s            2        true
```



vm支持配置retention period：

* vmstorage节点的启动参数中，配置-retentionPeriod；
* 不支持API修改；



### 四. 降采样(downsampling)



influxdb支持通过CQ(Continuous Query)进行指标的降采样;

比如：将cpu.user数据进行5min平均：

```
CREATE CONTINUOUS QUERY "cq_5" ON falcon 
BEGIN 
SELECT mean(value) INTO falcon.autogen."5_cpu.user" FROM falcon.autogen."cpu.user" GROUP BY time(5m), * 
END
```



vm仅商业版本支持：

* -downsampling.period=30d:5m，对30day之前的数据，以5min为interval计算采样值；





### 参考

1.vm支持的格式：https://docs.victoriametrics.com/?highlight=line%20protocol#how-to-import-time-series-data



