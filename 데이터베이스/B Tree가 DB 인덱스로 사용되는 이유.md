  

### B Tree의 시간 복잡도

  

![[image 219.png]]

B tree 시간 복잡도 : O(log N)

self-balancing BST 시간 복잡도 : O(log N)

  

둘 다 시간 복잡도가 같은데 왜 B Tree를 사용하는 것인가?

  

### DB 저장 및 입출력 특징

![[image 220.png]]

DB는 secondary storage(SSD, HDD)에 저장이 된다.

secondary storage의 특징은 처리 속도가 느리고 용량이 크다는 점이 있고

입출력을 block 단위로 한다.

그림 상 내가 원하는 데이터가 노란 영역이더라도 노란 영역이 포함된 빨간 block 전체를

읽어와야 한다는 것이다.

이는 불필요한 데이터까지 읽어와야 하는 단점이 있다.

  

![[image 221.png]]

block 단위로 읽고 쓰기 때문에 연관된 데이터를 모아서 저장하면 더 효율적인 활용이 가능하다.

  

### AVL Tree vs B Tree 인덱스 성능 비교

![[image 222.png]]

  

**AVL Tree 인덱스**

![[image 223.png]]

AVL 트리는 b=5를 찾기 위해 총 네 번 secondary storage에 접근 했다.

1. 3번 노드 이동 할 때
2. 4번 노드 이동할 때
3. 5번 노드 이동할 때
4. b=5 데이터 접근할 때

  

**B Tree 인덱스**

![[image 224.png]]

B Tree 인덱스는 총 두 번 접근했다.

  

**두 트리의 차이점**

![[image 225.png]]

먼저 자녀 노드의 수이다.

B Tree는 자녀 노드의 수를 2개 이상 가질 수 있다

따라서 AVL에 비해 탐색 범위를 더 빠르게 좁힐 수 있다는 장점이 있고

File IO를 최대한 줄여서 입출력 시간을 단축할 수 있다는 장점이 있다.

  

![[image 226.png]]

그리고 노드의 데이터 수도 B Tree가 더 많이 가지고 있어서 입출력의 효율도 좋다.

AVL은 그림과 같이 8을 가져올 때 8하나만 유효한 데이터이고 같은 블럭의 데이터는 버려야하는데

B Tree는 블럭으로 가져와도 같은 노드에 여러 키가 존재해서 한번의 입출력으로 여러 개의

유효 데이터를 얻을 수 있다.

  

### B Tree가 효율이 좋은 이유(101차 B Tree 예시)

![[image 227.png]]

Best case 이긴 하지만 level 3 밖에 안되지만 1억 개가 넘는 데이터를 저장할 수 있다.

  

![[image 228.png]]

worst case 여도 26만 개의 데이터 저장 가능

  

그러나 AVL, Red Black Tree 등 이진 트리 기반의 트리를 사용했을 때는?

레벨 3이라면 몇 개 저장 못한다.

  

이렇게 낮은 레벨을 가져도 수 많은 데이터를 저장 할 수 있어서

B Tree를 index로 사용하는 것이다..

  

정리를 하자면

![[image 229.png]]



**출처 : 유튜브 쉬운코드 https://www.youtube.com/@ezcd