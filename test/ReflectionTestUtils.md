ReflectionTestUtils란 단위 테스트나 통합 테스트 시 사용가능한 Reflection 기반의 유틸입니다. <br>
테스트 코드 작성 간 private 필드의 값을 주입하거나 private 메소드를 실행해야할 때가 있습니다. <br>
예를 들어 getter의 기능만 가지고 있는 클래스의 private 필드 값 변경이나 private 메소드 실행이 필요한 경우에 <br>
접근 제한자의 변경, setter 추가, 다른 클래스로 대체하는 등의 제품 코드 변경없이 테스트코드를 작성할 수 있게 도와줍니다. <br>


<h3> 테스트 스프링 부트 버전 </h3>

- 2.5.x

<br>

> ex) Component.java

~~~java
public class Component {
    private String field;

    public String getField() {
        return field;
    }

    private void call(){
        System.out.println("call!");
    }
}
~~~
private 필드, getter, private 메소드만 가진 클래스

<br>

> ReflectionTestUtils.setField()

~~~java
@DisplayName("field set 테스트")
@Test
void testSetFieldSuccess() {
    Component component = new Component();
    String field = "success";

    ReflectionTestUtils.setField(component, "field", field);

    assertEquals(component.getField(), field);
}
~~~

<br>

> ReflectionTestUtils.invoke()

~~~java
@DisplayName("invoke 테스트")
@Test
void testInvokeSuccess() {
    Component component = new Component();
    String expectResult = "success";

    String result = ReflectionTestUtils.invokeMethod(component, "call");

    assertEquals(expectResult, result);
}
~~~
