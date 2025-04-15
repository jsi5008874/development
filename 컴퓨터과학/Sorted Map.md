### Sorted Map의 정의

key : value를 key에 대하여 정렬하여 저장하는 Map

  

java에서 Sorted Map은 interface(ADT 개념)이고 이를 구현한 구현체 class(DT 개념)는

ConcuurentSkipListMap

TreeMap

  

  

정렬을 해주니까 Hash Map 보다 Sorted Map이 더 좋아보일수도 있지만

Sorted Map은 정렬을 얻고 시간을 잃었다.

즉 삽입, 삭제, 검색이 Hash Map이 Sorted Map 보다 더 빠르다.