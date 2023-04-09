# Homework10

Необходимо:

- Разворачиваем кластер Etcd любым способом. Проверяем отказоустойчивость

- Разворачиваем кластер Consul любым способом. Проверяем отказоустойчивость

## Разворачиваем кластер Etcd любым способом. Проверяем отказоустойчивость

Характеристики ВМ: 1 CPU 2 Gb RAM CentOS8 развернуты в Virtual Box

Создаем 3 одинаковых ВМ

Загружаем двоичный архив etcd.

    curl -s https://api.github.com/repos/etcd-io/etcd/releases/latest \
      | grep browser_download_url \
      | grep linux-amd64 \
      | cut -d '"' -f 4 \
      | wget -i -

Разархивируем файл в каталог /usr/local/bin.

Проверяем версии etcd и etcdctl

[root@etcd1 ~]# etcd --version

    etcd Version: 3.5.7
    Git SHA: 215b53cf3
    Go Version: go1.17.13
    Go OS/Arch: linux/amd64

[root@etcd1 ~]# etcdctl version

    etcdctl version: 3.5.7
    API version: 3.5

Создаем etcd.service

    cat <<EOF | sudo tee /etc/systemd/system/etcd.service
    [Unit]
    Description=etcd service
    Documentation=https://github.com/etcd-io/etcd

    [Service]
    Type=notify
    User=etcd
    ExecStart=/usr/local/bin/etcd \\
      --name ${ETCD_NAME} \\
      --data-dir=/var/lib/etcd \\
      --initial-advertise-peer-urls http://${ETCD_HOST_IP}:2380 \\
      --listen-peer-urls http://${ETCD_HOST_IP}:2380 \\
      --listen-client-urls http://${ETCD_HOST_IP}:2379,http://127.0.0.1:2379 \\
      --advertise-client-urls http://${ETCD_HOST_IP}:2379 \\
      --initial-cluster-token etcd-cluster-0 \\
      --initial-cluster etcd1=http://etcd1:2380,etcd2=http://etcd2:2380,etcd3=http://etcd3:2380 \\
      --initial-cluster-state new \

    [Install]
    WantedBy=multi-user.target
    EOF

[root@etcd1 ~]# etcdctl member list

    4d90a4b48954f955, started, etcd3, http://192.168.18.202:2380, http://192.168.18.202:2379, false
    618713c84a835f7b, started, etcd1, http://192.168.18.200:2380, http://192.168.18.200:2379, false
    e445dc02c1646a61, started, etcd2, http://192.168.18.201:2380, http://192.168.18.201:2379, false


[root@etcd1 ~]# etcdctl  endpoint health

127.0.0.1:2379 is healthy: successfully committed proposal: took = 6.00189ms


Проверяем работу кластера:

    etcdctl put foo bar
    OK

    [root@etcd1 ~]# etcdctl get foo
    foo
    bar

    [root@etcd2 ~]# etcdctl get foo
    foo
    bar

    [root@etcd3 ~]# etcdctl get foo
    foo
    bar

    [root@etcd2 ~]# etcdctl endpoint --cluster health
    http://192.168.18.200:2379 is healthy: successfully committed proposal: took = 12.022489ms
    http://192.168.18.202:2379 is healthy: successfully committed proposal: took = 44.719063ms
    http://192.168.18.201:2379 is healthy: successfully committed proposal: took = 47.053714ms


### Выясняем кто лидер


    [root@etcd2 ~]# etcdctl  endpoint --cluster status
    http://192.168.18.202:2379, 4d90a4b48954f955, 3.5.7, 20 kB, false, false, 9, 28, 28,
    http://192.168.18.200:2379, 618713c84a835f7b, 3.5.7, 20 kB, true, false, 9, 28, 28,
    http://192.168.18.201:2379, e445dc02c1646a61, 3.5.7, 20 kB, false, false, 9, 28, 28,

    Лидер http://192.168.18.200:2379

  ### Проверяем отказоустойчивость. 
  Останавливаем лидера

  [root@etcd1 ~]# systemctl stop etcd
  
[root@etcd2 ~]# etcdctl get foo

foo
bar

[root@etcd3 ~]# etcdctl get foo

foo
bar

    [root@etcd2 ~]# etcdctl  endpoint --cluster status
    {"level":"warn","ts":"2023-04-09T04:14:51.528+0300","logger":"etcd-client","caller":"v3@v3.5.7/retry_interceptor.go:62","msg":"retrying of unary invoker failed","target":"etcd-endpoints://0xc0002a4fc0/127.0.0.1:2379","attempt":0,"error":"rpc error: code = DeadlineExceeded desc = latest balancer error: last connection error: connection error: desc = \"transport: Error while dialing dial tcp 192.168.18.200:2379: connect: connection refused\""}
    Failed to get the status of endpoint http://192.168.18.200:2379 (context deadline exceeded)
    http://192.168.18.202:2379, 4d90a4b48954f955, 3.5.7, 20 kB, false, false, 10, 29, 29,
    http://192.168.18.201:2379, e445dc02c1646a61, 3.5.7, 20 kB, true, false, 10, 29, 29,


 Останавливаем еще одного лидера

 [root@etcd3 ~]# etcdctl get foo

    {"level":"warn","ts":"2023-04-09T04:15:50.497+0300","logger":"etcd-client","caller":"v3@v3.5.7/retry_interceptor.go:62","msg":"retrying of unary invoker failed","target":"etcd-endpoints://0xc0002d6a80/127.0.0.1:2379","attempt":0,"error":"rpc error: code = DeadlineExceeded desc = context deadline exceeded"}
    Error: context deadline exceeded

Запускаем etcd2

[root@etcd2 ~]# systemctl start etcd

[root@etcd2 ~]# etcdctl get foo

    foo
    bar

[root@etcd3 ~]# etcdctl get foo

    foo
    bar

Запускаем etcd1, проверяем статус кластера

[root@etcd2 ~]# etcdctl  endpoint --cluster status

    http://192.168.18.202:2379, 4d90a4b48954f955, 3.5.7, 20 kB, true, false, 12, 33, 33,
    http://192.168.18.200:2379, 618713c84a835f7b, 3.5.7, 20 kB, false, false, 12, 33, 33,
    http://192.168.18.201:2379, e445dc02c1646a61, 3.5.7, 20 kB, false, false, 12, 33, 33,

    Мастер теперь на 3 ноде etcd3












