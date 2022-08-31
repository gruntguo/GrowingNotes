

统计各时序数据库的代码量：

* vm的集群版=cluster分支，单机版=master分支；
* influxdb是自有版本；
* tdengine是master分支；



|                 | 语言   | 集群版(line)       | 单机版(line)      |
| --------------- | ------ | ------------------ | ----------------- |
| victoriametrics | golang | 146,272 / 123,466  | 140,572 / 118,911 |
| influxdb        | golang | 218,717 / 171, 634 |                   |
| tdengine        | C      | 502,018 / 346, 415 |                   |

