  

### Divide & Conquer

어떤 문제를 유사한 형태를 가지는 더 작은 크기의 서브 문제들로 나눈 후 이들을 재귀적으로 같은 방식으로 해결한 뒤 각 서브 문제들을 해결한 결과를 활용하여 원래 문제를 해결하는 방식

  

![[divide_and_conquer.png]]

  

divide & conquer 활용사례

>> merge sort, quick sort, binary search 등등

  

### merge sort 예시

![[merge_sort.png]]

1. 위와 같은 배열이 있을 때 전체를 반으로 자르고 더 이상 자를 수 없을 때 까지 반으로 잘라준다.

![[merge_sort2.png]]

1. 위와 같이 계속 자른 후 더 이상 자를 수 없을 때 여기서 merge sort를 진행

![[merge_sort3.png]]

1. merge sort 완료 되면 2번에서 진행한 merge sort의 짝이 되는 divide도 동일하게 merge sort를 진행

![[merge_sort4.png]]

1. 두 개의 divide가 모두 merge sort가 끝났으므로 다시 combine해서 merge sort를 진행
2. 이런식으로 최소 단위에서부터 merge sort를 진행해서 최종적으로 전체 배열의 merge sort를 완성하는 개념

  

### merge sort의 시간 복잡도 : O(N*logN)



**출처 : 유튜브 쉬운코드 https://www.youtube.com/@ezcd