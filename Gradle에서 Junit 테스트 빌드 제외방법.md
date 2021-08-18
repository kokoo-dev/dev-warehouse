> ex) build.gradle
~~~gradle
plugins {
  ... 내용 ...
}

dependencies {
  ... 내용 ...
}

test {
  exclude '**/*'
}
~~~

test{} 안에 exlude '**/*'를 넣으면 모든 경로의 테스트 코드가 빌드에서 제외됩니다.
