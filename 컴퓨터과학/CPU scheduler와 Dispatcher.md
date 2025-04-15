  

### CPU scheduler : CPU에서 실행될 프로세스를 선택하는 역할

  

### Dispatcher : 스캐줄러에게 선택된 프로세스에게 CPU를 할당하는 역할

  

## 스케줄링의 선점 방식

  

### Nonpreemptive(비선점) scheduling

running 상태인 프로세스가 자발적으로 ready, wait, teminated 중 하나의 상태로 바꾸는 것

즉 OS가 강제적으로 개입하지 않는 스케줄링 방식을 뜻한다.

  

특징 : 신사적, 협력적(cooperative / 스스로 양보하는 느낌), 느린 응답성

  

### **preemptive(선점) scheduling**

기본적로 선점 스케줄러는 비선점 스케줄러가 하는 일을 모두 하지만 추가적으로 개입하는 경우가 있음

ex) running 중인 프로세스가 정해진 timeslice를 모두 사용하면 스케줄러가 개입하여 ready상태로 바꿈

  

특징 : 적극적, 강제적, 빠른 응답성, 데이터 일관성 문제

  

## 스케줄링 알고리즘

  

FCFS(first-come, first-served) : 먼저 도착한 순서대로 처리

  

SJF(shortest-job-first) : 프로세스의 다음 CPU burst가 가장 짧은 프로세스부터 실행

  

SRTF(shortest-remaining-time-first) : 남은 CPU burst가 가장 짧은 프로세스부터 실행

>> 프로그램 실행 중간에 CPU burst가 짧은 프로세스가 새로 생기면 짧은 애부터 먼저 처리

ex) P1이 10초의 cpu burst를 가지고 있었고 실행 4초 후 6초정도 남았을 때 3초의 cpu burst를 가진 P2가 새로 나타나면 P2부터 처리해줌

  

Priority : 우선순위가 높은 프로세스부터 실행

  

RR(round-robin) : time slice로 나눠진 CPU time을 번갈아가며 실행

  

Multilevel queue : 프로세스들을 그룹화해서 그룹마다 큐를 두는 방식

>> 프로세스들을 우선순위에 따라 그룹화 하거나 cpu burst에 따라 그룹화하는 등 어떠한 기준에 따라 그룹화하여 그룹별로 큐를 두고 그룹별 큐마다 스케줄링방식을 적용함