
redis 캐시그룹 : realPrice
캐시 key : buildingCd 숫자 값
캐시 value : realPriceInquiryResponse 객체


**기존로직**
현재 부동산 시세조회는 hyphen 외부 API통신으로 검색한 주소의 시세를 받아서 조회했다.

주소입력 > 외부 API로 단지 코드 조회(buildingCd)  > 단지 코드를 기반으로 외부 API로
시세조회 



주소별 시세를 redis로 캐싱해서 성능을 올리기 위해 리팩토링

redis 캐싱 전략(Look Aside)

1.  redis cache store에 검색하는 데이터가 있는지 확인(cache store에 데이터 있다면 바로 데이터 사용)
2. redis cache store에 없을 경우 DB(hfemp_estate_supply_area)에서 데이터 조회
3.  DB에서 가져온 데이터를 cache store에 업데이트


# 환경 설정

**build.gradle에 의존성 추가**
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
    >> springDataRedis 모듈을 사용하여 Redis와 객체 매핑을 처리해주고
	    Redis 서버와 쉽게 연결할 수 있도록 기본 설정과 자동 설정 제공

**yml redis 속성 추가**
cache:  
  type: redis  
  redis:  
    cache-null-values: true  
  
redis:  
  host: hello-redis-ha-haproxy.redis-ns  
  port: 6379


# Configuration 클래스 설정

![[Pasted image 20241220094057.png|925]]

**RedisConnectionFactory**
Redis 서버와 연결을 생성 및 관리해주는 인터페이스

어플리케이션 서버와 Redis 서버 간의 데이터 송수신을 하는 클라이언트로
Lettuce와 Jedis가 있음
Lettuce : 비동기 및 이벤트 기반의 통신으로 높은 성능
Jedis : 동기 방식으로 안정성이 높지만 처리 능력이 좀 부족하다.

따라서 해당 코드에는 성능이 좋은 Lettuce를 사용했음

**RedisCacheManager**
Redis를 캐시 저장소로 사용하는 캐시 관리 기능 제공

지정된 이름으로 Redis 캐시를 생성 및 관리
TTL(Time-To-Live, 생명주기) 설정
직렬화/역직렬화 지원

RedisCacheManager를 통해 realPrice라는 이름으로 캐시 키를 생성하고 생명주기를 설정

Map<String, RedisCacheConfiguration> cacheConfigurations = new HashMap<>();  
cacheConfigurations.put("realPrice", getRealPriceCacheConfiguration());

해당 코드를 통해 key : realPrice, value : getRealPriceCacheConfiguration() >> 동적 TTL
구조로 Redis에 저장


**getRealPriceCacheConfiguration 메서드**
매주 목요일 오후 11:30에 Redis에서 캐시들이 삭제되도록 설정
(일주일 단위로 부동산 시세가 변동해서 매주 목요일에 기존 캐시 데이터를 삭제하는 것)

현재시간 기준으로 이번 주 목요일 오후 11:30까지의 시간차를 계산해서 반환하는 메서드

따라서 Redis Configuration 클래스의 역할은
1. 어플리케이션 서버와 Redis 서버간의 통신을 관리
2. RealPrice라는 이름의 캐시를 매주 목요일 오후 11:30까지의 TTL을 가지도록 설정



# 비즈니스 로직 개선
![[Pasted image 20241220101945.png]]
![[Pasted image 20241220101957.png]]

**admin, app 프로젝트 분기처리**
요구사항 중 app 프로젝트에서 요청이 들어올 시 Redis를 활용하고
admin 프로젝트에서 요청이 들어올 시 Redis 미활용 기존 로직대로 API 통신

요구사항을 충족하기 위해 URL 쿼리 파라미터를 활용했다.

![[Pasted image 20241220102154.png]]
admin 프로젝트에서 api 프로젝트로 통신할 때 URL에 "cache=False"를 추가

Redis를 적용하는 api 프로젝트에서는 service 메서드에 Boolean cache 파라미터를 추가

Boolean cache가 true이면 redis를 적용하고 false이면 admin 프로젝트에서 온 요청으로
api 통신을 하도록 분기처리


**Look Aside 전략 구현**

Cache searchCache = cacheManager.getCache("realPrice");  
String cacheKey = realPriceInquiryRequestDto.getBuildingCd();  
  
if(cache == null || cache) {  
    RealPriceInquiryResponse cacheData = searchCache.get(cacheKey, RealPriceInquiryResponse.class);  
  
    if(cacheData != null) {  
        RealPriceInquiryResponseDto realPriceInquiryResponseDto = RealPriceInquiryResponseDto.builder()  
                .errYn("N")  
                .errMsg("")  
                .outH0001(cacheData)  
                .build();  
  
        return new ResponseModel<>(realPriceInquiryResponseDto);  
    }

1. 우선 realPrice::buildingCd로 저장되어 있는 캐시가 있는지 조회
2. 있으면 RealPriceInquiryResponseDto 객체에 담아서 return
3. 만약 조회 결과가 없으면 기존 로직인 API 통신을 실시

if(!realPriceInquiryResponseDto.getErrYn().equals("Y")){  
    RealPriceInquiryResponse realPriceInquiryResponse = realPriceInquiryResponseDto.getOutH0001();  
    searchCache.put(cacheKey, realPriceInquiryResponse);  
}

4. API 데이터 통신 후 Response의 결과가 error가 아닐 때만 Redis에 저장


![[HF-DT-20 테스트 수행내역서_부동산 시세조회 API Redis 적용.xlsx]]

**그 결과 응답 속도 약 99.8% 성능 개선**
5번의 테스트 API 평균 응답 속도 : 3,494ms
5번의 Redis 평균 응답 속도 : 2.4ms