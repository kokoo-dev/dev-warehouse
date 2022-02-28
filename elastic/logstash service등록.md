<a href="./logstash설치.md">logstash설치.md</a> 에서 설치한 logstash를 서비스로 등록하고, <br>
작성했던 test.conf를 실행할 수 있는 service 등록 방법을 설명합니다. <br>

1. 서비스생성
~~~sh
sudo vi /etc/systemd/system/logstash.service
~~~

> logstash.service 내용
~~~service
[Unit]
Description=logstash

[Service]
Type=simple
ExecStart=/usr/share/logstash/bin/logstash -f /home/ec2-user/test.conf #(1)

[Install]
WantedBy=multi-user.target
~~~
(1)은 .conf파일의 경로를 적어주면 됩니다.
<br>

2. 실행
~~~sh
sudo systemctl daemon-reload
sudo systemctl start logstash
~~~

간단하게 끝입니다.
