Swagger 는 /swagger-ui/index.html 로 접속하면 api 문서 화면이 나오는데, <br>
Gson 과 함께 사용할 경우 오류 화면을 보여주게 됩니다. <br>

정상적인 경우 /v3/api-docs << 해당 path 로 확인해 보면 <br>

~~~json
{"openapi" : "3.0.3", "info":{"title"....
~~~

위 처럼 정상적인 JSON 형태의 데이터가 보여야 하는데 <br>

~~~json
{\"openapi\" : \"3.0.3\", \"info\":{\"title\"....
~~~

위와 같이 역슬래시(\) 가 더블 큰 따옴표(") 앞에 붙어서 나오는걸 확인할 수 있습니다. <br>
Spring 에서 기본적으로 사용하는 HttpMessageConverter 객체 처리는 Jackson 을 사용하지만 <br>
spring.http.converters.preferred-json-mapper=gson 설정을 통해 gson 으로 강제하면서 파싱에 문제가 생긴 것 같습니다.<br><br>

Stack Overflow 에서 찾은 답변을 참조하여 쉽게 처리가 가능합니다. <br>

> ex) GsonConfig.java
 
~~~java
@Configuration
public class GsonConfig {
    @Bean
    public Gson gson(){
        return new GsonBuilder()
                .registerTypeAdapyer(Json.class, new SwaggerJsonTypeAdapter())
                .create();
    }
    
    public class SwaggerJsonTypeAdapter implements JsonSerializer<Json> {
        @Override
        public JsonElement serialize(Json json, Type type, JsonSerializationContext context) {
            return JsonParser.parseString(json.value());
        }
    }
}
~~~