자바에서는 객체가 가지고 있는 값들을 문자열로 표현하기 위해 toString 이라는 메소드를 사용합니다. <br>
해당 메소드는 IDE 에서 자동 생성해주는 기능이나 lombok 의 @ToString 어노테이션 사용할 수도 있습니다. <br>
그러나 Json 형식과 같이 다른 형식을 사용하고 싶은 경우 직접 작성해줘야 하는데, <br>
늘어나고 줄어드는 필드를 직접 수작업을 해주거나 기능을 따로 만들어야 하는 번거로움이 생깁니다. <br>
이에 간단하게 대응 가능한 apache commons 의 라이브러리를 소개합니다. <br>

<h4> 1. 라이브러리 추가 </h4>

> build.gradle

~~~gradle
dependencies {
    implementation 'org.apache.commons:commons-lang3:3.12.0'
}
~~~

<br><br>

<h4> 2. toString() 적용 </h4>

> ex) Team.java, Member.java

~~~java
@Getter
@Builder
public class Team {
    private long teamNo;
    private String name;
    private boolean active;
    private Date regDt;
    private List<Member> members;

    @Override
    public String toString() {
        return ToStringBuilder.reflectionToString(this, ToStringStyle.JSON_STYLE); // (1)
    }
}
~~~

~~~java
@Getter
@Builder
public class Member {
    private long memberNo;
    private String name;
    private Date regDt;

    @Override
    public String toString() {
        return ToStringBuilder.reflectionToString(this, ToStringStyle.JSON_STYLE); // (2)
    }
}
~~~

(1), (2) 와 같이 ToStringStyle 만 적용해주면 됩니다. <br>
JSON_STYLE, MULTI_LINE_STYLE, NO_FIELD_NAMES_STYLE 등 여러가지가 있으므로 맞는 걸로 사용하면 됩니다. <br>
<br>

위 예시로 출력할 경우 아래와 같은 결과를 얻을 수 있습니다.

~~~json
{
    "active": true,
    "members": [
        {
            "memberNo": 1,
            "name": "member1",
            "regDt": "Tue Jul 26 10:59:22 KST 2022"
        },
        {
            "memberNo": 2,
            "name": "member2",
            "regDt": "Tue Jul 26 10:59:22 KST 2022"
        }
    ],
    "name": "teamName",
    "regDt": "Tue Jul 26 10:59:22 KST 2022",
    "teamNo": 1
}
~~~
