AWS EC2 (Linux)에서 logstash 7.x버전을 설치하는 방법입니다. <br>
자바는 설치되어있다는 가정하에 yum으로 설치하여 실행까지 설명합니다. <br>

1. repo 생성
~~~sh
sudo vi /etc/yum.repos.d/logstash.repo
~~~
> logstash.repo 내용
~~~repo
[logstash-7.x]
name=Elastic repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
~~~

<br>

2. 설치
~~~sh
sudo yum install -y logstash #(1)
sudo yum install -y logstash-7.11.0 #(2)
~~~
(1)은 최신 버전, (2)은 특정 버전을 지정하는 방식입니다.

<br>

3. 테스트 conf 생성
~~~sh
sudo vi /home/ec2-user/test.conf
~~~
> test.conf 내용
~~~conf
input { 
  stdin {} 
} 
output {
  stdout {} 
}
~~~

<br>

4. 실행
~~~sh
sudo /usr/share/logstash/bin/logstash -f /home/ec2-user/test.conf
~~~

별다른 에러문구가 안나온다면 성공입니다.
