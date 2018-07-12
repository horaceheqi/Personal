使用Python获取Hbase和Hive数据
---
### 1、连接Hbase
- 在Windows环境下进行会有很多库莫名其妙安装不了，本案例在linux环境下进行
- Python 2.7 和 Python 3.5 以上环境均可以
- 利用 `happybase`的 `thrift` 库实现
#### 步骤如下：
首先要启动hbase thrift server

```
habse thrift start &
```

然后查看9090端口是否启用

```
netstart -nl | grep 9090
```

最后编写Python代码

```
import happybase 
connection = happybase.Connection('你的ip地址哈') 
table = connection.table('Hbase表名') 
for key, data in table.scan(limit=10000,batch_size=10): 
    print key, data
```

### 2、连接Hive
- 在Windows环境下进行会导致很多库莫名其妙安装不了，本案例在linux环境下进行
- 在连接hive时，可以选择`thrift`本身，`pyhive`，`pyhs2`和`impyla`。在使用过程中首先尝试了`thrift`本身，但是通过配置后在连接返回：`thrift.transport.TTransport.TTransportException: None`，这是连接`hiveserver2`出现的问题，因此弃用`thrift`直接连接hive，选择其他三个。`pyhs2`是以前hive官方推荐使用的库，主要依赖了`thrift`和`sasl`，但是这个库后面没有维护了，因此在最新的python连接hive下有很多问题，并且pyhs2支持支linux环境。`impyla`是通过`impala`来对操作hive，需要有`impala`环境。
- 用 `Python 3` 以上版本会出现很多莫名其妙的错误，因此采用 `Python 2`
- 需要启动 HiveServer2 服务(两种启动方式) `hive --service hiveserver2 &`  `hiveserver2 &`
- 监听 HiveServer2 端口 `netstat -nl |grep 10002`
