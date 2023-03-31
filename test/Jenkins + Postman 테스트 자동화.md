<h2>Jenkins + Postman 을 활용한 API 테스트 자동화 </h2>

<br><br>
Jenkins 설치 과정 및 Postman Json Export 과정은 생략합니다. <br>

<h3>Test Version</h3>

- OS: macOS Ventura
- Jenkins: 2.387.1
- Postman: 10.11.2
- Nodejs: 19.8.0
- Newman: 5.3.2

<br><br>

<h3>순서</h3>

- <a href="#node-install-setting">Node.js Plugin 설치 및 설정</a>
- <a href="#jenkins-item-setting">Jenkins item 설정</a>
- <a href="#test-result">테스트 결과 확인</a>

<br><br>

<h3 id="node-install-setting">Node.js Plugin 설치 및 설정</h3>

Postman 에서 추출한 Json 파일을 커맨드에서 실행시키기 위해 필요한 Newman 을 설치해야하며, Node.js 가 필요합니다. <br>

**Node.js Plugin 설치** <br>

<img src="/img/test/nodejs_install_20230330.png" width="600px"> <br>

- Jenkins 관리 - Plugin Manager 이동
- nodejs 를 검색하여 다운로드

<br>

**Newman 설치를 위한 설정** <br>

<img src="/img/test/nodejs_setting_20230330.png" width="600px"> <br>

- Jenkins 관리 - Global Tool Configuration 이동
- NodeJS installations 클릭
- Name 입력, Version 선택
- newman install 을 위해 Global npm packages to install 에 newman 입력 후 저장

<br><br>

<h3 id="jenkins-item-setting">Jenkins item 설정</h3>
테스트 스크립트를 실행하기 위한 새 item 을 생성후 설정해줍니다. <br>

**Node.js 지정** <br>

<img src="/img/test/item_setting_build_env_20230330.png" width="600px"> <br>

- Global Tool Configuration 에서 설정한 Node.js 버전 지정

<br>

**Newman 실행 명령** <br>

<img src="/img/test/item_setting_build_steps_20230330.png" width="600px"> <br>

- Add build step - Execute shell 선택
- newman 실행 명령어 입력

~~~sh
newman run /Users/kojinha/Desktop/test-automation.json --reporters cli,json,junit --reporter-junit-export "newman/myreport.xml" --reporter-json-export "newman/myreport.json"
~~~

- newman run: Postman에서 Export한 Json 파일을 실행시키는 명령어
- --reporters: 결과 제공방식 지정
  - cli: 기본적으로 활성화지만 다른 reporter와 사용할 때는 명시해주어야 함
  - json: json 형태 제공
  - junit: xml 형태 제공
- --reporter-junit-export: 현재 jenkins workspace 기준 상대 경로에 생성
  - 생성 경로 예) /Users/kojinha/.jenkins/workspace/test-automation/newman/myreport.xml
- --reporter-json-export: 현재 jenkins workspace 기준 상대 경로에 생성
  - 생성 경로 예) /Users/kojinha/.jenkins/workspace/test-automation/newman/myreport.json

<br>

**테스트 결과 출력** <br>

XML 파일을 파싱하여 테스트 결과를 보여주도록 설정 <br>

<img src="/img/test/item_setting_after_build_20230330.png" width="600px"> <br>

- Test report XMLs: 현재 jenkins workspace 기준 상대 경로의 파일 지정
- Do not fail the build on empty test results: 테스트 결과가 없음에도 빌드에 영향을 주지 않음.

<br><br>

<h3 id="test-result">테스트 결과 확인</h3>

**테스트 결과 Console** <br>

<img src="/img/test/console_output_20230330.png" width="600px"> <br>

**전체 테스트 결과 보고서** <br>

Dashboard - item에서 전체 테스트 결과 확인 <br>

<img src="/img/test/all_test_result_20230330.png" width="600px"> <br>

**빌드 개별 테스트 결과 보고서** <br>

Dashboard - item - 빌드 결과 - Test Results에서 개별 테스트 결과 확인 <br>

<img src="/img/test/build_detail_test_result_20230330.png" width="600px"> <br>