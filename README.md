# spring-rabbitMQ

## spring rabbitMQ 공식문서 가이드를 통해 rabbitMQ에 대해 알아보자

원문출처 : [https://spring.io/guides/gs/messaging-rabbitmq](https://spring.io/guides/gs/messaging-rabbitmq/#scratch)

### What you will build

RabbitTemplate로 메시지를 보내고 MessageListenerAdapter로 메시지를 수신하는 앱을 만든다.

### What you need

jdk 1.8+,  gradle 4+ or maven 3.2+, RabbitMQ Broker 설치.

- rabbit mq broker 설치

    ```bash
    brew install rabbitmq

    rabbitmq-server
    ```

- docker로도 rabbit-mq를 설치/실행 가능하다. → docker 커뮤니티에서 이미지 받으면 된다.

### Spring 프로젝트 생성

![spring-rabbitMQ%206365d749d6134f63aea8f9161a7aa651/Screen_Shot_2021-01-21_at_5.02.49_PM.png](https://user-images.githubusercontent.com/37807838/105333684-58e0d700-5c19-11eb-927c-67dfac398f59.png)

### Create a RabbitMQ Message Receiver

- 메시징 기반의 다른 앱들처럼 publish된 메시지들에 반응하는 receiver를 생성한다.
    - Reciever.java 생성
        - 데모버전 확인을 위해 CountDownLatch를 만들고 message 수신시에 신호를 준다.
        - 운영버전에서는 사용하지 말것.

### Register the Listener and Send a Message

- Spring AMQP의 RabbitTemplate은 RabbitMQ의 메시지를 수신/발신하는데 필요한 모든것을 제공해준다. 해당 기능을 위해 아래기능들을 설정해준다.
    - message listener container
    - queue와 exchange를 명시해주고, 그들을 결합해준다.
    - component를 설정하고  메시지를 보내 listener를 테스트해준다.
- Spring boot는 자동적으로 connection factory와 RabbitTemplate를 생성하여 작성해야할 코드를 줄여줍니다.
- @SpringBootApplication을 통해 아래 설정들을 편리하게 추가해줍니다.
    - @Configuration :  Application context에서 bean으로 동작하기위한 클래스를 태그한다.
    - @EnableAutoConfigruation : Spring boot에게 클래스 경로 설정, 기타 bean, 다양한 property설정에 따라 bean 추가를 하도록 지시한다.
    - 예를들어, spring-webmvc가 classpath에 있다면, 이 annotation은 이 application을 웹 응용프로그램으로 플래그를 지정하며, DispatcherServlet 설정과 같은 주요 동작을 활성화합니다.
    - @ComponentScan: Spring이 컨트롤러를 찾도록 다른 component, configuration, service들을 root package 경로(src/main/com/henry/rabbitmq)에서 찾게 합니다.
- main()메소드는 스프링부트 어플리케이션을 작동시킵니다.
- container()에서  listenerAdapter()은 message listener의 역할로 등록되었습니다.
    - spring boot에서 message들을 수신합니다.
    - Reciver.java는 POJO로 작성 되었기 때문에 MessageListenerAdapter에서 wrap되었습니다.
        - MessageListener는 recieveMessage를 부릅니다.

JMS queue와 AMQP queue는 다른 의미입니다. 예를 들어, JMS는 queue에 있는 메시지들을 오직 하나의 consumer에게 보냅니다. AMQP queue는 같은 일을 합니다만, AMQP producer는 queue에 직접 메시지를 보내지않습니다. 대신에, 메시지는 exchange에 보내지고, 이것은 단일 queue에 가거나,  여러 queue에 보내집니다. 마치 JMS Topic처럼.

- message listener와 receiver bean은 메시지를 수신하기위해 필요한 전부입니다.
- 메시지를 보내기위해 RabbitTemplate가 필요합니다.
    - queue()메소드는 AMQP queue를 생성합니다.
    - exchange()메소드는 topic을 생성합니다.
    - binding()메소드는 RabbitTemplate가 exchange에 발송할때 일어나는 동작을 정의해서 queue와 topic을 결합합니다.
- 이번 예제에서는 우리는 topic exchange를 사용하며, queue는 foo.bar.#이라는 라우팅 키와 결합되었습니다. 이것으로 foo.bar로 시작하는 라우팅키와 함께 보내지는 어떤 메시지들은 모두 queue로 라우팅됩니다.

### Send a Test Message

- 이번 예제에서 테스트 메시지는 CommandLineRunner에 의해 보내집니다. 이것은 receiver에서 latch를 기다리며 applicationContext를 종료합니다.
- Runner.java에서 해당하는 내용을 보여줍니다.
- 코드를 보면, rabbitTemplate은 메시지를 exchange로 보내는데, 바인딩된 값과 일치하는 routingKey "foo.bar.baz"를 사용하고 있습니다.

### Run the Applcation

main()메소드를 실행하면 아래와 같은 결과를 보여줍니다.

![spring-rabbitMQ%206365d749d6134f63aea8f9161a7aa651/Screen_Shot_2021-01-21_at_6.46.17_PM.png](https://user-images.githubusercontent.com/37807838/105333787-7746d280-5c19-11eb-8e60-24ee3b1d3f91.png)

### Build an executable JAR

```bash
./gradlew build
```