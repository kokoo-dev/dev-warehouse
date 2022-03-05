Elastic Search의 클러스터 구성의 대한 에제로 클러스터 상태 정보를 관리하는 Master 노드 1대, <br>
데이터를 저장하고 처리하는 Data 노드 1대로 설정해보고자 합니다. <br><br>

클러스터 구성 시 대표적으로 주의해야할 점은 Master 노드 개수를 홀수로 해줘야 한다는 것입니다. <br>
이는 Split Brain 문제의 이유로, Master노드들의 네트워크 단절과 같은 상황 발생 시 노드가 개별로 작동하여 데이터 정합성 등의 문제를 야기할 수 있는 상태에 놓이기 때문입니다. <br><br>

그럼 Elastic Search 7.X버전이 각기 다른 서버에 설치되어 있다는 가정하에 이 후 설정부분부터 설명합니다. <br>

> master 노드의 elasticsearch.yml

~~~yml
cluster.name: es-test #(1)

node.name: ${HOSTNAME} #(2)
node.master: true #(3)
node.data: false  #(4)

path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch

network.host: 0.0.0.0 #(5)

http.port: 9200
transport.tcp.port: 9300
transport.tcp.compress: true

discovery.seed_hosts: ["{Master서버IP}"] #(6)

cluster.initial_master_nodes: ["{Master서버IP}"] #(7)
~~~

<br>

> data 노드의 elasticsearch.yml

~~~yml
cluster.name: es-test #(1)

node.name: ${HOSTNAME} #(2)
node.master: false #(3)
node.data: true #(4)

path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch

network.host: {Data서버IP} #(5)

http.port: 9200
transport.tcp.port: 9300
transport.tcp.compress: true

discovery.seed_hosts: ["{Master서버IP}", "{Data서버IP}"] #(6)

cluster.initial_master_nodes: ["{Master서버IP}"] #(7)
~~~

(1) cluster.name: 같은 클러스터로 구성되는 노드들의 이름은 같게 설정해줍니다. <br>
(2) node.name: 노드 별로 이름을 다르게 설정하며 host명을 사용하였습니다. <br>
(3), (4): 각 노드의 역할에 맞게 true, false로 지정해줍니다 <br>
(5) network.host: 테스트 용도로 master 노드에서는 전체 (0.0.0.0)으로 설정해주었고 data 노드에서는 마스터와 통신가능한 ip를 설정하였습니다. <br>
(6) discovery.seed_hosts: master 노드의 경우는 자신의 ip만을, Data 노드의 경우 모든 Data 노드의 ip를 적어줍니다. <br>
(7) cluster.initial_master_nodes: master 노드의 ip만 적어줍니다.

<br> <br>

Master node를 Data node랑 같이 사용하다가 Master only로 변경할 경우 충돌 에러가 발생할 수 있습니다.
이 경우 노드에서 원하지 않는 데이터를 삭제하도록 elasticsearch-node repurpose를 실행해줍니다. <br>
~~~sh
/usr/share/elasticsearch/bin/elasticsearch-node repurpose
~~~

또는 Master 노드와 Data 노드에 있는 데이터가 달라서 클러스터링되지 못할 수가 있습니다. <br>
이 경우에는 데이터가 저장된 디렉토리를 통째로 지워줍니다. <br>

~~~sh
sudo rm -rf /var/lib/elasticsearch/nodes
~~~

이 후 Master 노드에 데이터를 넣어서 Data 노드에 적재되는지 확인해봅니다.
