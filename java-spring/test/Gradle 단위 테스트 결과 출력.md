Gradle 빌드 후에 테스트 결과를 출력하기 위한 설정

<br><br>

> ex) build.gradle

~~~gradle

tasks.named('test') {
    useJUnitPlatform()
    testLogging {
        afterSuite { descriptor, result ->
            if (descriptor.parent == null) {
                println "Test Result: ${result.resultType} " +
                        "${result.testCount} test, " +
                        "${result.successfulTestCount} successful, " +
                        "${result.failedTestCount} failed, " +
                        "${result.skippedTestCount} skipped" +
                        
            }
        }
    }
}
~~~

- afterSuite: test 종료 후 실행
  - 테스트 케이스 별 종료 후 실행을 위해서는 afterTest 사용
- descriptor: 테스트에 대한 정보
- result: 테스트 결과에 대한 정보