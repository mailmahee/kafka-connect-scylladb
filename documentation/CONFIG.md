#ScyllaDB Sink Connector

Configuration Properties
------------------------

To use this connector, specify the name of the connector class in the ``connector.class`` configuration property.

    connector.class=io.connect.scylladb.ScyllaDbSinkConnector

Connector-specific configuration properties are described below.

###Connection

``scylladb.contact.points``

  The ScyllaDB hosts to connect to. Scylla nodes use this list of hosts to find each other and learn the topology of the ring. 
  You must change this if you are running multiple nodes.
  It's essential to put at least 2 hosts in case of bigger cluster, since if first host is down, 
  it will contact second one and get the state of the cluster from it.
  Eg. When using the docker image, connect to the host it uses. 
  To connect to private Scylla nodes, provide a JSON string having all internal private network address:port mapped to
  an external network address:port as key value pairs. Need to pass it as 
  {\"private_host1:port1\",\"public_host1:port1\",\"private_host2:port2\",\"public_host2:port2\", ...}
  Eg. {\"10.0.24.69:9042\": \"sl-eu-lon-2-portal.3.dblayer.com:15227\", \"10.0.24.71:9042\": \"sl-eu-lon-2-portal.2.dblayer.com:15229\", \"10.0.24.70:9042\": \"sl-eu-lon-2-portal.1.dblayer.com:15228\"}
  
  * Type: List
  * Importance: High
  * Default Value: [localhost]
  
``scylladb.port``
 
  The port the ScyllaDB hosts are listening on.
  Eg. When using a docker image, connect to the port it uses(use docker ps)
   
  * Type: Int
  * Importance: Medium
  * Default Value: 9042
  * Valid Values: ValidPort{start=1, end=65535}
  
``scylladb.loadbalancing.localdc``

  The case-sensitive Data Center name local to the machine on which the connector is running. 
  It is a recommended configuration if we have more than one DC.

  * Type: string
  * Default: ""
  * Importance: high

``scylladb.security.enabled``
 
  To enable security while loading the sink connector and connecting to ScyllaDB.
  
  * Type: Boolean
  * Importance: High
  * Default Value: false
  
``scylladb.username``
  
  The username to connect to ScyllaDB with. Set ``scylladb.security.enable = true`` to use this config.
  
  * Type: String
  * Importance: High
  * Default Value: cassandra
  
``scylladb.password``
  
  The password to connect to ScyllaDB with. Set ``scylladb.security.enable = true`` to use this config.
  
  * Type: Password
  * Importance: High
  * Default Value: cassandra
  
``scylladb.compression``
  
  Compression algorithm to use when connecting to ScyllaDB.
  
  * Type: string
  * Default: NONE
  * Valid Values: [NONE, SNAPPY, LZ4]
  * Importance: low
  
``scylladb.ssl.enabled``
  
  Flag to determine if SSL is enabled when connecting to ScyllaDB.
  
  * Type: boolean
  * Default: false
  * Importance: high
  
###SSL

``scylladb.ssl.truststore.path``

  Path to the Java Truststore.

  * Type: string
  * Default: ""
  * Importance: medium

``scylladb.ssl.truststore.password``

  Password to open the Java Truststore with.

  * Type: password
  * Default: [hidden]
  * Importance: medium

``scylladb.ssl.provider``

  The SSL Provider to use when connecting to ScyllaDB.

  * Type: string
  * Default: JDK
  * Valid Values: [JDK, OPENSSL, OPENSSL_REFCNT]
  * Importance: low

###Keyspace

**Note**: Both keyspace and table names consist of only alphanumeric characters, 
cannot be empty and are limited in size to 48 characters (that limit exists 
mostly to avoid filenames, which may include the keyspace and table name, 
to go over the limits of certain file systems). By default, keyspace and table names 
are case insensitive (myTable is equivalent to mytable) but case sensitivity 
can be forced by using double-quotes ("myTable" is different from mytable).

``scylladb.keyspace``

  The keyspace to write to. This keyspace is like a database in the ScyllaDB cluster.
  * Type: String
  * Importance: High

``scylladb.keyspace.create.enabled``

  Flag to determine if the keyspace should be created if it does not exist.
  **Note**: Error if a new keyspace has to be created and the config is false.

  * Type: Boolean
  * Importance: High
  * Default Value: true

``scylladb.keyspace.replication.factor``
    
  The replication factor to use if a keyspace is created by the connector. 
  The Replication Factor (RF) is equivalent to the number of nodes where data (rows and partitions) 
  are replicated. Data is replicated to multiple (RF=N) nodes
  
  * Type: int
  * Default: 3
  * Valid Values: [1,...]
  * Importance: high

###Table

``scylladb.table.manage.enabled``

  Flag to determine if the connector should manage the table.

  * Type: Boolean
  * Importance: High
  * Default Value: true
  
``scylladb.table.create.compression.algorithm``
  
  Compression algorithm to use when the table is created.

  * Type: string
  * Default: NONE
  * Valid Values: [NONE, SNAPPY, LZ4, DEFLATE]
  * Importance: medium

``scylladb.offset.storage.table``

  The table within the ScyllaDB keyspace to store the offsets that have been read from Apache Kafka.

  * Type: String
  * Importance: Low
  * Default: kafka_connect_offsets
  
###Topic to Table

These configurations can be specified for multiple Kafka topics from which records are being processed. 
Also, these topic level configurations will be override the behavior of Connector level configurations such as 
``scylladb.consistency.level``, ``scylladb.deletes.enabled`` and ``scylladb.ttl``

``topic.my_topic.my_ks.my_table.mapping``

  For mapping topic and fields from Kafka record's key, value and headers to ScyllaDB table and its columns.
  
  **Note**: Ensure that the data type of the Kafka record's fields are compatible with the data type of the ScyllaDB column.
  In the Kafka topic mapping, you can optionally specify which column should be used as the ttl (time-to-live) and 
  timestamp of the record being inserted into the database table using the special property __ttl and __timestamp.
  By default, the database internally tracks the write time(timestamp) of records inserted into Kafka. 
  However, this __timestamp feature in the mapping supports the scenario where the Kafka records have an explicit 
  timestamp field that you want to use as a write time for the database record produced by the connector. 
  Eg. "topic.my_topic.my_ks.my_table.mapping": 
  "column1=key.field1, column2=value.field1, __ttl=value.field2, __timestamp=value.field3, column3=header.field1"
  
``topic.my_topic.my_ks.my_table.consistencyLevel``

  By using this property we can specify table wide consistencyLevel.

``topic.my_topic.my_ks.my_table.ttlSeconds``

  By using this property we can specify table wide ttl(time-to-live).

``topic.my_topic.my_ks.my_table.deletesEnabled``

  By using this property we can specify if tombstone records(records with Kafka value as null) 
  should processed as delete request.


###Write

``scylladb.consistency.level``

  The requested consistency level to use when writing to ScyllaDB. The Consistency Level (CL) determines how many replicas in a cluster that must acknowledge read or write operations before it is considered successful.

  * Type: String
  * Importance: High
  * Default Value: LOCAL_QUORUM
  * Valid Values: ``ANY``, ``ONE``, ``TWO``, ``THREE``, ``QUORUM``, ``ALL``, ``LOCAL_QUORUM``, ``EACH_QUORUM``, ``SERIAL``, ``LOCAL_SERIAL``, ``LOCAL_ONE``

``scylladb.deletes.enabled``

  Flag to determine if the connector should process deletes. 
  The Kafka records with kafka record value as null will result in deletion of ScyllaDB record 
  with the primary key present in Kafka record key.

  * Type: boolean
  * Default: true
  * Importance: high

``scylladb.execute.timeout.ms``

  The timeout for executing a ScyllaDB statement.

  * Type: Long
  * Importance: Low
  * Valid Values: [0,...]
  * Default Value: 30000

``scylladb.ttl``

  The retention period for the data in ScyllaDB. After this interval elapses, ScyllaDB will remove these records. If this configuration is not provided, the Sink Connector will perform insert operations in ScyllaDB  without TTL setting.

  * Type: Int
  * Importance: Medium
  * Default Value: null

``scylladb.offset.storage.table.enable``

  If true, Kafka consumer offsets will be stored in ScyllaDB table. If false, connector will skip writing offset 
  information into ScyllaDB.

  * Type: Boolean
  * Importance: Medium
  * Default Value: True
  
###ScyllaDB

``behavior.on.error``

  Error handling behavior setting. Must be configured to one of the following:

  ``fail``
  The Connector throws ConnectException and stops processing records when an error occurs while processing or inserting records into ScyllaDB.

  ``ignore``
  Continues to process next set of records when error occurs while processing or inserting records into ScyllaDB.
  
  ``log``
  Logs the error via connect-reporter when an error occurs while processing or inserting records into ScyllaDB and continues to process next set of records, available in the kafka topics.

  * Type: string
  * Default: FAIL
  * Valid Values: [FAIL, LOG, IGNORE]
  * Importance: medium


###Confluent Platform Configurations.

``tasks.max``

The maximum number of tasks to use for the connector that helps in parallelism.

* Type:int
* Default: 1
* Importance: high

``topics``

The name of the topics to consume data from and write to ScyllaDB.

* Type: list
* Importance: high

``confluent.topic.bootstrap.servers``

 A list of host/port pairs to use for establishing the initial connection to the Kafka cluster used for licensing. All servers in the cluster will be discovered from the initial connection. This list should be in the form <code>host1:port1,host2:port2,…</code>. Since these servers are just used for the initial connection to discover the full cluster membership (which may change dynamically), this list need not contain the full set of servers (you may want more than one, though, in case a server is down).

* Type: list
* Default: localhost:9092
* Importance: high

------------------------
      

