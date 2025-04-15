  

### Controller VS Router

**Spring Webflux는 비동기적 서비스를 위한 Spring 프레임워크의 한 요소로 ‘Controller’와 ‘Router’라는 두 가지 주요 컴포넌트를 제공합니다.**

Controller 방식은 Spring MVC와 유사한 방식으로 HTTP 요청을 처리하는 데 사용됩니다. 이 컴포넌트는 주로 RESTful API 엔드포인트를 구현하는 데 사용됩니다.

 Router 방식은 HTTP 요청을 처리하는 데 사용되는 라우터 및 핸들러 함수를 정의하는 데 사용됩니다. 비동기적으로 실행되며 스레드가 블로킹되지 않아 더 높은 처리량을 달성할 수 있습니다.

**결론적으로는 Controller 방식은 동기식 요청 처리에 적합하며, Router는 비동기식 요청처리에 적합합니다.**

  

![[image 357.png]]

 **엔드포인트(Endpoint)**

- RESTful API에서 URI(Uniform Resource Identifier)로 식별되는 특정한 인터넷 자원에 대한 접근 경로를 말합니다.
- 클라이언트는 URI를 통해 서버에 요청을 보내고, 서버는 해당 요청에 대한 응답을 반환합니다.
- 예를 들어, https://example.com/api/users는 사용자 정보를 가져오기 위한 엔드포인트입니다.

  

 **Java DSL(Java Domain Specific Languague)**

- 특정 도메인에서 사용되는 언어를 의미하며 자바 코드를 사용하여 Router를 정의할 수 있습니다. Router는 요청을 처리하기 위해 적절한 핸들러 함수를 선택하는 메커니즘을 제공합니다.
- Java DSL을 사용하면 코드 가독성을 높일 수 있고, Router에 대한 정적 타입 검사를 수행할 수 있습니다.

  

### Router Function

- **Handler Function을 호출하는 역할을 합니다. 이를 통해 HTTP 요청을 받고 적절한 Handler Function으로 라우팅 할 수 있습니다.**
- Router Function은 RouterFunctions 클래스를 사용하여 생성할 수 있으며 다양한 메소드를 사용하여 HTTP 요청에 따라 적절한 Handler Function으로 라우팅 할 수 있습니다.
- Router Function을 사용하면 라우팅 규칙을 보다 직관적이고 유연하게 제어할 수 있습니다.

  

```java
@Bean  
public RouterFunction<ServerResponse> routerFunction(ServiceHandler handler) {  
return RouterFunctions.route(RequestPredicates.GET("/service/{id}"), handler::getService)  
.andRoute(RequestPredicates.GET("/services"), handler::getServices)  
.andRoute(RequestPredicates.POST("/service"), handler::createService)  
.andRoute(RequestPredicates.PUT("/service/{id}"), handler::updateService)  
.andRoute(RequestPredicates.DELETE("/service/{id}"), handler::deleteService);  
}  
```

  

# Handler Function

- **단일 입력과 단일 출력을 가지며, Spring WebFlux Framework가 HTTP 요청을 처리하기 위해 호출하는 메서드입니다**

.

- Handler Function은 Router Function에 매핑됩니다. Router Function은 HTTP 요청을 Handler Function으로 라우팅하는 Spring WebFlux Framework의 메서드입니다.

  
```java
@Service  
public class ServiceHandler {  
private final ServiceRepository serviceRepository;  


public ServiceHandler(ServiceRepository serviceRepository) {
    this.serviceRepository = serviceRepository;
}

public Mono<ServerResponse> getService(ServerRequest request) {
    return serviceRepository.findById(request.pathVariable("id"))
            .flatMap(service -> ServerResponse.ok().body(BodyInserters.fromValue(service)))
            .switchIfEmpty(ServerResponse.notFound().build());
}

public Mono<ServerResponse> getServices(ServerRequest request) {
    return ServerResponse.ok().body(serviceRepository.findAll(), Service.class);
}

public Mono<ServerResponse> createService(ServerRequest request) {
    return request.bodyToMono(Service.class)
            .flatMap(service -> serviceRepository.save(service))
            .flatMap(service -> ServerResponse.created(URI.create("/service/" + service.getId())).build());
}

public Mono<ServerResponse> updateService(ServerRequest request) {
    return request.bodyToMono(Service.class)
            .flatMap(service -> serviceRepository.findById(request.pathVariable("id"))
                    .flatMap(existingService -> {
                        existingService.setName(service.getName());
                        existingService.setDescription(service.getDescription());
                        return serviceRepository.save(existingService);
                    }))
            .flatMap(service -> ServerResponse.noContent().build())
            .switchIfEmpty(ServerResponse.notFound().build());
}

public Mono<ServerResponse> deleteService(ServerRequest request) {
    return serviceRepository.deleteById(request.pathVariable("id"))
            .flatMap(result -> ServerResponse.noContent().build())
            .switchIfEmpty(ServerResponse.notFound().build());
}


}
```


### Mono

- **Reactor 라이브러리에서 제공하는 Reactive Streams의 Publisher 중 하나로 오직 ‘0개 또는 하나의 데이터항목 생성’하고 이 결과가 생성되고 나면 스트림이 종료되면 결과 생성을 종료합니다.**
- Mono를 사용하여 비동기적으로 결과를 반환하면 해당 결과를 구독하는 클라이언트는 결과가 생성될 때까지 블로킹하지 않고 다른 작업을 수행할 수 있습니다.

  
```java
public Mono<String> getData() {

_// perform some database or API call to get data_

return Mono.just("example data");  
}  
```


  

### Flux

- **Reactor 라이브러리에서 제공하는 Reactive Streams의 Publisher 중 하나로 Mono와 달리 ‘여러 개의 데이터 항목’를 생성하고 스트림이 종료되면 결과 생성을 종료합니다.**
- 비동기 작업을 수행하면 작업이 완료될 때까지 블로킹하지 않고 다른 작업을 수행할 수 있습니다.
- Spring WebFlux에서 Flux를 사용하여 HTTP 요청을 처리하는 경우, 요청을 수신한 즉시 해당 요청을 처리하고 결과를 생성하는 대신 결과 생성이 완료될 때까지 다른 요청을 처리할 수 있습니다.

  
```java
public Flux<String> getStreamedData() {

_// perform a continuous stream of data_

return Flux.just("streamed data 1", "streamed data 2", "streamed data 3");  
}  
```


  

### Spring MVC와 Spring Webflux의 차이점

![[image 358.png]]

  

**Webflux랑 SQL Mapper(MyBatis)는 함께 사용이 불가능 한가?**

- 함께 사용하기가 어렵다. WebFlux의 경우는 비동기적이고 논 블로킹 방식으로 동작하며, MyBatis는 동기적 블로킹 방식으로 동작하기 때문에 같이 사용하면 MyBatis가 디폴트로 사용하는 스레드 풀을 블로킹하게 되어 전체적인 성능에 악영향을 미칠 수 있다고 합니다.

 **그럼 Webflux랑 ORM(Object-Relational Mapping)는 함께 사용이 가능한가?**

- WebFlux는 비동기 및 논 블로킹 방식으로 동작하며, ORM은 동기적인 블로킹 방식으로 동작하므로 함께 사용하면 전체적인 성능에 악영향을 미칠 수 있습니다.

**그럼 SQL Mapper, ORM 말고 무엇과 Webflux를 함께 사용해야하는가?**

- RDBMS를 사용하는 것이 아닌 MongoDB를 함께 사용하는것이 일반적입니다. MongoDB는 비동기적이고 논 블로킹 방식으로 동작하기 때문입니다.