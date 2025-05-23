  

모든 노드의 왼쪽 서브 트리는 해당 노드의 값보다 작은 값들만 가지고

모든 노드의 오른쪽 서브트리는 해당 노드의 값보다 큰 값들만 가진다

  

![[이진탐색트리.png]]

  

이진 탐색 트리의 최소값 : 트리의 가장 왼쪽에 존재

이진 탐색 트리의 최대값 : 트리의 가장 오른쪽에 존재

  

### 중위 순회(inorder traversal)

- 재귀적으로 왼쪽 서브 트리 순회
- 현재 노드 방문(e.g. 값 출력)
- 재귀적으로 오른쪽 서브 트리 순회

  

1. 왼쪽 서브 트리부터 순회하기 때문에 20 > 15 > 3 여기서 3의 자식 노드가 없어서 3 출력

1. 5로 올라가서 5를 출력 후 오른쪽인 15로 이동 > 10 방문 10에 자식 노드가 없어서 10 출력
2. 10출력 후 15로 올라가서 15 출력 후 17 방문, 17 자식 노드 없어서 17 출력
3. 17 출력 후 15로 이동 15는 이미 출력해서 그 부모노드인 5로 이동, 5도 출력했으니 20으로 이동
4. 20 출력 후 20의 왼쪽 서브노드는 모두 방문해서 출력했으므로 오른쪽 서브노드로 이동
5. 똑같은 방식으로 오른쪽 서브노드도 모두 출력

  

### 결과적으로 출력 값은 3, 5, 10, 15, 17, 20, 30, 40, 50 순서대로 출력됨

  

### 전위 순회(preorder traversal)

- 현재 노드 방문(e.g. 값 출력)
- 재귀적으로 왼쪽 서브 트리 순회
- 재귀적으로 오른쪽 서브 트리 순회

  

출력값 : 20, 5, 3, 15, 10, 17, 50, 30, 40

  

### 후위 순회(postorder traversal)

- 재귀적으로 왼쪽 서브 트리 순회
- 재귀적으로 오른쪽 서브 트리 순회
- 현재 노드 방문(e.g. 값 출력)

  

출력값 : 3, 10, 17, 15, 5, 40, 30, 50, 20

  

### 노드의 successor(후임자)

- 해당 노드보다 값이 큰 노드들 중에서 가장 값이 작은 노드
- 20의 successor : 30
- 17의 successor : 20
- 10의 successor : 15

  

### 노드의 predecessor(선임자)

- 해당 노드보다 값이 작은 노드들 중에서 가장 값이 큰 노드
- 20의 predecessor : 17
- 10의 predecessor : 5
- 40의 predecessor : 30

  

### 이진탐색트리의 삽입

- 루트노드부터 비교를 시작해서 삽입을 하는데 크기가 크면 오른쪽 작으면 왼쪽으로 보내서 저장

ex) 루트 노드는 50이라고 가정

1. insert(70)을 했을 때 50과 70을 비교 > 70이 크니까 오른쪽으로 저장
2. insert(90)을 했을 때 50과 90 비교 > 오른쪽으로 이동 > 70과 90을 비교 > 90이 크니까 오른쪽에 저장

이런식으로 크면 오른쪽 작으면 왼쪽으로 보내면서 저장을 한다.

  

### 이진탐색트리의 삭제

- 삭제하려는 노드가 있는지 검색 후 있으면 삭제
- 자녀가 없는 노드 삭제 시 삭제 될 노드를 가리키던 레퍼런스를 가리키는 것이 없도록 처리

>> leaf 노드 삭제 시 그 부모노드가 가리키던 레퍼런스도 없애는 것

- 자녀가 하나인 노드 삭제 시 삭제 될 노드를 가리키던 레퍼런스를 삭제 될 노드의 자녀를 가리키게 변경

>> 예를 들어 30을 삭제 시 50이 40을 가리키도록 함(20이 없다고 가정)

- 자녀가 두개인 노드 삭제 시 삭제 될 노드의 오른쪽 또는 왼쪽 서브 트리에서 제일 값이 작은 노드가 삭제 될 노드를 대체(오른쪽으로 할 지 왼쪽으로 할 지는 정해줘야 함)

>> 50 삭제 시 70으로 대체 됨

  

### 이진 탐색 트리의 시간복잡도

![[이진탐색트리_시간복잡도.png]]

  

### 이진 탐색 트리의 장점

- 삽입 삭제가 유연하다
- 값의 크기에 따라 죄우 서브트리가 나눠지기 때문에 삽입/삭제/검색이 빠르다
- 값의 순서대로 순회 가능하다

  

### 이진 탐색 트리의 단점

- 트리가 구조적으로 한쪽으로 편향되면 삽입 삭제 검색 등등 여러 동작들의 수행시간이 악화된다.

이 문제를 해결하기 위해 스스로 균형을 잡는 이진 탐색 트리가 사용됨(AVL 트리, Red-Black 트리)


**출처 : 유튜브 쉬운코드 https://www.youtube.com/@ezcd