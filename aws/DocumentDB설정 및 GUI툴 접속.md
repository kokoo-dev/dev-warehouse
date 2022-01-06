AWS의 DocumentDB생성 및 설정 후 MongoDB GUI툴인 Stuio 3T에서 연결하는 예제입니다. <br>
local에서 ec2로 ssh접속 후 ec2에서 mongo-cli를 이용한 mongoDB접속을 하게됩니다. <br>

보안그룹은 ec2에서 documentDB에 27017 포트로 붙을 수 있게 설정해줍니다.<br>


1. ec2서버에서 CA인증서를 다운로드 해줍니다.<br>
~~~sh
wget https://s3.amazonaws.com/rds-downloads/rds-combined-ca-bundle.pem
~~~

2. ec2 서버에서 몽고 쉘 설치 후 몽고db 접속 확인 <br>
~~~sh
sudo apt-get install -y mongodb-org-shell

mongo --ssl --host {{클러스터 엔드포인트}}:27017 --sslCAFile rds-combined-ca-bundle.pem --username {마스터계정} --password {마스터계정 암호}
~~~

에러없이 접속됐으면 이제 studio 3t를 설치해줍니다. <br>
구글에 검색해서 설치를 해도 되고 맥의 경우 brew install studio-3t 로 다운로드할 수 있습니다.<br>


이 후 studio 3t에서 설정입니다.<br><br>
<h4>3.1. server</h4>
<img src="/img/20220106_1.png" width="800px">
- 클러스터 엔드포인트와 포트 기입

<h4>3.2. Authentication</h4>
<img src="/img/20220106_2.png" width="800px">
- 마스텨계정, 암호를 기입해주고, 따로 생성한 Database가 없을 경우 AuthenticationDB는 admin으로 지정해줍니다.

<h4>3.3. SSL</h4>
<img src="/img/20220106_3.png" width="800px">
- 위 1.에서 받은 방식대로 로컬에서도 인증서를 다운로드 해주고 CA파일로 등록해줍니다.

<h4>3.4. SSH</h4>
<img src="/img/20220106_4.png" width="800px">
- ec2서버의 호스트, 계정 이름을 적어주고 Private Key를 등록합니다.

접속방법은 이렇게 끝입니다. <br>
robo 3t는 몽고db 엔진 버전의 문제인지 접속이 안되고, studio 3t로 접속하였습니다.
