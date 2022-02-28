AWS EC2 (Linux)에서 ElasticSearch 7.x버전을 설치하는 방법입니다.


ElasticSearch는 자바8이상의 버전이 필요하며, 자바설치부터 외부에서의 접근까지 설명합니다.

1. 자바8 설치 및 버전확인
~~~sh
sudo yum install -y java-1.8.0-openjdk-devel.x86_64
java -version
~~~

<br>

2. repo생성
~~~sh
vi /etc/yum.repos.d/elasticsearch.repo
~~~

> elasticsearch.repo 내용
~~~repo
[elasticsearch-7.x]
name=Elasticsearch repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
~~~
yum을 이용해 7버전대를 다운로드 받기 위한 repo내용을 작성해줍니다.

<br>

3. 설치
~~~sh
sudo yum install -y elasticsearch #(1)
sudo yum install -y elasticsearch-7.11.0 #(2)
~~~
(1)은 최신 버전, (2)은 특정 버전을 지정하는 방식입니다.

<br>

4. 실행
~~~sh
sudo systemctl start elasticsearch
~~~

<br>

5. 확인
~~~sh
curl -X GET 'localhost:9200'
~~~
curl을 통해 엘라스틱서치의 http기본 포트인 9200을 확인합니다.

<br>

6. 외부 접근 설정
~~~sh
sudo vi /etc/elasticsearch/elasticsearch.yml
~~~

> elasticsearch.yml 내용
~~~yml
network.host: 0.0.0.0
cluster.initial_master_nodes: []
~~~
설정파일에 다른 내용들도 있지만 외부에서의 접속을 확인하기 위해서는 위 설정은 꼭 해줍니다.

<br>

7. 재시작
~~~sh
sudo systemctl restart elasticsearch
~~~
이 후 외부에서 http://서버IP:9200 로 접속하여 정상적으로 구동되었는지 확인하면 됩니다.



