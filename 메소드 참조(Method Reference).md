## 메소드 참조 (Method Reference) 사용 예제
자바 8부터 제공되는 방식으로 람다 표현식에서 파라미터, 괄호를 생략하여 보다 더 간략하게 표현하며 <br/>
:: (이중 콜론)을 이용해 아래와 같이 사용합니다.

- Class::instanceMethod
- Class::staticMethod
- Object::instanceMethod
<br/><br/><br/>

### 예제에 사용할 클래스, 인터페이스 생성
> ex) ExampleMethod.java
~~~java
public class ExampleMethod {
    public static void printClass(Object object){
        Class cls = object.getClass();
        System.out.println(cls.getName());
    }

    public void printClassInstance(Object object){
        Class cls = object.getClass();
        System.out.println(cls.getName());
    }
}
~~~
<br/>

> ex) Person.java
~~~java
public class Person {
    private String name;
    private int age;
    
    //생성자, Getter, Setter 생략
    
    public void print(){
        System.out.println("name :: " + name + " age :: " + age);
    }
}
~~~
<br/>

> ex) Printer.java
~~~java
public interface Printer {
    public void printArgStr(Object object);
}
~~~
<br/><br/>

### 1. 스태틱 메소드 (Static Method)
~~~java
public class MethodReference {
    public static void executeStaticMethod(){
        String str = "";
        Printer printer = ExampleMethod::printClass;
        printer.printArgStr(str);
        
        // 출력 : java.lang.String
    }
}
~~~
ExampleMethod 클래스의 printClass 스태틱 메소드를 사용한 경우

<br/><br/>
### 2. 인스턴스 메소드 (Instance Method)
~~~java
public class MethodReference {
    public static void executeInstanceMethod(){
        //1번
        ExampleMethod exampleMethod = new ExampleMethod();
        Printer printer = exampleMethod::printClassInstance;
        printer.printArgStr(new String(""));
        
        // 출력 : java.lang.String

        //2번
        List<Person> personList = Arrays.asList(new Person("김린이", 10), new Person("김성인", 30), new Person("김노인", 80));
        personList.stream().forEach(Person::print);
        
        // 출력
        // name :: 김린이 age :: 10
        // name :: 김성인 age :: 30
        // name :: 김노인 age :: 80
    }
}
~~~
1번의 경우 ExampleMethod 클래스를 인스턴스화 하여 printClassInstance 메소드를 사용한 경우, <br/>
2번의 경우 forEach 구문에서 person 객체가 올 것을 알고 있기에 Class:Method 형태로 사용한 경우다.

<br/><br/>

### 3. 생성자 (Constructor)
~~~java
public class MethodReference {
    //1번
    Function<String, Person> function1 = Person::new;
    Person person1 = function1.apply("박지성");
    person1.print();
    
    // 출력 : name :: 박지성 age :: 0

    //2번
    BiFunction<String, Integer, Person> function2 = Person::new;
    Person person2 = function2.apply("히딩크", 74);
    person2.print();
    
    // 출력 : name :: 히딩크 age :: 74
}
~~~
클래스::new 형태로 메소드 위치에 new 연산자가 위치한 형태 <br/>
1, 2번 차이는 파라미터 개수에 따라 다른 함수형 인터페이스를 사용한 정도입니다.

<br/><br/><br/>
예제 소스: https://bit.ly/3ynBU7H
