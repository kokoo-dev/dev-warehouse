<a href="https://github.com/kokoo-dev/dev-warehouse/blob/main/spring/cloud/spring%20cloud%20config%20%EA%B8%B0%EB%B3%B8%EC%84%A4%EC%A0%95.md">여기</a>에서 github public 저장소에 설정파일을 올려 놓은 상태로 접근하였고 <br>
이번에는 private 저장소로 접근할 수 있게 설정하는 방법입니다. <br><br>

위 링크의 설정을 이어서합니다. <br>

<h3>ssh 생성, 설정 및 config 서버 설정 <h3>

<h4>1. ssh키 생성 및 설정</h4>

~~~sh
ssh-keygen -m PEM -t rsa -b 4096 -C
~~~

설정 방법에 대해 구글에서 찾아봤을 때 위와 같은 방식으로 rsa 키를 생성하라고 하였으나 <br>
You're using an RSA key with SHA-1, which is no longer allowed. Please use a newer client or a different key type. 라는 에러 문구를 보여주며 접근을 실패하게 됩니다. <br><br>
더 이상 해당키가 사용되지 않는다는 얘기로 보이며, https://github.blog/2021-09-01-improving-git-protocol-security-github/ << 이 url 로 확인해보라며 안내해줍니다. <br>

결과적으로 RSA 가 아닌 ECDSA 로 키를 생성하는 것입니다.<br>

~~~sh
ssh-keygen -t ecdsa -b 256 -m PEM
~~~

해당 명령어를 입력하면 id_ecdsa (개인키), id_cedsa.pub (공개키) 가 생성됩니다. <br><br>

<h4>2. 공개키 등록</h4>
<img src="/img/20220416_1.png" width="500px">

github에 로그인 후 (1), (2), (3) 순서대로 클릭하여 공개키 내용을 복사해 그대로 등록해줍니다. <br><br>

<h4>3. config 서버 개인키 설정</h4>

> ex) public 저장소 설정의 application.yml
~~~yml
spring:
  cloud:
    config:
      server:
        git:
          default-label: master
          uri: https://github.com/{저장소 경로}
~~~

<br>

> ex) private 저장소 설정의 application.yml
~~~yml
spring:
  cloud:
    config:
      server:
        git:
          default-label: master
          uri: git@github.com:{저장소 경로} #(1)
          ignore-local-ssh-settings: true
          privateKey: | #(2)
            -----BEGIN EC PRIVATE KEY-----
            ... 개인키 내용 ...
            -----END EC PRIVATE KEY-----
~~~

(1): http가 아닌 ssh 주소를 입력 <br>
(2): | 로 시작하며 줄바꿈 후 개인키를 입력하는데, privateKey 보다 한칸 들여쓰기
<br><br>

해당 설정으로 저장소의 설정 파일을 정상적으로 불러오는 것을 확인할 수 있습니다.