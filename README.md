# event-driven-starter

## EDM(Event Driven MicroService)
### 개요
> Event Driven MicroService(EDM)는 MSA가 적용된 시스템에서 이벤트 발생시 해당 이벤트 로그를 보관하고 이를 기반으로 동작하며, 
> 비동기 통신을 통해 시스템 내 통합(integration)을 수행하는 Architecture 입니다.  
> 
> **이벤트**  
> IT 영역에서 이벤트는 다양한 정의를 갖지만, 이 곳에서 언급하는 이벤트는 상태의 변경. 즉, 데이터의 변경,생성,삭제(CUD)를 통해 발생하는 서비스의 의미있는 변화를 뜻합니다.
> 
> **이벤트 로그를 보관**  
> 현재의 데이터는 상태 변경의 누적이라는 생각에서 시작합니다. 이 때 상태 변경은 이벤트를 뜻하고 이를 누적하는 행위는 이벤트 로그를 보관하는 것입니다. 
> EDM 에서 생성된 이벤트는 반드시 보관되어야 합니다. 보관된 이벤트는 데이터의 현재 상태를 구성하는 근간이 됩니다. 또한, 보관된 이벤트를 바탕으로 장애 발생 
> 또는 특정 요구사항에 따라 지정된 시점으로 복원을 수행합니다. 이벤트 로그를 보관하는 장소를 이벤트 스토어라 칭합니다.
> 
> **비동기 통신**  
> AMQP, MQTT, JMS 등 메세징 프로토콜을 통한 메세지 큐 방식이 자주 사용됩니다. 서비스에서 데이터의 생성,변경,삭제(CUD)를 통해 이벤트가 발생하면 
> 발행 서비스는 메세지의 형태로 이벤트를 발행하고, 해당 이벤트에 관심이 있는 서비스에서 구독을 수행합니다. 메세지 큐를 사용함으로 requeue/dlq 등의 기능을 활용할 수 있습니다.
> 
> **시스템 내 통합(integration)**  
> 이상적으로 구현된 MSA는 서비스 간 데이터 참조를 위한 내부 통신이 필요없지만, 현실적으로 서비스 간 내부 통신이 전혀 없는 시스템을 구현하기란 불가능에 가깝습니다. 
> 다양한 사유로 여러 서비스 간 통신을 통해 연동이 발생합니다.  

### 왜 Event Driven MicroService 를 적용하는가?
> MSA를 적용한 시스템은 서비스가 쪼개지고 Database가 쪼개집니다. MSA 에서 주요 원칙 중 하나는 서비스 별 자체 비즈니스 로직과 데이터, 그에 따른 최적의 Database 를 선택하는 것입니다.    
> Database Per Service 는 MSA의 느슨한 결합, 관심사의 집중, 폴리글랏 프로그래밍, 독립적인 배포 주기 등을 달성하기 위한 핵심 키워드입니다.  

### 분산 트랜잭션 처리
> 기존의 Lagacy 시스템에서 문제 발생시 일관된 commit 또는 rollback 처리나 이전에 발생한 상태 변경에 직접 접근해서 데이터 수정이 가능합니다. 
> 하지만 MSA가 적용된 시스템에서 서로 다른 서비스에 걸쳐진 기능을 수행하는 도중 일관된 commit 또는 rollback을 수행할 수 없습니다. 
> 이 때 EDM을 적용해 rollback 또는 retry를 처리할 수 있습니다. rollback이 필요한 경우 Failed 이벤트를 발생시키고, 
> 이를 이전 스텝을 수행한 서비스에서 구독하여 보관되어 있던 이벤트 로그 기반으로 rollback을 수행합니다. 
> retry가 필요한 경우 메세지 큐의 requeue 또는 dead letter queue 기능을 사용해 retry 처리를 수행할 수 있습니다.

### 참조사이트
> [Event Driven Architecture란?](https://medium.com/dtevangelist/event-driven-microservice-란-54b4eaf7cc4a)

---

## EDA(Event Driven Architecture)
### 개요
> 분산된 시스템에서 이벤트를 생성(발행)하고 발행된 이벤트를 수신자에게 전송하는 구조로 수신자는 그 이벤트를 처리하는 방식의 아키텍처입니다.  
> 분산 아키텍처 환경에서 상호 간 결합도를 낮추기 위해 비동기 방식으로 메시지를 전달하는 패턴으로 주로 Message Broker(Kafka, RabbitMQ)와 결합하여 구성됩니다.  

### EDA의 구성요소
> **Event Generator (Publisher, Producer, Creater)**  
> 표준화된 형식의 이벤트를 생성(발행)합니다. 생성된 이벤트는 Event Channel로 전송합니다.
> 
> **Event Channel (Bus)**  
> Event Generator에서 Event Processing Engine으로 수집된 데이터를 전파하는 메커니즘입니다.
> 즉, 이벤트를 필요로 하는 시스템까지 발송하는 역할입니다.
> 
> **Event Processing Engine (Consumer, Processor)**  
> 수신한 이벤트를 식별/처리하는 역할을 합니다. 처리 결과에 따라 새로운 이벤트를 생성할 수 있습니다.
> Consumer는 이벤트의 송신자에 대한 정보를 알 필요가 없습니다.

### EDA의 동작 방식
> **Message 생성 (Publish/Subscribe)**  
> 이벤트가 생성되면 Subscriber(수신자)에게 전달합니다.
> 이벤트는 반복되어 전달되지 않으며, 수신자는 송신자의 정보를 알 필요가 없습니다.
> 
**Event Source**  
> Event Processor에게 이벤트를 전달하는 역할을 합니다.
> Event Source는 1개 이상일 수 있으며, 1개 이상의 Event Processor에게 전달합니다.
> 
> **Event Processor**  
> 수신된 이벤트에 대한 여러 Action을 수행하는 역할입니다.
> 단일 이벤트에 대하여 타임스템프를 추가한다거나, 파생 이벤트를 만드는 등의 작업을 수행합니다.
> 
> **Event Consumer**  
> 이벤트에 대한 처리를 합니다. 실질적인 Biz Logic을 수행합니다.

### 토폴로지 구성
> 일반적으로 한가지 작업만 필요한 단순한 단일 이벤트일 경우에는 `Broker Topology` 를
> 이벤트 조율이 필요한 복잡한 이벤트 플로우일 경우 `Mediator Topology` 를 사용합니다.
> 
> **Mediator Topology (중재자 토폴로지)**  
> 여러 단계의 과정을 중재자를 통해 조율할 필요성이 있으면 일반적으로 사용합니다.
> 동시에 처리하지 않거나 실행 전에 처리해야 할 요소가 있으면 사용하는 토폴로지입니다.
> 큐를 이용하여 사전처리를 진행한 후 관련된 Consumer에게 전달합니다.
> 
> **Broker Topology (브로커 토폴로지)**  
> 큐나 중재자 없이 이벤트와 응답을 직접적으로 연관시키고자 할 때 사용합니다.
> 이벤트 플로우가 단순한 중앙집중식 토폴로지이며 Message Broker 와 Consumer 가 가장 중요한 요소입니다.
> 모든 작업은 비동기로 처리되며 이벤트에 대한 조율이 필요 없습니다.

### EDA의 장단점
> **장점**  
> 1. Loosely Coupling: 분산 시스템간 느슨한 결합도를 제공합니다.
> 2. 분산된 시스템간 의존성 배제: 약속된 Message를 통해 통신하기 때문에 다른 시스템의 정보를 알 필요가 없으므로 시스템 간 의존성이 배제됩니다.
> 3. 확장성, 탄력성 향상
> 
> **단점**  
> 1. Broker Dependency: 시스템 간 의존도는 낮아지지만 메시지브로커에 대한 의존성이 발생합니다. 만약 메시지 브로커의 장애가 발생하면 큰 장애로 확산될 가능성이 있습니다.
> 2. Transaction 단위 분리: 장애나 이슈발생시 Retry/Rollback에 대한 고려가 필요합니다.
> 3. 시스템 Flow파악이 어려움
> 4. 디버깅이 어려움

### EDA를 선택할 때 고려할 특징
> **Multicast 통신**  
> 한 개의 이벤트 Publisher는 다양한 Consumer에게 이벤트를 전달할 수 있습니다.
> 
> **실시간 전송**  
> 실시간으로 발생하는 이벤트에 대한 처리가 가능합니다.
> 
> **비동기 통신**  
> 비동기로 통신이 이루어지므로 Publisher는 처리 결과를 기다릴 필요가 없습니다.
> 
> **세밀한 통신**  
> 한 덩어리의 큰 이벤트(데이터)보다 작고 자세한 이벤트로 통신이 이루어집니다.
> 
> **이벤트 온톨로지**  
> 이벤트들은 확실하게 구분된 특징을 지니며 그에 상응하는 Consumer에게 전달됩니다.
> 
> **자유로운 배포**  
> 느슨할 결합도를 제공하므로 새로운 서비스의 배포가 자유롭습니다.

### 참조사이트
> [EDA(Event Driven Architecture)이란?](https://akasai.space/architecture/about_event_driven_architecture/)

---

## Spring Event
### 스프링 이벤트를 사용하는 이유와 장점
> spring event 를 사용하는 가장 주된 이유는 '서비스 간의 강한 의존성을 줄이기 위함'이라고 볼 수 있는데요.
> 예를 들어 어떤 상품을 주문하는 프로세스가 있고, 해당 프로세스는 내부적으로 주문을 처리한 뒤 푸시 메시지를 발송하고, 메일을 전송하는 과정을 거친다고 가정하겠습니다.
> '주문 처리'와 '푸시 메시지 발송', '메일 전송' 기능이 각각의 서비스(OrderService, PushService, MailService)에 구현되어 있을 경우, 
> 주문 처리를 하는 OrderService 에서 푸시 메시지 발송을 하는 PushService 와 메일 전송을 하는 MailService 에 대한 의존성을 주입받아 
> 사용하게 되는데요.  
> 도메인 사이의 강한 의존성으로 인해 시스템이 복잡해지는 경우가 발생할 수 있다고 하며, 스프링 이벤트를 통해 이러한 도메인 간의 의존성을 줄일 수 있게 됩니다.  

### 스프링 이벤트 구성 요소 및 동작 구현
> spring event 는 크게 'event class' 와 이벤트를 발생시키는 'event publisher' 그리고 이벤트를 받아들이는 'event listener' 3가지 요소로 볼 수 있는데요.  
> event class: OrderedEvent.java
> ```java
> public class OrderedEvent {
>     private String productName;
> 
>     public OrderedEvent(String productName) {
>         this.productName = productName;
>     }
> 
>     public String getProductName() {
>         return productName;
>     }
> }
> ```
> 
> event publisher: OrderService.java
> ```java
> @Service
> @RequiredArgsConstructor
> public class OrderService {
> 
>     private final ApplicationEventPublisher publisher;
> 
>     public void order(String productName) {
>         //주문 처리
>         log.info(String.format("주문 로직 처리 [상품명 : %s]", productName));
>         publisher.publishEvent(new OrderedEvent(productName));
>     }
> }
> ```
> 
> event listener: OrderedEventListener.java
> ```java
> @Component
> public class OrderedEventListener {
> 
>     @EventListener
>     public void sendPush(OrderedEvent event) throws InterruptedException {
>         log.info(String.format("푸시 메세지 발송 [상품명 : %s]", event.getProductName()));
>     }
> 
>     @EventListener
>     public void sendMail(OrderedEvent event) throws InterruptedException {
>         log.info(String.format("메일 전송 [상품명 : %s]", event.getProductName()));
>     }
> }
> ```

### 비동기 처리
> @Async 을 추가하면 해당 메서드는 기존 스레드와 분리되게 됩니다. 따라서 OrderService 의 order 메서드의 응답 대기는 사라지며 트랜잭션도 분리됩니다.  
> Main class
> ```java
> @EnableAsync
> @SpringBootApplication
> public class EventApplication {
> 
> 	public static void main(String[] args) {
> 		SpringApplication.run(EventApplication.class, args);
> 	}
> }
> ```
>
> event listener: OrderedEventListener.java
> ```java
> @Component
> public class OrderedEventListener {
>     @Async 
>     @EventListener
>     public void sendPush(OrderedEvent event) throws InterruptedException {
>         log.info(String.format("푸시 메세지 발송 [상품명 : %s]", event.getProductName()));
>     }
> 
>     @Async
>     @EventListener
>     public void sendMail(OrderedEvent event) throws InterruptedException {
>         log.info(String.format("메일 전송 [상품명 : %s]", event.getProductName()));
>     }
> }
> ```

### @TransactionalEventListener 를 사용해야 하는 경우
> SignUpMemberService.java
>  ```java
> ...
> private final MemberRepository memberRepository;
> private final ApplicationEventPublisher eventPublisher;
> 
> @Transactional
> public void signup(MemberDto memberDto) {
>     //1. 회원가입 회원 정보 저장
>     memberRepository.save(new Member(memberDto.getId(), memberDto.getName()));
>     //2. 회원가입 축하 메일 전송 이벤트 발생
>     eventPublisher.publishEvent(new SavedMemberEvent(memberDto));
> 
>     //3. 어떠한 사유로 인해 exception 발생
>     if (memberDto.getName().equals("master")) {
>         throw new RuntimeException("can not use this name.");
>     }
> }
> ```
> @EventListener 의 경우 publishEvent() 메서드가 호출되는 시점에서 바로 이벤트를 publishing 하는데요.
> 만약 다음과 같이 트랜잭션으로 묶인 signup() 메서드에서 '1. 회원가입 회원 정보 저장' 부분과 '2. 회원가입 축하 메일 전송 이벤트 발생' 부분이 정상적으로 동작한 뒤에 
> '3. 어떠한 사유로 인해 exception 발생' 부분에서 exception 이 발생된다면, '1. 회원 정보 저장' 부분은 트랜잭션에 의해 롤백이 실행되지만, 
> '2. 축하 메일 전송' 부분은 롤백이 되지 않는 상황이 발생하게 됩니다.  
> 이러한 경우가 발생하기 때문에 트랜잭션이 적용되는 로직에서 이벤트 처리가 필요할 때는 @TransactionalEventListener 가 사용되는 것인데요.  

### @TransactionalEventListener 옵션
> 사용법은 phase 옵션을 통해 트랜잭션 상태에 따른 이벤트 처리를 적용할 수 있는데요.
> 1. @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT): default 값이며, 트랜잭션이 commit 되었을 때 이벤트를 실행합니다.
> 2. @TransactionalEventListener(phase = TransactionPhase.ROLLBACK): 트랜잭션이 rollback 되었을 때 이벤트를 실행합니다.
> 3. @TransactionalEventListener(phase = TransactionPhase.AFTER_COMPLETION): 트랜잭션이 completion(commit 또는 rollback) 되었을 때 이벤트 실행합니다.
> 4. @TransactionalEventListener(phase = TransactionPhase.BEFORE_COMMIT): 트랜잭션이 commit 되기 전에 이벤트를 실행합니다.

### Propagation.REQUIRES_NEW
> ```java
> @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
> public void transactionalEventListenerAfterCommit(SavedMemberEvent event) {
>     eventLogRepository.save(new EventLog(1L, "event log1"));
> }
> ```
> 위 코드와 같이 이벤트 리스너에서 추가로 데이터베이스에 insert, update, delete 작업을 진행해야 하는 경우가 있을 수 있는데요.  
> 실제로 해당 코드가 동작하였을 때, 오류가 발생하지 않고 동작하였음에도 불구하고 eventLog 에 대한 데이터는 insert 되지 않는 상황이 생기게 됩니다.
> 이유는 @TransactionalEventListener 의 경우 event publisher(여기서는 SavedMemberEvent 를 발생시킨 Service) 의 트랜잭션 안에서 동작하며, 
> 커밋이 된 이후 추가 커밋을 허용하지 않기 때문인데요.  
> 때문에 insert, update, delete 같은 작업이 필요한 경우 아래 코드와 같이 이벤트 리스너에서 @Transactional(propagation = Propagation.REQUIRES_NEW)를 
> 추가 설정하는 과정이 필요합니다. (혹은 @Async 를 붙이면 별도의 스레드에서 동작하기 때문에 propagation 이 없이도 정상 commit 됨.)    
> ```java
> @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
> @Transactional(propagation = Propagation.REQUIRES_NEW)
> public void transactionalEventListenerAfterCommit(SavedMemberEvent event) {
>     eventLogRepository.save(new EventLog(1L, "event log1"));
> }
> ```
> REQUIRES_NEW 설정은 해당 메서드가 이전 트랜잭션을 이어받지 않고 새로운 트랜잭션을 시작하겠다는 설정인데요.  
> event publisher 의 commit 을 보장하고, 이벤트 리스너에서는 새로운 트랜잭션에서의 작업 수행을 가능하게 합니다.  

### 참조사이트
> [spring 이벤트 사용하기(event publisher, event listener)](https://wildeveloperetrain.tistory.com/217)  
> [Spring Event, @TransactionalEventListener 사용하기](https://wildeveloperetrain.tistory.com/246)  
> [Spring - Event Driven](https://velog.io/@backtony/Spring-Event-Driven)  
