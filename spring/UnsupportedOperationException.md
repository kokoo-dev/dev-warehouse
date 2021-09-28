스프링 배치에서 writer 부분을 작업하다 java.lang.UnsupportedOperationException 가 발생했습니다. <br/>
ItemWriter를 상속받아 write메소드를 재정의하여 items 인자를 컨트롤하려던 찰나에 발생하였습니다.<br/><br/>

해당 Exception의 경우 Arrays.asList()를 통해 생성된 list에 대해 add, remove등 list구조를 변경하려는 메소드를 사용할 경우 발생한다고 합니다. <br/>
Arrays.asList()로 생성된 list는 고정된 크기의 불변 list이기에 추가 삭제가 안되는 것이었습니다.<br/><br/>

간단한 예시를 통해 회피 방법을 남기겠습니다.<br/>
> ex) Test.java
~~~java
public class Test {

    public void run(){
        List<String> failList = Arrays.asList("test1", "test2");
        
        //(1) UnsupportedOperationException
        //list.add("test3");
        //list.remove(0);
        
        List<String> successList = new ArrayList<>(list); //(2)
        successList.add("test3");
        successList.remove(0);
    }
}
~~~

(1)은 불변 리스트를 변경하려 하기에 UnsupportedOperationException이 발생하며 (2)처럼 새로운 list를 만들어서 사용하면 됩니다.
