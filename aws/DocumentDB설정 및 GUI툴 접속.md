AWS의 DocumentDB생성 및 설정 후 MongoDB GUI툴인 Stuio 3T에서 연결하는 예제입니다. <br>
local에서 ec2로 ssh접속 후 ec2에서 mongo-cli를 이용한 mongoDB접속을 하게됩니다. <br>

보안그룹은 ec2에서 documentDB에 27017 포트로 붙을 수 있게 설정해줍니다.<br>


1. ec2서버에서 인증서를 다운로드 해줍니다.<br>
~~~sh
wget https://s3.amazonaws.com/rds-downloads/rds-combined-ca-bundle.pem
~~~

2. ec2 서버에서 몽고 쉘 설치 후 몽고db 접속 확인 <br>
~~~sh
sudo apt-get install -y mongodb-org-shell

mongo --ssl --host {{클러스터 엔드포인트}}:27017 --sslCAFile rds-combined-ca-bundle.pem --username {마스터계정} --password {마스터계정 암호}
~~~
