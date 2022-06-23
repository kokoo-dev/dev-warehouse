스프링에서 빈의 이름을 지정하지 않을 시에는 클래스명을 기준으로 이름이 지정되기에, <br/>
다른 패키지에 있더라도 클래스명이 같으면 컴파일 시 Bean name 중복 오류를 맞닥뜨리게 됩니다. <br/>

이 경우 Bean name에 패키지명을 포함시켜 중복 오류를 피하는 법에 대한 내용입니다. <br/>

테스트 환경: Spring Boot 2.5.2 <br/>

> ex) CustomBeanNameGenerator.java
~~~java
public class CustomBeanNameGenerator implements BeanNameGenerator {

    private final AnnotationBeanNameGenerator defaultGenerator = new AnnotationBeanNameGenerator();

    @Override
    public String generateBeanName(final BeanDefinition definition, final BeanDefinitionRegistry registry) {
        final String result;

        if (isSpringAnnotation(definition)) {
            result = generateFullBeanName((AnnotatedBeanDefinition) definition);
        } else {
            result = this.defaultGenerator.generateBeanName(definition, registry);
        }

        return result;
    }

    private String generateFullBeanName(final AnnotatedBeanDefinition definition) {
        return definition.getMetadata().getClassName();
    }

    private Set<String> getAnnotationTypes(final BeanDefinition definition) {
        final AnnotatedBeanDefinition annotatedDef = (AnnotatedBeanDefinition) definition;
        final AnnotationMetadata metadata = annotatedDef.getMetadata();
        return metadata.getAnnotationTypes();
    }

    private boolean isSpringAnnotation(final BeanDefinition definition) {
        if (definition instanceof AnnotatedBeanDefinition) {

            final Set<String> annotationTypes = getAnnotationTypes(definition);

            for (final String annotationType : annotationTypes) {
                if (annotationType.equals(Controller.class.getName())) {
                    return true;
                }
                if (annotationType.equals(RestController.class.getName())) {
                    return true;
                }
                if (annotationType.equals(Service.class.getName())) {
                    return true;
                }
                if (annotationType.equals(Repository.class.getName())) {
                    return true;
                }
            }
        }
        return false;
    }
}
~~~
<br/>

> ex) ExampleApplication.java
~~~java
@SpringBootApplication
public class ExampleApplication extends SpringBootServletInitialize{
  public static void main(String[] args){
    final SpringApplicationBuilder builder = new SpringApplicationBuilder(ExampleAPplication.class);
    builder.beanNameGenerator(new CustomBeanNameGenerator());
    builder.run(args);
  }
}
~~~


본 예제에서는 Controller, Service, Repository에 대해서 적용하였습니다. 
