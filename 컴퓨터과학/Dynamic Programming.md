  

  

![[optimization_problem.png]]

value의 개념 : 소요시간과 수익이 value가 된다.

solution의 개념 : 가장 빨리 도착하는 경로와 주식을 사고 파는 시점이 된다.

  

### Dynamic programming의 개념

![[dynamic_programing.png]]

  

### **Dynamic programming의 두 가지 접근방식**

![[dynamic_programing_접근방식.png]]

Top-Down 방식

>> 재귀적으로 함수를 실행해서 리턴값을 저장해놓고 동일한 함수가 나오면 저장한 값을 리턴해준다.

  

Bottom-Up 방식

>> 작은 단위의 for 루프를 통해 값을 구하고 해당 값을 테이블에 계속 채워넣어서 저장해 놓는다.

  

주로 Bottom-Up 방식을 많이 사용

### **Dynamic programming의 알고리즘 설계 순서**

![[dynamic_programing_알고리즘_설계.png]]

  

### DP 예제 : Climbing Stairs

![[climbing_stair.png]]

주어진 문제를 subproblem으로 나누고 그 subproblem들 각각의 optimal solution을 활용해서 원래 problem의 optimal solution을 구할 수 있을까?

  

![[climbing_stair2.png]]

한 번에 한 스텝 혹은 두 스텝만 오를 수 있으므로 정상인 n번째 계단에 도달하기 한 개 아래와 두 개 아래의 계단까지 도달하는 경우의 수를 구해서 더해주면 n 번째 계단까지 도달하는 경우의 수를 구할 수 있음

  

즉 f(n) = f(n-1) + f(n-2)

  

### Climbing stair Top-Down 방식

겹치는 subproblem들이 많아서 동일한 input에 대한 function call은 처음 한번만 계산하고 결과를 메모 해 뒀다가 이후에 재사용

![[climbing_stair_top_down.png]]

그림에서 보듯이 겹치는 애들은 저장을 해두고 재사용해서 필요없는 연산을 줄임

  

memoization은 function call 결과를 저장해서 이후에 같은 input에 대한 호출은 저장한 결과를 사용하는 것

  

**Climbing stair Bottom-Up 방식**

제일 작은 문제부터 시작해서 for 루프를 통해 중첩해서 문제 해결

  

![[climbing_stair_bottom_up.png]]

먼저 tabular에 n=1일 때의 경우의 수와 n=2 일 때의 경우의 수를 저장

이후에 for 루프를 통해 숫자를 점차 늘려가면서 최종적으로 n까지의 경우의 수를 구함

  

tabulation은 작은 subproblem부터 시작해서 최종 problem 결과를 도출 할 때 까지 결과를 차례대로 기록하는 것

  

### climbing stair 시간복잡도

![[climbing_stair_시간복잡도.png]]

  

### DP 예제2 : Maximum Subarray

![[maximum_subarray.png]]

가장 큰 합계를 구해야 하기 때문에 optimization problem이다.

  

모든 경우를 다 확인해서 찾아보는 것은 비효율적이다.

  

DP를 적용하려면 먼저 어떻게 subproblem으로 나눌것인가?

  

1. 배열에서 max sum을 가지는 subarray를 max subarray라고 가정
2. i 번째 정수가 추가되기 전의 배열에도 max subarray가 있었을 것

>> 배열의 마지막 숫자가 추가 되기 전, 배열에 i-1가 마지막 숫자일 때에도 max subarray가 있었다는 뜻

1. i 번째 정수가 추가된 후에 배열의 max subarray는 어떤 변화가 있을까?

가. i번째 정수가 추가 된 후에도 max subarray는 달라지지 않음

나. i번째 정수 추가 후 max subarray가 달라진 경우

>> 새로운 max subarray는 i번째 정수를 포함한다는 뜻

1. 만약 3번의 나와 같은 상황일 때 다시 i번째 정수를 포함하는 subarray 중에 합계가 가장 큰 subarray를 찾아야 한다는 뜻
2. i번째 정수가 추가되기 전, (i-1)번째 정수를 포함하는 max subarray가 있었을 것이다.
3. i번째 정수를 포함하는 max subarray는 직전에 i-1번째 정수를 포함한 max subarray에 i번째 정수가 추가된 형태이거나, i 홀로 있는 형태일 것이다.
4. 이럴 때 i번 째 정수까지의 배열에서 i번째 정수를 포함하는 max subarray의 합계를 cur_max(i)라고 하고
5. i번째 정수까지의 배열에서 max subarray의 합계를 total_max(i)라고 할 때 결국 우리가 구하는 값은 total_max(i)이다.
6. total_max(i) = max(total_max(i-1), cur_max(i)

cur_max(i) = max(i번째 정수 + cur_max(i-1), i번째 정수)

>> total_max(i)는 직전의 total_max(i-1)과 cur_max(i)중에 큰 값이고 cur_max(i)는 i번째 정수 + 직전의 cur_max 값과 i번째 홀로 있는 값중에 더 큰 값을 가진다.

  

### Maximum Subarray Bottom-Up 방식

![[maximum_subarray_bottom_up.png]]

for 루프를 돌며 i가 증가할 때마다 계속해서 가장 높은 합계값인 total_max를 찾아서 리턴해줌

  

  

### 언제 DP를 사용하면 되는가?

DP를 사용하기 위해서 확인해야할 두 가지 조건

1. optimization problem이 optimal substructure여야 한다.

>> optimization problem의 optimal solution이 subproblem의 optimal solution을 포함하는 구조 >> climbing stair, maximum subarray 같은 방식

여기서 subproblem은 독립적이어야하고 다른 subproblem의 solution에 영향을 줘서는 안된다.

![[optimal_structure.png]]

a에서 C까지의 가장 먼 경로를 찾는다고 가정할 때

a에서 B까지 가장 먼 경로를 구하고 b에서 c까지의 가장 먼 경로를 구해서 더하는 논리로 접근한다면

a > d > c > b를 먼저 구하게 될텐데 이러면 결국 C에 먼저 도달하는 딜레마가 생기게 된다.

즉 하나의 subproblem의 solution이 다른 solution에 영향을 주는 것

  

1. optimization problem이 overlapping subproblems을 가져야 한다.

>> 재귀적 형태의 알고리즘이 동작할 때 동일한 subproblem을 여러 번 해결해야한다면 optimization problem은 overlapping subproblems를 가진다고 표현


**출처 : 유튜브 쉬운코드 https://www.youtube.com/@ezcd