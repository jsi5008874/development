  

### Partitioning

DB table을 더 작은 table들로 나누는 것

  

### Partitioning의 종류

![[image 187.png]]

  

  

### Vertical Partitioning

![[image 188.png]]

예시를 보면 ARTICLE 테이블에서 content를 제외한 컬럼을 조회하는 쿼리이다.

content는 다른 컬럼에 비해 사이즈가 큰 컬럼인데

문제는 쿼리 상 content를 빼고 조회를 한다해도 우선 DB에서 해당 row에 해당하는 모든 컬럼의 값을 가져온 후 필터링해서 조회결과를 보여준다.

ex) where id=1일 때 id가 1인 id, title…,comment_cnt를 조회하는데

이 때 우선 content까지 포함된 id=1인 row의 모든 데이터를 메모리에 올린 후

필터링해서 content를 제외한 컬럼의 값을 조회 결과로 내어준다.

  

1개일 때는 상관없지만 row가 많아지면 입출력에 부담이 생기게 된다.

따라서 vertical partitioning을 사용하게 된다.

  

![[image 189.png]]

이런식으로 컬럼을 기준으로 잘라서 입출력에 부담이 생기지 않도록 해주는 것이다.

정규화와 비슷하다.

  

### Horizontal Partitioning

![[image 190.png]]

  

예시

![[image 191.png]]

유튜브 사용자의 구독 채널을 보여주는 테이블이 있다고 가정

이 때 SUBSCRIPTION 테이블이 가질 수 있는 최대 크기는 10억

>> user가 100만명이고 channels가 1천개 일 때 테이블의 최대 크기는 user*channels이므로

10억이 된다.

  

![[image 192.png]]

여기서 문제점은 테이블이 커질수록 인덱스도 커지고

점점 처리 시간도 길어진다는 점이다.

서비스의 규모가 커질 수록 체감이 될 정도로 성능저하가 발생하게된다.

  

### Hash Based Horizontal Partitioning

![[image 193.png]]

Hash Function을 통해 테이블을 나누는 것이다.

예시를 보면 user_id를 hash function에 돌려서 나오는 값에 해당하는 테이블에 저장

ex) aaaaaaa의 hash function 결과가 0이면 SUBSCRIPTION_0 테이블에 저장

yeah의 hash function 결과가 1이면 SUBSCRIPTION_1 테이블에 저장

  

하지만 **Hash Based Horizontal Partitioning은 한번 partition이 나눠져서 사용되면**

이후에 partition을 추가하기 까다롭기 때문에 신중하게 설계해야한다.

  

예시를 기준으로 hash 결과를 0, 1, 2, 3으로 늘린다면 기존에 0, 1 테이블에 있던 애들을

다시 hash에 돌려서 나온 결과값으로 옮겨주는 작업이 필요하기 때문에

서비스를 하고 있는 중에 테이블을 이동시키고 하는 과정이 매우 까다롭다.

  

  

### Sharding

![[image 194.png]]

  

![[image 195.png]]

![[image 196.png]]

  

### Replication

![[image 197.png]]

DB 서버를 똑같이 복제해서 만들고

원래 서버를 master/primary/leader 라고 부르고

복제된 서버를 slave/secondary/replica 라고 부른다.

  

이렇게 복제해서 사용하면 read를 나눠서 서버 부하를 분산 시키는 효과도 있고

한 서버가 다운되더라도 서비스가 지속될 수 있게 해준다

  

  

  

![[image 198.png]]



**출처 : 유튜브 쉬운코드 https://www.youtube.com/@ezcd