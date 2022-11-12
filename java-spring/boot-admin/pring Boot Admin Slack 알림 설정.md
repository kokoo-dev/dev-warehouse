Slack 에서 Webhook 설정을 한 이후에 대한 애플리케이션 설정입니다. <br>

> ex) application.yml

~~~yml
spring:
  boot:
    admin:
      notify:
        slack:
          enabled: true
          username: # (1)
          webhook-url: # (2)
          channel: # (3)
          message: "*#{instance.registration.name}* (#{instance.id}) is *#{event.statusInfo.status}*
          \n\n `Status Details`
          \n #{T(com.example.bootadmin.MessageParser).parse(event.statusInfo.details)}
          \n `Registration`
          \n *Service* Url: #{instance.registration.serviceUrl}
          \n *Health* Url: #{instance.registration.healthUrl}
          \n *Management* Url: #{instance.registration.managementUrl}
          " # (4)
~~~

(1): Slack 에 알림을 보낼 사용자 이름 <br>
(2): Slack 에서 생성한 webhook-url <br>
(3): Slack 에서 사용중인 채널명<br>
(4): 보낼 메세지<br>

Mail 의 경우 status-changed.html 을 템플릿으로 사용하여 그대로 메세지를 보낼 수 있었는데, Slack 의 경우 메세지를 직접 작성해주어야 합니다. <br>
SpEL 표기법을 지원하며 상세 메세지의 경우 Map 타입의 계층 적인 구조를 가지기에 위 설정 중 spring.boot.admin.notify.slack.message 의 3번째 줄 처럼 직접 파싱처리 해줍니다. <br><br>

> ex) MessageParser.java

~~~java
public class MessageParser {

    public static String parse(Map<String, Object> detail) {
        StringBuffer message = new StringBuffer();
        int indent = 0;

        addDetailMessage(message, detail, indent);

        return message.toString();
    }

    private static void addDetailMessage(StringBuffer message, Map<String, Object> detail,
        int indent) {

        for (String key : detail.keySet()) {
            addKeyTab(message, indent);

            message.append("*");
            message.append(key);
            message.append("*");
            message.append(": ");
            message.append("\n");

            Object value = detail.get(key);
            if (value instanceof Map) {
                addDetailMessage(message, (Map<String, Object>) value, indent + 1);
            } else {
                addTab(message, indent + 1);
                message.append(value.toString());
            }
            message.append("\n");
        }
    }

    private static void addKeyTab(StringBuffer message, int indent) {
        if(indent < 1){
            return;
        }

        addTab(message, indent);
    }

    private static void addTab(StringBuffer message, int indent) {
        for(int i=0; i<indent; i++) {
            message.append("\t");
        }
    }
}
~~~
