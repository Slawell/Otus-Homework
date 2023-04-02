# Homework 6

Необходимо:
 - развернуть Kubernetes кластер в облаке или локально (используя "ПРЕРЕКВИЗИТЫ.docx" из материралов);
 - поднять 3 узловый Cassandra кластер на Kubernetes (используя "How to Run Cassandra on Azure Kubernetes Service (AKS), part1.pdf" из материралов);
 - нагрузить кластер при помощи Cassandra Stress Tool (используя "How to use Apache Cassandra Stress Tool.pdf" из материралов).

## 1  Развернуть Kubernetes кластер в облаке или локально, поднять 3 узловый Cassandra кластер на Kubernetes

Сначала мы создадим сеть docker.
Все контейнеры cassandra docker будут находиться в этой сети.

- docker network create cassandra-network

Создадим первый узел Cassandra:
 - docker run --network  cassandra-network --name cassandra-node1 -d cassandra:4.0

 Cоздаем второй и третий и привязываем к первому:
 - docker run --network  cassandra-network --name cassandra-node2 -e CASSANDRA_SEEDS=cassandra-node1 -d cassandra:4.0
 - docker run --network  cassandra-network --name cassandra-node3 -e CASSANDRA_SEEDS=cassandra-node1 -d cassandra:4.0


  docker ps
| FCONTAINER ID |     IMAGE      |      COMMAND         |   CREATED    |      STATUS      |                PORTS                       |      NAMES       |
| ------------- | -------------  |     -------------    |------------- |  -------------   |             -------------                  |    ------------- |
| 5d6d0768d5ce  | cassandra:4.0  |"docker-entrypoint.s…"|3 minutes ago |Up About a minute |7000-7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp | cassandra-node3  |
| 198f9a09b3f6  | cassandra:4.0  |"docker-entrypoint.s…"|3 minutes ago |Up 3 minutes      |7000-7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp | cassandra-node2  |
| 88352c86ac9a  | cassandra:4.0  |"docker-entrypoint.s…"|11 minutes ago|Up 11 minutes     |7000-7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp | cassandra-node1  |

    docker run -it --network cassandra-network --rm cassandra:4.0 cqlsh cassandra-node1
    Connected to Test Cluster at cassandra-node1:9042
    [cqlsh 6.0.0 | Cassandra 4.0.8 | CQL spec 3.4.5 | Native protocol v5]
    Use HELP for help.
    cqlsh> consistency;
    Current consistency level is ONE.
    cqlsh> exit

/opt/cassandra/bin/nodetool status

    Datacenter: datacenter1
    Status=Up/Down
     |/ State=Normal/Leaving/Joining/Moving
    --  Address     Load        Tokens  Owns (effective)  Host ID                               Rack 
    UN  172.18.0.4  128.06 KiB  16      76.0%             4a15119c-6a8d-430a-8192-5b8dba67e10e  rack1
    UN  172.18.0.3  69.07 KiB   16      59.3%             370641de-1c48-460f-9479-4c8e5d9691ba  rack1
    UN  172.18.0.2  74.12 KiB   16      64.7%             ef66a827-186b-497c-ab1e-11227a78d738  rack1


## Даем нагрузку

### Запись

cassandra-node1 : /opt/cassandra/tools/bin/cassandra-stress write n=1000000 -rate

Connected to cluster: Test Cluster, max pending requests per connection 128, max connections per host 8
Datacenter: datacenter1; Host: localhost/127.0.0.1:9042; Rack: rack1
Datacenter: datacenter1; Host: /172.18.0.4:9042; Rack: rack1
Datacenter: datacenter1; Host: /172.18.0.3:9042; Rack: rack1
Created keyspaces. Sleeping 1s for propagation.
Sleeping 2s...
Warming up WRITE with 50000 iterations...
Failed to connect over JMX; not collecting these stats
Thread count was not specified

### Running with 4 threadCount
### Running WRITE with 4 threads for 1000000 iteration

    Results:
    Op rate                   :    6,778 op/s  [WRITE: 6,778 op/s]
    Partition rate            :    6,778 pk/s  [WRITE: 6,778 pk/s]
    Row rate                  :    6,778 row/s [WRITE: 6,778 row/s]
    Latency mean              :    0.6 ms [WRITE: 0.6 ms]
    Latency median            :    0.5 ms [WRITE: 0.5 ms]
    Latency 95th percentile   :    0.8 ms [WRITE: 0.8 ms]
    Latency 99th percentile   :    1.2 ms [WRITE: 1.2 ms]
    Latency 99.9th percentile :    4.7 ms [WRITE: 4.7 ms]
    Latency max               :  177.1 ms [WRITE: 177.1 ms]
    Total partitions          :  1,000,000 [WRITE: 1,000,000]
    Total errors              :          0 [WRITE: 0]
    Total GC count            : 0
    Total GC memory           : 0.000 KiB
    Total GC time             :    0.0 seconds
    Avg GC time               :    NaN ms
    StdDev GC time            :    0.0 ms
    Total operation time      : 00:02:27

### Running with 8 threadCount
### Running WRITE with 8 threads for 1000000 iteration
    Results:
    Op rate                   :   11,775 op/s  [WRITE: 11,775 op/s]
    Partition rate            :   11,775 pk/s  [WRITE: 11,775 pk/s]
    Row rate                  :   11,775 row/s [WRITE: 11,775 row/s]
    Latency mean              :    0.7 ms [WRITE: 0.7 ms]
    Latency median            :    0.6 ms [WRITE: 0.6 ms]
    Latency 95th percentile   :    1.0 ms [WRITE: 1.0 ms]
    Latency 99th percentile   :    1.8 ms [WRITE: 1.8 ms]
    Latency 99.9th percentile :    9.9 ms [WRITE: 9.9 ms]
    Latency max               :  317.7 ms [WRITE: 317.7 ms]
    Total partitions          :  1,000,000 [WRITE: 1,000,000]
    Total errors              :          0 [WRITE: 0]
    Total GC count            : 0
    Total GC memory           : 0.000 KiB
    Total GC time             :    0.0 seconds
    Avg GC time               :    NaN ms
    StdDev GC time            :    0.0 ms
    Total operation time      : 00:01:24

 - Improvement over 4 threadCount: 74%

### Running with 16 threadCount
### Running WRITE with 16 threads for 1000000 iteration
    Results:
    Op rate                   :   20,079 op/s  [WRITE: 20,079 op/s]
    Partition rate            :   20,079 pk/s  [WRITE: 20,079 pk/s]
    Row rate                  :   20,079 row/s [WRITE: 20,079 row/s]
    Latency mean              :    0.8 ms [WRITE: 0.8 ms]
    Latency median            :    0.6 ms [WRITE: 0.6 ms]
    Latency 95th percentile   :    1.4 ms [WRITE: 1.4 ms]
    Latency 99th percentile   :    2.5 ms [WRITE: 2.5 ms]
    Latency 99.9th percentile :   14.2 ms [WRITE: 14.2 ms]
    Latency max               :  197.0 ms [WRITE: 197.0 ms]
    Total partitions          :  1,000,000 [WRITE: 1,000,000]
    Total errors              :          0 [WRITE: 0]
    Total GC count            : 0
    Total GC memory           : 0.000 KiB
    Total GC time             :    0.0 seconds
    Avg GC time               :    NaN ms
    StdDev GC time            :    0.0 ms
    Total operation time      : 00:00:49

- Improvement over 8 threadCount: 71%

### Чтение

cassandra-node1 : /opt/cassandra/tools/bin/cassandra-stress read n=1000000 -rate

### Running with 4 threadCount
### Running READ with 4 threads for 1000000 iteration

    Results:
    Op rate                   :    5,827 op/s  [READ: 5,827 op/s]
    Partition rate            :    5,827 pk/s  [READ: 5,827 pk/s]
    Row rate                  :    5,827 row/s [READ: 5,827 row/s]
    Latency mean              :    0.7 ms [READ: 0.7 ms]
    Latency median            :    0.6 ms [READ: 0.6 ms]
    Latency 95th percentile   :    0.9 ms [READ: 0.9 ms]
    Latency 99th percentile   :    1.4 ms [READ: 1.4 ms]
    Latency 99.9th percentile :    6.2 ms [READ: 6.2 ms]
    Latency max               :  179.2 ms [READ: 179.2 ms]
    Total partitions          :  1,000,000 [READ: 1,000,000]
    Total errors              :          0 [READ: 0]
    Total GC count            : 0
    Total GC memory           : 0.000 KiB
    Total GC time             :    0.0 seconds
    Avg GC time               :    NaN ms
    StdDev GC time            :    0.0 ms
    Total operation time      : 00:02:51

### Running with 8 threadCount
### Running READ with 8 threads for 1000000 iteration

    Results:
    Op rate                   :   10,361 op/s  [READ: 10,361 op/s]
    Partition rate            :   10,361 pk/s  [READ: 10,361 pk/s]
    Row rate                  :   10,361 row/s [READ: 10,361 row/s]
    Latency mean              :    0.8 ms [READ: 0.8 ms]
    Latency median            :    0.7 ms [READ: 0.7 ms]
    Latency 95th percentile   :    1.2 ms [READ: 1.2 ms]
    Latency 99th percentile   :    2.0 ms [READ: 2.0 ms]
    Latency 99.9th percentile :   10.7 ms [READ: 10.7 ms]
    Latency max               :   46.1 ms [READ: 46.1 ms]
    Total partitions          :  1,000,000 [READ: 1,000,000]
    Total errors              :          0 [READ: 0]
    Total GC count            : 0
    Total GC memory           : 0.000 KiB
    Total GC time             :    0.0 seconds
    Avg GC time               :    NaN ms
    StdDev GC time            :    0.0 ms
    Total operation time      : 00:01:36

- Improvement over 4 threadCount: 78%   

### Running with 16 threadCount
### Running READ with 16 threads for 1000000 iteration

    Results:
    Op rate                   :   16,447 op/s  [READ: 16,447 op/s]
    Partition rate            :   16,447 pk/s  [READ: 16,447 pk/s]
    Row rate                  :   16,447 row/s [READ: 16,447 row/s]
    Latency mean              :    0.9 ms [READ: 0.9 ms]
    Latency median            :    0.8 ms [READ: 0.8 ms]
    Latency 95th percentile   :    1.8 ms [READ: 1.8 ms]
    Latency 99th percentile   :    3.4 ms [READ: 3.4 ms]
    Latency 99.9th percentile :   16.2 ms [READ: 16.2 ms]
    Latency max               :   49.2 ms [READ: 49.2 ms]
    Total partitions          :  1,000,000 [READ: 1,000,000]
    Total errors              :          0 [READ: 0]
    Total GC count            : 0
    Total GC memory           : 0.000 KiB
    Total GC time             :    0.0 seconds
    Avg GC time               :    NaN ms
    StdDev GC time            :    0.0 ms
    Total operation time      : 00:01:00

- Improvement over 8 threadCount: 59%

