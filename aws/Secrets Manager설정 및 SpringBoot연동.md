DB의 접속정보나 API의 access key, secret key 등 민감정보를 설정파일에 기입한채 git에 올리는건 정보가 그대로 노출되기에 위험합니다. <br>
Secrets Manager는 코드상에서 직접 관리하는것이 아닌 원격지에서 관리함으로써 보안상 안전하고, 여러 서비스에 설정값을 일괄적으로 변경할 수 있는 편리합니다. <br>
물론 다른 방법이 있지만 사내에서 대부분의 서비스를 AWS로 이용하기에 Secrets Manager를 사용하게 되었고 이에 대한 설정방법을 공유하고자 합니다.<br>


1. AWS Secrets Manager 생성
<br><br>
 1.1. 암호 생성 및 요금표
 <img src="/img/20220118_1.png" width="800px">
 - Secrets Managers 검색 후 접속하여 (1)새 보안 암호 저장 클릭하여 생성화면으로 넘어갑니다. <br>
    (2)를 보니 생각보다 요금이 저렴한편으로 보입니다.
<br><br>
 1.2. 보안암호유형 선택
 <img src="/img/20220118_2.png" width="800px">
 - 어플리케이션에서 사용할것이기에 (1)다른 유형의 보안암호 선택 후 (2)사용할 키/값 입력. (키/값은 이후에 추가가 가능합니다.)
<br><br>
 1.3. 보안암호이름 입력
 <img src="/img/20220118_3.png" width="800px">
 - prefix는 기본적으로 /secret이며 그 뒤로 사용할 이름을 작성해줍니다.
<br><br>
 1.4. 로컬에서 사용하기 위한 IAM생성
  <img src="/img/20220118_4.png" width="800px">
 - (1) 이름 설정 및 (2) 프로그래밍 방식 선택
<br><br>
 1.5. IAM 정책설정<br>
  <img src="/img/20220118_5.png" width="800px">
 - (1)을 선택 후 SecretsMangerReadWrite 정책 추가. 이 후 나오는 accesskey, secretkey는 저장해둡니다.
 <br><br>
2. awscli 설치 및 설정 (Mac)

~~~sh
brew install awscli //설치
aws --version //설치 및 버전확인
aws configure //configure 설정
AWS Access Key ID [None] : {access key}
AWS Secret Access Key [None] : {secret key}
Default region name [None] : {리전 ex)ap-northeast-2}
Default output format [None] : json
~~~
 - IAM 키값들과 사용하는 리전, 포맷 정보를 입력해줍니다.


3. Spring Boot 설정 및 테스트
