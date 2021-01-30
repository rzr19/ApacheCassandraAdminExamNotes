# Apache Cassandra Admin Exam Notes
Apache Cassandra 3.x Administrator Associate Certification Exam Notes

### DS201: DataStax Enterprise 6 Foundations of Apache Cassandra - https://academy.datastax.com/resources/ds201-datastax-enterprise-6-foundations-of-apache-cassandra ###
* Partitions
  * Group rows physically together on disk based on the partition key.
  * The partitioner hashes the partition key values to create a partition token.
  * Used to place data on the ring.
  * https://www.datastax.com/dev/blog/the-most-important-thing-to-know-in-cassandra-data-modeling-the-primary-key
  * https://docs.datastax.com/en/archived/cql/3.3/cql/cql_using/useCompositePartitionKeyConcept.html
* Clustering Columns
  * Non-partition section of the PK.
  * Clustering columns order data within a partition. https://docs.datastax.com/en/dse/5.1/cql/cql/cql_using/whereClustering.html
  * Equality or range queries can be performed on clustering columns.
  * Default ascending order.
  * https://docs.datastax.com/en/dse/6.7/cql/cql/cql_using/useCompoundPrimaryKeyConcept.html
  * https://www.bmc.com/blogs/cassandra-clustering-columns-partition-composite-key/
  * 2+ inserts on the same PK will be an upsert i.e an update
* Application Connectivity
  * Drivers - https://docs.datastax.com/en/driver-matrix/doc/driver_matrix/common/driverMatrix.html
* Node
  * 6K-12K transactions/seconds/core
  * 2-4TB
* Ring
  * Nodes can have four statuses in the ring UP / DOWN / JOINING / LEAVING.
  * https://academy.datastax.com/units/2017-ring-dse-foundations-apache-cassandra?resource=ds201-datastax-enterprise-6-foundations-of-apache-cassandra
* Peer to Peer
  * No one is a leader, no one is a follower. All nodes are equal.
* Vnodes
  * Vnodes help keep a cluster balanced as new Apache Cassandra nodes are introduced to the cluster and old nodes are decommissoned, and also automate token range assignment. With vnodes, each node is responsible for several smaller slices of the ring, instead of just one large slice.
  * default number of vnodes = 128.
  * Can be configured in the num_tokens parameter in cassandra.yaml.
  * https://docs.datastax.com/en/archived/cassandra/3.0/cassandra/architecture/archDataDistributeVnodesUsing.html
  * https://www.youtube.com/watch?v=G4SMNU1aOJg
* Gossip - nodetool gossipinfo
  * A Gossip round is initiated by a Node every 1 second. Nodes pick 1-3 others to gossip with.
  * Can gossip with any Node but favours (slightly) seeds and downed Nodes.
  * https://www.edureka.co/blog/gossip-protocol-in-cassandra/
  * STATUS, DC, RACK, SCHEMA, LOAD, EP, HB, etc.  
* Snitch
  * Which one is configured in cassandra.yaml
  * SimpleSnitch is the default for RAC=1 and DC=1
  * GossipingPropertyFileSnitch < cassandra-rackdc.properties file
  * PropertyFileSnitch < cassandra-topology.properties file
  * RackInferringSnitch
* Replication
  * RF = 1,2 or 3 (copies of node data on neighbour nodes)
  * it is a keyspace parameter
* Consistency
  * Coordinator node sends a direct read request to one node and digest requests to remainder ones to meet CL level
  * ANY - Store a hint. Do not use.
  * ONE, TWO, THREE - Closest to Coordinator
  * QUORUM - Majority vote 51%. 2 is quorum for 3. 3 for 4. 3 for 5.
  * LOCAL_ONE - Limit to local DC.
  * LOCAL_QUORUM Majority vote in local DC.
  * EACH_QUORUM - Majority vote in each DC.
  * ALL - All nodes must participate. Worst performance.
* Hinted Handoff
  * Enabled by default for 3 hours.
  * settings in cassandra.yaml for hints
* Read Repair
  * Cassandra concept to re-sync nodes from time to time.
  * Always occurs when consistency level = ALL.
  * read_repair_chance - probability for read repair with CL other than ALL.
* Node Sync
   DataStax Enterprise 6 feature.
* Write Path
  * MemTable ordered by PK/CC but we always append to end of CommitLog
  * Flush the MemTable to SSTable on disk from time to time.
  * You can have more than one SSTable for same op on same ring PK
  * ./nodetool cfstats for more on this
* Read Path
  * Partition Summary - RAM structure storing byte offset references into the partition index. So ranges of tokens in ram to redirect to to an index entry token on disk in SSTable.
  * The key cache - Stores the byte offset of the most recently accessed records.
  * Bloom Filter - A Bloom filter is a space-efficient probabilistic data structure, conceived by Burton Howard Bloom in 1970, that is used to test whether an element is a member of a set. A query returns either "possibly in set" or "definitely not in set".
  * http://cassandra.apache.org/doc/latest/operating/bloom_filters.html
* Compaction
  * Process used to remove stale data from existing sstables based on timestamps
  * Think of it like a merge of tables with oldest duplicates and older sstables  getting dropped
  * Compaction Strategies
    * SizeTiered Compaction - Default. Triggers when multiple sstables of a similar size are present. Good for high writes.
    * Leveled Compaction - Groups sstables into levels. Each level has a fixed size limit which is 10 times larger than the previous level. Good for read heavy use-cases.
    * TimeWindow Compaction - Create time windowed buckets of sstables that are compacted using the Size Tiered compaction strategy.
  * A tombstone is a delete with a timestamp newer than its duplicate record
  * Tombstones older than gc_grace_seconds period are removed
  * ./nodetool flush - memtables to disk
  * Advanced Performance
    * DSE considerations


### DS210: DataStax Enterprise 6 Operations with Apache Cassandra -https://academy.datastax.com/resources/ds210-datastax-enterprise-6-operations-with-apache-cassandra ###
* Configuring Clusters
  * cassandra.yaml - Main configuration file.
  * cluster_name - Default "Test Cluster".
  * listen_address - Default localhost.
  * native_transport_address - Default localhost.
  * seeds - Default "127.0.0.1".
  * endpoint_snitch - Default SimpleSnitch.
  * initial_token - Default 128 (or is it 256?? Some confusion https://datastaxacademy.slack.com/archives/C4ZQ1EWNM/p1560929813009900).
  * commitlog_directory - Default /var/lib/cassandra/commitlog.
  * data_file_directories - Default /var/lib/cassandra/data.
  * hints_direcory - Default /var/lib/cassandra/hints.
  * saved_caches_directory - Default /var/lib/cassandra/saved_caches.
* Cluster Sizing
  * Throughput.
  * Growth Rate.
  * Latency.
  * read/write ratio
* cassandra-stress
  * Benchmarking tool used to determine schema performance, scaling and determine production capacity.
  * Configured through a yaml file.
  * Example here - https://github.com/justinbreese/dse-cassandra-stress/blob/master/stress.yaml
* dstat - combines iostat vmstat ifstat
  * Versatile tool for generating system resource statistics.
  * https://linux.die.net/man/1/dstat
  * dstat -am - All default stats and memory.
* nodetool
  * A command line interface for managing a cluster.
  * http://cassandra.apache.org/doc/latest/tools/nodetool/nodetool.html
  * https://docs.datastax.com/en/archived/cassandra/3.0/cassandra/tools/toolsNodetool.html
  * https://www.youtube.com/watch?v=0qr9z0lsbuk
  * https://www.youtube.com/watch?v=Sz7OiUWgs5U
* logging & JVM GC activity
  * nodetool getlogginglevels
  * /var/log/cassandra/system.log also a debug.log 
  * logback.xml same as for Tomcat
  * /etc/dse/cassandra/jvm.options 
* Adding Nodes
  * Add a single node at a time.
  * https://docs.datastax.com/en/archived/cassandra/3.0/cassandra/operations/opsAddNodeToCluster.html
* Removing Nodes
  * nodetool decommission - Causes a live node to decommission itself, streaming its data to the next node on the ring to replicate appropriately.
  * nodetool removenode - Shows the status of current node removal; forces completion of pending removal, or removes identified node. Use when the node is down and nodetool decommission cannot be used. If the cluster does not use vnodes, adjust the tokens before running this command.
  * nodetool assassinate - Just make the node go away. Doesn't redistribute any data.
  * https://docs.datastax.com/en/archived/cassandra/3.0/cassandra/operations/opsRemoveNode.html
* Bootstrapping
  * The process of a new node joining a cluster.
  * http://cassandra.apache.org/doc/latest/operating/topo_changes.html
  * https://thelastpickle.com/blog/2017/05/23/auto-bootstrapping-part1.html
  * https://thelastpickle.com/blog/2018/08/02/Re-Bootstrapping-Without-Bootstrapping.html
  * https://de.slideshare.net/ArunitGupta1/boot-strapping-in-cassandra
* Replacing a Downed Node
  * Replace better than remove and add.
  * In jvm.options add replace_address or (better) replace_address_first_boot.
  * Monitor with nodetool netstats.
* Size Tiered Compaction
  * Default compaction strategy.
  * Good for insert heavy and general workloads.
  * Groups similarly sized tables together.
  * Tiers with less than min_threshold (four) sstables are not considered for compaction.
* Leveled Compaction
  * Groups sstables into levels of a fixed size limit.
  * Each level is 10 times larger than the previous.
  * Best for read heavy workloads or where there are more updates than inserts.
  * Very IO intensive.
  * Compacts more frequently than size tiered.
* Time Window Compaction
  * Designed to work on time-series data.
  * One sstable for each time window.
  * size tiered within a time window.
* Repair
  * Synchronizing replicas.
  * Occurs when detected by reads, randomly with non_quorum reads (read_repair_chance) or manually with nodetool repair.
  * Repairs should be run if a node is down for a long time.
  * Regular repair should also be run to ensure the integrity of cluster data.
* Nodesync
  * DSE only feature.
  * Background, low-overhead repair like feature.
* sstablesplit
  * Splits SSTable files into multiple SSTables of a maximum designated size.
  * Do not execute when Cassandra is running.
  * We might sometimes wish to split a very large sstable to ensure compaction occurs on it.
  * sstablesplit -m 500 /path/to/massive/sstable-Data.db
* CQL COPY
  * Built in command to copy data in and out of Cassandra.
  * https://docs.datastax.com/en/archived/cql/3.3/cql/cql_reference/cqlshCopy.html
  ```sql
  COPY cycling.cyclist_name (id,firstname)
  FROM '../cyclist_firstname.csv'
  WITH HEADER = TRUE ;
  ```
* sstabledump
  * This tool outputs the contents of the specified SSTable in the JSON format.
  * https://docs.datastax.com/en/archived/cassandra/3.0/cassandra/tools/ToolsSSTabledump.html
* sstableloader
  * Bulk load external data into a cluster.
  * Load existing SSTables into another cluster with a different number of nodes or replication strategy.
  * Restore snapshots.
  * https://docs.datastax.com/en/archived/cassandra/3.0/cassandra/tools/toolsBulkloader.html
* DSBulk
  * The DataStax Bulk Loader dsbulk can be used for both loading data from a variety of sources and unloading data from DataStax Enterprise (DSE) or DataStax Distribution of Apache Cassandra™ (DDAC) databases for transfer, use, or storage of data.
  * More performant than CQL COPY when you have TB of data.
  * https://docs.datastax.com/en/dsbulk/doc/dsbulk/reference/dsbulkCmd.html
* Backup
* JVM
  * cassandra-env.sh - jvm.options are in this shell script.
  * max_heap_size of 8
  * G1 gc default since java9
* Garbage Collection
* Heap Dump
* Kernel Tuning
  * ulimit -a - Allow Cassandra unfettered access to system resources.
  * Turn off swap.
  * https://docs.datastax.com/en/dse/6.7/dse-dev/datastax_enterprise/config/configRecommendedSettings.html
  * https://tobert.github.io/pages/als-cassandra-21-tuning-guide.html
* Hardware
  * Persistent storage type. Avoid SAN / NAS / NFS.
  * Memory.
  * CPU.
  * Number of nodes.
  * Network bandwidth.
* Cloud
* Security
* OpsCenter
  * browser based DSE cluster tool for monitoring, configuring, management
  * Backup Service with lots of features
  * Best Practice service like Azure Advisor
  * Capacity Service, Performance Service
  * has a dark theme ++

### DS220: DataStax Enterprise 6 Practical Application Data Modeling with Apache Cassandra
* Data Types
  * there is an IP one inet with IPv4/IPv6 validation
* Clustering columns
  * by default ASC and must be used in order of DDL query with PK #1 bcuz full table scans
* Denormalization
  * no JOIN statements but object_by_object tables
  * it's OK to have duplicate data if you need data when you need it and got the $$
* Collections
  * set: {'email1@d.d','email2@d.d'}
  * list: ['berlin','london','france'] - can duplicate, ordered
  * map: {'key1' : 'val1', 'key2' : 'val2'}
  * use FROZEN keyword to nest datatypes and turn cells to blobs
* UDTs
  * create type full_name ( first_name text, last_name text );
* Counters
  * 64 bit signed it, changed incrementally, by UPDATE, need dedicated tables
  * update foo_counts set foo_count = foo_count + 8 where foo_name = 'foox';
  * default 0, cannot insert
* UDFs and UDAs
  * create or replace function avgState (foo, bar) .. language java .. 
* Write Techniques
  * maintain consistency with log BATCH DMLs on related tables
  * BEGIN BATCH .. INSERT/UPDATE/DELETE .. APPLY BATCH;
  * Lightweight transactions - condition check before DML /* INSERT INTO xx () values () IF NOT EXISTS; || IF reset_token = 'foo';
* Read Techniques
  * secondary indexes

### Quiz Trivia ###

* What is the node that handles a request called? Coordinator node.
* Write Path Order Commitlog > MemTable > SSTable.
* Cassandra does not do any writes or deletes in place.
* SSTables are immutable.
* Compaction is the progress of taking small SSTables and merges them into bigger ones.
* Last writes wins - based on Timestamps.
