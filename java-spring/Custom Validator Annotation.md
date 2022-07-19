기본으로 제공되는 Valid 어노테이션 외에 Valid 를 Custom 하여 사용할 수 있습니다. <br>
<br>

<h4> 1. annotation </h4>

> ex) CustomValid.java

~~~java
@Documented
@Constraint(validatedBy = CustomValidator.class) // (1)
@Target({ElementType.FIELD}) // (2)
@Retention(RetentionPolicy.RUNTIME)
public @interface CustomValid {
    String message() default "";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
    String pattern() default ""; // (3)
}
~~~

(1): 검증을 구현한 클래스를 지정 <br>
(2): 검증은 필드에 사용할 것이기에 ElementType.FIELD 로 지정<br>
(3): 검증에 사용하기위해 어노테이션 선언 시 입력받을 값<br>

<br>

<h4> 2. 검증 구현 클래스 </h4>

> ex) CustomValidator.java

~~~java
public class CustomValidator implements ConstraintValidator<CustomValid, Object> { // (1)
    private String pattern; 

    @Override
    public void initialize(CustomValid customValid) { // (2)
        this.pattern = customValid.pattern();
    }

    @Override
    public boolean isValid(Object object, ConstraintValidatorContext context) {
        // (3)
        context.disableDefaultConstraintViolation(); // (4)
        context.buildConstraintViolationWithTemplate(message) // (5)
                .addConstraintViolation();
    }
}
~~~

(1): ConstraintValidator 를 구현하기 위해 implements 해주며 검증 어노테이션과 필드 유형 지정<br>
(2): 어노테이션 선언 시 입력받은 값을 초기화해주는 설정<br>
(3): 검증을 하기 위한 로직<br>
(4): 기본 메세지 비활성화<br>
(5): 새로운 메세지 추가<br>

<br>

<h4> 3. 사용 예시 </h4>

> ex) User.java

~~~java
@Getter
@Setter
public class User {
    private String name;
    
    @CustomValid(message = "", pattern = "") // (1)
    private String phone;
}
~~~

필드에 어노테이션을 추가해줍니다. <br>
이 후 @Valid 나 javax.validation.Validator의 validate() 메소드를 통해 검증을 하면 되겠습니다.



