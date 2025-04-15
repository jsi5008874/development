**반응형 및 비동기적인 웹 애플리케이션 개발을 지원하는 모듈입니다. 이 모듈은 Reactive Streams 사양을 기반으로 하여, 비동기적인 이벤트 지향 프로그래밍을 통해 높은 확장성과 성능을 제공**

  

반응형 프로그래밍을 통해 '높은 처리량'과 '확장성'을 갖는 애플리케이션을 만드는 것을 목표

  

### 반응형 프로그래밍

- **Spring Webflux는 반응형 프로그래밍(Reactive Programming) 방식을 통해 ‘이벤트 기반의 비 동기식 애플리케이션’을 구축할 수 있습니다.**
- 이 이벤트는 ‘비 동기적’으로 처리되며 새로운 이벤트가 발생하면 이벤트 스트림이 생성이 되며 스트림을 구독하면 이벤트를 처리할 수 있습니다.

  

**프로그래밍 패러다임으로 본 명령형 프로그래밍 vs 반응형 프로그래밍**

- 명령형 프로그래밍의 경우는 컴퓨터가 수행해야 하는 일을 명령의 목록으로 나열을 하고 이를 순서대로 실행합니다. 반응형 프로그래밍의 경우는 데이터 스트림을 처리하며 이 데이터 스트림이 변경될 때마다 반응하는 것을 의미합니다.

  

![[image 300.png]]

  

### Reactor

- **반응형 프로그래밍(Reactive Programming)을 구현하기 위한 Reactor는 Reactive 라이브러리 중 하나입니다.**
- Publisher-Subscriber 패턴을 중심으로 동작하며 데이터를 생성하고 가공하고 구독자에게 전달하는 역할을 합니다.
- Reactor에서는 Mono와 Flux의 데이터 스트림 유형을 지원합니다.

  

데이터 스트림 : **Stream 데이터**는 **연속적이고 끝이 없는 데이터 흐름**을 의미하며, 데이터가 **일정한 간격**으로 생성되어 전달되는 형태  
  
  

**반응형 스트림(Reactive Stream)**

- **비 동기적 및 이벤트 기반 응용 프로그램을 위한 ‘스트림 처리 기술’을 의미합니다. 해당 기술의 핵심은 ‘Publisher’가 ‘Subscriber’에게 데이터를 제공하는 것을 의미합니다.**
- 이는 Publisher는 데이터를 생성하고 Subscriber는 이를 처리합니다. 이러한 방식으로 스트림을 처리함으로써 데이터를 더 효율적으로 처리할 수 있습니다.

  

![[image 301.png]]

  

반응형 스트림 수행과정

![[image 302.png]]

1.**[subscribe]**

Subscriber를 Publisher에 ‘등록’하고 데이터 스트림을 ‘수신할 준비’가 되었음을 Publisher에게 알립니다.

2.**[onSubscribe]**

Publisher가 Subscriber에게 데이터 스트림을 ‘전송하기 시작하기 전에 호출’됩니다. 이 메서드를 통해 Subscriber는 Subscription 객체를 받아들여 데이터의 양을 제어할 수 있습니다.

3.**[request(n)/cancel]**

request(n) 메서드는 Publisher에게 n개의 데이터를 요청하고, cancel 메서드는 데이터 스트림을 취소합니다.

4.**[onNext(data)]**

Publisher가 ‘생성한 데이터’를 Subscriber에게 전달합니다. 이 메서드는 데이터가 전송될 때마다 호출됩니다.

5.**[onComplete/onError]**

Publisher가 모든 데이터를 전송하고, 더 이상 데이터가 없을 때 호출됩니다.

- onComplete 메서드는 모든 데이터가 성공적으로 전송되었음을 나타냅니다.
- onError 메서드는 데이터 전송 중 오류가 발생했음을 나타냅니다.

  

**이벤트(Event)**

- 구독자(Subscriber)가 발행자(Publisher)에게 데이터를 요청하고 발행자가 데이터를 생성하고 전송하는 것을 의미

  

**백프레셔(BackPressure)**

- 비동기 스트림 처리에서 ‘데이터의 양을 제어’하는 것을 의미합니다. 발행자(Publisher)가 생성한 데이터를 구독자(Subscriber)가 처리할 때 구독자가 처리할 수 있는 데이터 양을 초기화하여 데이터가 생성되는 상황을 방지하기 위해 백 프레셔를 사용합니다.
- 백프레셔는 Subscriber가 처리할 데이터의 양을 Subscription을 통해 제어하며, Publisher는 Subscriber의 처리 속도에 맞춰 데이터를 생성합니다. 이를 통해 Subscriber가 처리할 수 있는 양 이상의 데이터가 생성되는 상황을 방지하고, 시스템의 안정성을 보장할 수 있습니다.

  

### Mono

- **Reactor 라이브러리에서 제공하는 Reactive Streams의 Publisher 중 하나로 오직 ‘0개 또는 하나의 데이터항목 생성’하고 이 결과가 생성되고 나면 스트림이 종료되면 결과 생성을 종료합니다.**
- Mono를 사용하여 비동기적으로 결과를 반환하면 해당 결과를 구독하는 클라이언트는 결과가 생성될 때까지 블로킹하지 않고 다른 작업을 수행할 수 있습니다.

![[image 303.png]]

  

> [ Mono 사용 예시 ]

- "Hello, world!"라는 문자열을 Mono.just를 통해 Mono로 만든 후, map 연산자를 이용해 문자열을 대문자로 변환합니다.
- 이후, flatMap 연산자를 이용해 "Mono: "이라는 문자열과 결합합니다.
- 마지막으로, 비동기적으로 처리된 결과값을 subscribe 메서드를 이용하여 출력합니다.

```Less
Mono.just("Hello, world!")
    .map(String::toUpperCase)
    .flatMap(s -> Mono.just("Mono: " + s))
    .subscribe(System.out::println);
```

  

  

### Flux

- **Reactor 라이브러리에서 제공하는 Reactive Streams의 Publisher 중 하나로 Mono와 달리 ‘여러 개의 데이터 항목’를 생성하고 스트림이 종료되면 결과 생성을 종료합니다.**
- 비동기 작업을 수행하면 작업이 완료될 때까지 블로킹하지 않고 다른 작업을 수행할 수 있습니다.
- Spring WebFlux에서 Flux를 사용하여 HTTP 요청을 처리하는 경우, 요청을 수신한 즉시 해당 요청을 처리하고 결과를 생성하는 대신 결과 생성이 완료될 때까지 다른 요청을 처리할 수 있습니다.

  

![[image 304.png]]

![[image 305.png]]

  

> [ Flux 예시 ]

- "apple", "banana", "cherry" 세 문자열을 Flux.just를 통해 Flux로 만든 후, map 연산자를 이용해 문자열을 대문자로 변환합니다.
- 이후, flatMap 연산자를 이용해 "Flux: "이라는 문자열과 결합합니다.
- 마지막으로, 비동기적으로 처리된 결과값을 subscribe 메서드를 이용하여 출력합니다.

```TypeScript
Flux.just("apple", "banana", "cherry")
    .map(String::toUpperCase)
    .flatMap(s -> Mono.just("Flux: " + s))
    .subscribe(System.out::println);
```

  

  

### Netty

- **자바 기반의 네트워크 애플리케이션 프레임워크로 ‘비동기식 이벤트 기반 서버’를 만드는 데 사용이 됩니다.**
- 높은 성능과 확장성을 갖추고 있으며, 다양한 프로토콜을 지원하고 있어 네트워크 애플리케이션 개발에 매우 유용합니다.

![[image 306.png]]

  

### Multi Event Loop

**- Reactor Core에서 지원을 하며 단일 스레드로 동작하는 ‘이벤트 루프를 여러 개 사용’하는 것을 의미합니다.** 이를 통해 블로킹 I/O 작업이 발생하더라도 다른 이벤트 루프를 통해 애플리케이션이 멈추지 않고 계속 동작할 수 있습니다.  
  
  

![[image 307.png]]

**Multi Event Loop 처리 과정**

**1. [Event Queue]**

이벤트가 발생하면 해당 이벤트를 처리하기 위한 콜백 함수를 저장합니다.

**2. [Process Events]**

이벤트 루프(Event Loop)는 이벤트 큐에서 이벤트를 하나씩 가져와 처리합니다.

**3. [Event Loop]**

이벤트 루프는 주어진 작업을 처리합니다.

**4. [Register Callback]**

작업을 처리하기 위한 콜백 함수를 등록합니다.

**5. [Intensive Operation(Platform)]**

블로킹 I/O 작업이 발생하면, Event Loop는 해당 작업을 처리하기 위한 새로운 스레드를 생성합니다.

**6. [Operation Completion]**

블로킹 I/O 작업이 완료되면, 결과를 Event Loop에게 알리고 새로운 스레드는 종료됩니다.

**7. [Trigger Callback]**

Event Loop는 결과를 처리하고, 다른 작업을 처리합니다.

  

  

### WebClient

- **WebFlux의 일부인 Webclient는 비동기적인 방식으로 HTTP 요청을 보내고 응답을 받을 수 있는 라이브러리를 의미합니다.**
- 다수의 외부 API 호출이나, 다른 서비스들과의 통합 작업에서 유용합니다.
- WebFlux의 WebClient는 비동기적인 방식으로 HTTP 요청을 보내고 응답을 받을 수 있는 라이브러리입니다. 이를 통해 Reactive Streams를 이용하여 높은 성능의 네트워크 통신을 구현할 수 있습니다.

  

WebClient webClient = WebClient.builder().build();

Mono resultMono = webClient.get()  
.uri("  
[https://www.example.com/](https://www.example.com/)")  
.retrieve()  
.bodyToMono(String.class);  

String result = resultMono.block();

System.out.println(result);

  

  

  

[[WebFlux 구현]]