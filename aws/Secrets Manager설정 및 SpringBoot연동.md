DB의 접속정보나 API의 access key, secret key 등 민감정보를 설정파일에 기입한채 git에 올리는건 정보가 그대로 노출되기에 위험합니다. <br>
Secrets Manager는 코드상에서 직접 관리하는것이 아닌 원격지에서 관리함으로써 보안상 안전하고, 여러 서비스에 설정값을 일괄적으로 변경할 수 있는 편리합니다. <br>
물론 다른 방법이 있지만 사내에서 대부분의 서비스를 AWS로 이용하기에 Secrets Manager를 사용하게 되었고 이에 대한 설정방법을 공유하고자 합니다.<br>


1. AWS Secrets Manager 생성


2. awscli 설치 및 설정 (Mac)


3. Spring Boot 설정 및 테스트
