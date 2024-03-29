> ex) build.gradle

~~~gradle
implementation 'org.springframework.boot:spring-boot-starter-mail'
~~~

> ex) application.yml

~~~yml
spring:
  mail:
    host: # (1)
    username: # (2)
    password: # (3)
    port: 587
    test-connection: true
    protocol: smtp
    properties:
        mail:
          debug: true
          smtp:
            timeout: 3000
            writetimeout: 3000
            auth: true
            starttls:
              enable: true
  boot:
    admin:
      notify:
        mail:
          enabled: true
            to:
              - # (4)
            template: classpath:/status-changed.html
            from: #(5)
~~~
(1), (2), (3): SMTP 서버 정보 입력 <br>
(4): 받는 사람 메일 정보 (List) <br>
(5): 보내는 사람 메일 형태 주소 <br>

> ex) status-changed.html

~~~html
<!DOCTYPE html>
<!--
  ~ Copyright 2014-2018 the original author or authors.
  ~
  ~ Licensed under the Apache License, Version 2.0 (the "License");
  ~ you may not use this file except in compliance with the License.
  ~ You may obtain a copy of the License at
  ~
  ~     http://www.apache.org/licenses/LICENSE-2.0
  ~
  ~ Unless required by applicable law or agreed to in writing, software
  ~ distributed under the License is distributed on an "AS IS" BASIS,
  ~ WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  ~ See the License for the specific language governing permissions and
  ~ limitations under the License.
  -->

<html xmlns:th="http://www.thymeleaf.org">
<head>
  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
  <style>
    h1, h2, h3, h4, h5, h6 {
      font-weight: 400
    }

    ul {
      list-style: none
    }

    html {
      box-sizing: border-box
    }

    *, :after, :before {
      box-sizing: inherit
    }

    table {
      border-collapse: collapse;
      border-spacing: 0
    }

    td, th {
      text-align: left
    }

    body, button {
      font-family: BlinkMacSystemFont, -apple-system, "Segoe UI", Roboto, Oxygen, Ubuntu, Cantarell, "Fira Sans", "Droid Sans", "Helvetica Neue", Helvetica, Arial, sans-serif
    }

    code, pre {
      -moz-osx-font-smoothing: auto;
      -webkit-font-smoothing: auto;
      font-family: monospace
    }
  </style>
</head>
<body>
<th:block th:remove="all">
  <!-- This block will not appear in the body and is used for the subject -->
  <th:block th:remove="tag" th:fragment="subject">
    [[${instance.registration.name}]] ([[(${instance.id})]]) is [[${event.statusInfo.status}]]
  </th:block>
</th:block>
<h1><span th:text="${instance.registration.name}"/> (<span th:text="${instance.id}"/>)
  is <span th:text="${event.statusInfo.status}"/>
</h1>
<p>
  Instance <a th:if="${baseUrl}" th:href="@{${baseUrl + '/#/instances/' + instance.id + '/'}}"><span
    th:text="${instance.id}"/></a>
  <span th:unless="${baseUrl}" th:text="${instance.id}"/>
  changed status from <span th:text="${lastStatus}"/> to <span th:text="${event.statusInfo.status}"/>
</p>

<h2>Status Details</h2>
<dl th:fragment="statusDetails" th:with="details = ${details ?: event.statusInfo.details}">
  <th:block th:each="detail : ${details}">
    <dt th:text="${detail.key}"/>
    <dd th:unless="${detail.value instanceof T(java.util.Map)}" th:text="${detail.value}"/>
    <dd th:if="${detail.value instanceof T(java.util.Map)}">
      <dl th:replace="${#execInfo.templateName} :: statusDetails (details = ${detail.value})"/>
    </dd>
  </th:block>
</dl>

<h2>Registration</h2>
<table>
  <tr th:if="${instance.registration.serviceUrl}">
    <td>Service Url</td>
    <td>
      <a th:href="${instance.registration.serviceUrl}" th:text="${instance.registration.serviceUrl}"></a>
    </td>
  </tr>
  <tr>
    <td>Health Url</td>
    <td>
      <a th:href="${instance.registration.healthUrl}" th:text="${instance.registration.healthUrl}"></a>
    </td>
  </tr>
  <tr th:if="${instance.registration.managementUrl}">
    <td>Management Url</td>
    <td>
      <a th:href="${instance.registration.managementUrl}" th:text="${instance.registration.managementUrl}"></a>
    </td>
  </tr>
</table>
</body>
</html>
~~~

해당 html 파일을 resources 하위에 위치시킵니다.