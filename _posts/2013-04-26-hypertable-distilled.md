---
layout: post
title: "Hypertable Distilled"
description: "Hypertable Distilled"
category: 
tags: [Hypertable, HBase, BigTable]
---
## Preface

NoSQL is the most interesting parts in database system recently. There are two reason why use that - Scalability and Productivity. Many NoSQL Database have those features and difficult to consider to choose one. Here is very powerful and sophisticate one, Hypertable based on Google BigTable, like to recommend to all. I will focus to how to use it in real time application, not only logging. Is it possible mission critical state something like user table no one believe that a big NoSQL database used to retrieve single row in CRUD. I hope this will be helpful to choose a database out of them when you wonder.

**Note**: This paper approaches how to programm and implement in a real application, if I get a chance next time will see to benchmark performance of distributed environments.

**Tip**: What is the DBMS and How to store data shortly?
- ** RDBMS & ORDBMS : structured and object-oriented data
- ** Document-oriented DBMS : semi-structured data like xml, json based on documents
- ** Graph DBMS : edges and nodes which have properties data represented graph
- ** Column-oriented DBMS : atttribute data
- ** Typically row-oriented (=row based) database is mentioned to Relational DBMS & ORDBMS and column-oriented (= column based) & document-oriented is mentioned to NoSQL database.

## Contents

- **Preface**
- **Contents**
- **Abstract**
- **Design**

  0. System overview
  1. Key and Value
  2. Access group
  3. Secondary indices

- **Installation**
- **Implementation**
- **Benchmark released (Hypertable vs HBase)**
- **Summary**
- **References**

## Abstract

Hypertable is a high-performance alternative to HBase on Hadoop. The essential characteristics of Hypertable are quite similar to HBase, which in turn is a clone of the Google Bigtable. Hypertable is actually not a new project. It started around the same time as HBase in 2007. Hypertable runs on top of a distributed filesystem like HDFS.

In HBase, column-family-centric data is stored in a row-key sorted and ordered manner by Map/Reduce. The each cell of data maintains multiple versions of data. In Hypertable all version information is appended to the row-keys. The version information is identified via timestamps. All data for all versions for each row-key is stored in a sorted manner for each column-family and is supported multiple indices.
*1 reference source : Professional NoSQL by Shashank Tiwari

## Design

### System overview

[http://hypertable.com/uploads/Overview.png]
*2 reference source : [http://hypertable.com](http://hypertable.com)

**Hypersapce** is a filesystem for locking and storing small amounts of metadata means the root of all distributed data structures. *Master* has all meta information of operations such as creating and deleting tables. *Master* has not a single point of failure as client data does not move through it for short periods of time when switch standby to primary. *Master* is also responsible for range server load balancing. *Range server* manage ranges of table data, all reading and writing data. *DFS Broker* has the interface to *Hypersapce*. it translates normalized filesystem requests into native filesystem requests for HDFS, MapReduce, local and so on and vice-versa.

### Key and Value

[<img src="http://edydkim.github.com/assets/images/Data_Representation.jpg">]

Here are some check points to need to know.
**Control** describes the format of remaining fields. ex) HAVE REVISION = 0x80, HAVE TIMESTAMP = 0x40, AUTO TIMESTAMP = 0x20, SHARED = 0x10, TS_CHRONOLOGICAL = 0x01
**flag** is a logical delete flag to be garbage collected.

### Access group

Would you have head about Partitioning in some RDBMS? The **Access group** provide the all data defined the access group to store the same disk together. It can be limited Disk I/O and the frequency by reducing the amount of data transferred from disk during query execution.

{% capture text %}
create table User (
name
, department
, position
access group default(name, position)
, access group department(department)
)
{% endcapture %}
{% include JB/liquid_raw %}
+Note+: The column family 'department' can belong to only one access group. If the column family belong to one more access group HYPERTABLE HQL parse error is occured.

Run a query below then, check elaspsed time and throughput.
{% capture text %}
select department from User;
{% endcapture %}
{% include JB/liquid_raw %}

### Secondary indices

It is very important that the table can have one more indices or not about each a single column family. Hypertable has two types of indices: a cell value index and qualifier index. *A cell value index* scans on a single column family that do an exact match or prefix match of the value, and *qualifier index* scans on same as a cell valud index but of the column qualifier.

**Note**: The indices is stored index table in same namespace as the primary table. You need to consider additional disk storage for indexing data and cause a very small performance impact of side effect for inserting data.

How to define and see below:
{% capture text %}
create table User (
name
, department
, position
, index name
, index department
, qualifier index department
, index position
);
{% endcapture %}
{% include JB/liquid_raw %}
{% capture text %}
hypertable> create table User (
-> name
-> , department
-> , position
-> , index name
-> , index department
-> , qualifier index department
-> , index position
-> );

Elapsed time: 35.21 s
hypertable> insert into User values ('A00', 'name', 'I am'), ('A00', 'department', 'Sales'), ('A00', 'position', 'Manager');

Elapsed time: 0.05 s
Avg value size: 5.33 bytes
Total cells: 3
Throughput: 59.39 cells/s
Resends: 0
hypertable> insert into User values ('B00', 'name', 'You are'), ('B00', 'department:old', 'Human Resource'), ('B00', 'position', 'Senior Manager');

Elapsed time: 0.12 s
Avg value size: 11.67 bytes
Avg key size: 1.33 bytes
Throughput: 319.36 bytes/s
Total cells: 3
Throughput: 24.57 cells/s
Resends: 0
hypertable> select name from User where name = 'I am';
A00 name I am

Elapsed time: 0.07 s
Avg value size: 4.00 bytes
Avg key size: 4.00 bytes
Throughput: 121.82 bytes/s
Total cells: 1
Throughput: 15.23 cells/s
hypertable> select name from User where name =^ 'I';
A00 name I am

Elapsed time: 0.00 s
Avg value size: 4.00 bytes
Avg key size: 4.00 bytes
Throughput: 15009.38 bytes/s
Total cells: 1
Throughput: 1876.17 cells/s

hypertable> select department:old from User where department = 'Human Resource';
B00 department:old Human Resource

Elapsed time: 0.00 s
Avg value size: 14.00 bytes
Avg key size: 7.00 bytes
Throughput: 8183.94 bytes/s
Total cells: 1
Throughput: 389.71 cells/s
hypertable> select department:old from User;
B00 department:old Human Resource

Elapsed time: 0.00 s
Avg value size: 14.00 bytes
Avg key size: 7.00 bytes
Throughput: 17632.24 bytes/s
Total cells: 1
Throughput: 839.63 cells/s
hypertable> select department:^o from User;
B00 department:old Human Resource

Elapsed time: 0.00 s
Avg value size: 14.00 bytes
Avg key size: 7.00 bytes
Throughput: 39325.84 bytes/s
Total cells: 1
Throughput: 1872.66 cells/s
{% endcapture %}
{% include JB/liquid_raw %}

## Installation

Hypertable standalone can downlaod here: [http://hypertable.com/download/0971/](http://hypertable.com/download/0971/)
(this paper is tried Hypertable Binary Version 0.9.7.1)
Before follow steps below, you better to make a user as owner.

Install Filesystem Hierarchy Standard the directories.
{% capture text %}
$ sudo mkdir /etc/opt/hypertable /var/opt/hypertable
$ sudo chown hypertable:hypertable /etc/opt/hypertable /var/opt/hypertable

$ /opt/hypertable/$VERSION/bin/fhsize.sh
{% endcapture %}
{% include JB/liquid_raw %}

Set current link.
{% capture text %}
$ cd /opt/hypertable
$ ln -s $VERSION current
{% endcapture %}
{% include JB/liquid_raw %}

Edit hypertable configuration file.
My local configuration file for experiment is below :
{% capture text %}
cat /opt/hypertable/current/conf/hypertable.cfg [~]
#
# hypertable.cfg
#

# HDFS Broker
HdfsBroker.Hadoop.ConfDir=/etc/hadoop/conf

# Ceph Broker
CephBroker.MonAddr=10.0.1.245:6789

# Local Broker
DfsBroker.Local.Root=fs/local

# DFS Broker - for clients
DfsBroker.Port=38030

# Hyperspace
# for one instance only
Hyperspace.Replica.Host=localhost
# for multi instance repli
# Hyperspace.Replica.Hostt=PC-7370.local or pc-2473.cyberagent.co.jp
Hyperspace.Replica.Port=38040
Hyperspace.Replica.Dir=hyperspace

# Hypertable.Master
Hypertable.Master.Port=38050

# Hypertable.RangeServer
Hypertable.RangeServer.Port=38060

Hyperspace.KeepAlive.Interval=30000
Hyperspace.Lease.Interval=1000000
Hyperspace.GracePeriod=200000

# ThriftBroker
ThriftBroker.Port=38080
{% endcapture %}
{% include JB/liquid_raw %}

make a rc script like this:
{% capture text %}
#!/bin/sh /usr/local/etc/hypertable.sh

HYPERTABLE_BASE_DIR="/opt/hypertable/current/bin/"
__start( ) {
${HYPERTABLE_BASE_DIR}start-all-servers.sh local
}

__stop( ) {
${HYPERTABLE_BASE_DIR}stop-servers.sh
}


__restart( ) {
${HYPERTABLE_BASE_DIR}stop-servers.sh && ${HYPERTABLE_BASE_DIR}start-all-servers.sh local
}

##
# :: main ::
case "${1}" in
start|stop|restart)
# [ -x ${HYPERTABLE_DAEMON} ] || exit 2
__${1}
;;
*)
;;
esac
{% endcapture %}
{% include JB/liquid_raw %}

set environment:
{% capture text %}
# for hypertable
export PATH=/opt/hypertable/current/lib/java/:$PATH

# aliases
alias ht="/opt/hypertable/current/bin/ht"
{% endcapture %}
{% include JB/liquid_raw %}

then, start and test:
{% capture text %}
$ /usr/local/etc/hypertable.sh start [~]
DFS broker: available file descriptors: 256
Started DFS Broker (local)
Started Hyperspace
Started Hypertable.Master
Started Hypertable.RangeServer
Started ThriftBroker

$ ht shell
Welcome to the hypertable command interpreter.
For information about Hypertable, visit http://hypertable.com

Type 'help' for a list of commands, or 'help shell' for a
list of shell meta commands.

hypertable> use '/';

Elapsed time: 0.00 s
hypertable> get listing;
Tutorial (namespace)
sys (namespace)
test (namespace)
tmp (namespace)

Elapsed time: 0.00 s
hypertable> use test;

Elapsed time: 0.01 s
hypertable> show tables;
{% endcapture %}
{% include JB/liquid_raw %}

finally, shutdown it:
{% capture text %}
$ /usr/local/etc/hypertable.sh stop [~]
Killing ThriftBroker.pid 30946
Shutdown master complete
Sending shutdown command
Shutdown range server complete
Sending shutdown command to DFS broker
Killing DfsBroker.local.pid 30729
Killing Hyperspace.pid 30725
Shutdown thrift broker complete
Shutdown hypertable master complete
Shutdown DFS broker complete
Shutdown hyperspace complete
{% endcapture %}
{% include JB/liquid_raw %}

## Implementation

Look at source first. You, engineer, love it as nothing better than understanding by source.
So, how to use this for work on actual fields.

Let's try each methods of retreiving data below:

### 0. Prerequisite

### 1. HQL Query

### 2. Shared Mutex

### 3. Asynchronization

### 4. Put them into one table togather

### 5. Delete all or each value

