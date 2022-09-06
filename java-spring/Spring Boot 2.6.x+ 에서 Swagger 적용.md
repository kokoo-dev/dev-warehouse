스프링 2.6.x 이상에서 swagger 적용 후 컴파일 시 오류가 발생합니다. <br>

~~~log
org.springframework.context.ApplicationContextException: Failed to start bean 'documentationPluginsBootstrapper'; nested exception is java.lang.NullPointerException
~~~
<br>

Spring boot 2.6 부터 path match 방식을 Ant-based path matcher 대신 PathPattern-based matcher 로 변경하여 위와 같은 예외 발생합니다. <br>
이는 아래와 같은 설정으로 쉽게 대응이 가능합니다. <br>

~~~yml
mvc:
  view:
  pathmatch:
    matching-strategy: ant_path_matcher
~~~

<br>

하지만 이와 같은 설정에도 actuator 를 사용할 경우 동일한 예외가 발생합니다. <br>

~~~gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
}
~~~

이 때는 빈을 등록해주는 설정 하나로 처리할 수 있습니다. <br>

~~~java
@Configuration
public class SwaggerConfig {
    @Bean
    public WebMvcEndpointHandlerMapping webEndpointServletHandlerMapping(WebEndpointsSupplier webEndpointsSupplier,
                                                                         ServletEndpointsSupplier servletEndpointsSupplier,
                                                                         ControllerEndpointsSupplier controllerEndpointsSupplier,
                                                                         EndpointMediaTypes endpointMediaTypes,
                                                                         CorsEndpointProperties corsProperties,
                                                                         WebEndpointProperties webEndpointProperties,
                                                                         Environment environment) {
        Collection<ExposableWebEndpoint> webEndpoints = webEndpointsSupplier.getEndpoints();

        List<ExposableEndpoint<?>> allEndpoints = new ArrayList();
        allEndpoints.addAll(webEndpoints);
        allEndpoints.addAll(servletEndpointsSupplier.getEndpoints());
        allEndpoints.addAll(controllerEndpointsSupplier.getEndpoints());

        String basePath = webEndpointProperties.getBasePath();
        EndpointMapping endpointMapping = new EndpointMapping(basePath);
        boolean shouldRegisterLinksMapping = shouldRegisterLinksMapping(webEndpointProperties, environment, basePath);

        return new WebMvcEndpointHandlerMapping(endpointMapping,
                webEndpoints,
                endpointMediaTypes,
                corsProperties.toCorsConfiguration(),
                new EndpointLinksResolver(allEndpoints, basePath),
                shouldRegisterLinksMapping, null);
    }

    private boolean shouldRegisterLinksMapping(WebEndpointProperties webEndpointProperties, Environment environment, String basePath) {
        return webEndpointProperties.getDiscovery().isEnabled()
                && (StringUtils.hasText(basePath) || ManagementPortType.get(environment).equals(ManagementPortType.DIFFERENT));
    }
}
~~~
