### **Hash Map 내부구현 방법**

1. 배열을 사용하여 구현
2. 배열을 저장하기 위해 key의 Hash를 구함
3. Hash Function으로 key의 Hash를 구함

이때 구해진 Hash는 정수여야함(배열의 index는 정수이기 때문에 Hash Function의 결과는 정수로 나와야 한다.)

1. 구해진 Hash를 modular(%) 한 값을 index로 사용

>> 모듈러연산은 구한 Hash를 Hash Map 배열의 사이즈내의 정수로 맞게 변환해주는 절차

Hash Function을 통해 나온 정수값을 배열의 사이즈로 나눈 나머지를 인덱스로 사용한다.

(만약 배열 사이즈가 5라면 Hash Function의 결과값을 5로 나눈 나머지가 인덱스가 된다.)

array[hf(key)% M] = value

hf : hash function

M : hash map 사이즈

  

### **해시충돌(Hash Collision)**

개념 : 서로 다른 key들이 같은 Hash를 가질 때 발생

key1 ≠ key2 But hf(key1) = hf(key2)

>> 키값은 다르지만 Hash값이 같음

  

Hash Map에서 또 다른 의미의 해시 충돌

hf(key1) ≠ hf(key2) But hf(key1)% M = hf(key2)% M

>> Hash값이 다르지만 모듈러연산을 했을 때 값이 같음

  

### 해시충돌이 발생하는 이유

1. perfect hash function 구현의 어려움

hash function의 input과 output 사이즈 차이때문에 사실상 perfect hf는 구현 불가

ex) 천글자의 문자열을 input으로 넣어도 hf를 거치면 4byte의 정수를 output으로 내야하기때문

즉 input은 무한대에 가까운데 output은 정해져있다.

1. key의 사이즈에 비해 hash map의 사이즈가 작기 때문

예시 : 내 서비에서 가입한 사람의 핸드폰 번호를 key 그 사람의 정보를 value로 Hash map에 저장

이 때 대충 국민 1명당 핸드폰 1개로 했을 때 key의 범위는 5,000만이지만

내 서비스를 누가 사용할지는 모르기 때문에 Hash Map의 사이즈는 어림잡아 정한다.

사이즈를 2,000으로 잡고 생성했다.(Hash Map의 사이즈를 Key의 범위와 같게 만들면 메모리를 비효율적으로 사용하게 될 확률이 너무 크기 때문에 서비스 이용자를 어림잡아 계산해서 사이즈를 정한다. 이후에 사이즈가 부족하면 점점 늘려가는식으로 구현한다.)

이 때 5,000만이라는 가능성을 가진 key의 범위에서 모듈러 연산을 통해 2,000의 사이즈로

욱여넣는다면 모듈러 연산과정에서 충돌이 생길 수 밖에 없다.


**출처 : 유튜브 쉬운코드 https://www.youtube.com/@ezcd