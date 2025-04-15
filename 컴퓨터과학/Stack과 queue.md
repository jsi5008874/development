ADT의 관점에서만 설명

  

### 스택(stack)

LIFO(Last In First Out) 형태로 데이터를 저장하는 구조

  

스택의 주요동작

-push : 스택 안에 데이터를 넣는 것

-pop : 스택에서 데이터를 빼오는 것(가장 최근에 들어간 데이터를 꺼내옴)

-peek : 스택에서 가장 최근에 들어간 데이터를 확인하는 것(빼오지는 않음)

  

### 큐(queue)

FIFO(First In First Out) 형태로 데이터를 저장하는 구조

  

큐 주요 동작

-enqueue : 큐에 데이터를 넣는 것

-dequeue : 큐에서 데이터를 빼오는 것(가장 먼저 들어간 데이터를 빼옴)

-peek : 큐에서 가장 먼저 들어간 데이터를 확인하는 것(빼오지는 않음)

  

스택 사용사례 : stack memory & stack frame

큐 사용사례 : producer/consumer architecture

  

### 기술 문서에서 큐를 만났을 때 팁

항상 FIFO를 의미하지는 않음

ex) 멀티태스킹에서 P1, P2, P3가 CPU에서 실행대기 위해 대기를 할 때

ready queue에서 대기를 하는데 여기서는 FIFO가 아니라 priority queue 즉 우선순위에 따라

먼저 CPU에 진입하기도 한다. 여기서는 큐가 FIFO가 아닌 그냥 대기열의 느낌이다.

이처럼 queue라고 해서 항상 FIFO가 아닐 수 있다.

  

### 스택 / 큐 관련 에러와 해결 방법

StackOverFlowError : 스택 메모리 공간을 다 썻을 때 발생하는 에러

>> 대부분이 재귀함수(recursive function, 자기 자신을 계속해서 호출하는 함수)에서 탈출 못해 발생

탈출 조건을 잘 써주면 해결 가능

  

OutOfMemoryError : java의 힙 메모리를 다 썻을 때 발생

>> 큐에 데이터가 계속 쌓이기만 한다면 발생 >> 큐 사이즈를 고정해서 해결

>> 큐가 다 찼을 때는 어떻게?

-예외(exception) 던지기

-특별한 값(null or false)를 반환

-성공할 때 까지 영원히 스레드 블락(대기한다는 뜻)

-제한된 시간만 블락되고 그래도 안되면 포기

>> LinkedBlockingQueue 클래스를 사용해보면 됨