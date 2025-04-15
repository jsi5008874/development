![[화면 캡처 2025-03-28 121452.png]]

회사 시스템의 보안 취약점 점검을 받았는데 네트워크 패킷에서 패스워드가 평문으로 노출되는 문제가 식별되었다.

처음 봤을 때는 이해가 가지 않았다.
현재 우리의 서버는 TLS를 적용중인데 어떻게 패킷에서 평문으로 데이터가 노출된건지? 라는
의문이 생겼다.

뭐가 문제일까?

# 문제점 파악

문제점을 파악하다보니 용어의 혼선에서 발생된 궁금증이었다는 것을 알았다.
나는 네트워크 패킷이라고 표현이 되어 있어서 TCP 통신 중(Transport Layer) 암호화가 되어있지
않다는 것으로 이해를 했다.
그래서 현재 TLS를 적용중이고 wire shark로 패킷을 확인했을 때 이상 없이 TLS가 작동 중 인것을 확인 했는데 어떤게 문제인지 이해가 되지 않았던 것이다.

![[와이어샤크.png]]
wire shark 화면 캡처본(TLS 프로토콜로 통신중인 것을 알 수 있음)

하지만 보안 업체에서 제시한 문제점은 Transport Layer 통신의 문제가 아닌
application Layer ~ Transport Layer 구간에서 평문으로 노출된다는 의미였던 것이다.

그래서 보안 업체에서 제시한 입력 즉시 암호화 방식(RSA 방식)을 적용하기로 했다.

# 로직 정의

1. 클라이언트 공개키 요청
2. 서버 키페어 생성 >> 키페어 레디스 저장 >> 공개키 응답
3. 클라이언트 공개키로 암호화 후 서버 전송
4. 서버 요청 받으면 레디스에서 해당 공개키 확인 후 있으면 개인키로 복호화, 없으면 요청 거절
5. 서버 개인키 복호화 후 비밀번호 일치 여부 확인 등 로직 실행

**키 관리를 레디스에 저장한 이유는 서버가 쿠버네티스에서 여러 파드로 나눠서 실행되기 때문이다.
(여러 파드로 실행되면 JVM이 따로 실행되기 때문에 중앙 관리 방식이 필요하다.)**


# RSA 구현

# 1. 키 쌍 발급 로직

하나의 쌍으로 이루어지는 _공개키(public key)_ 와 _개인키(private key)_ 를 생성하는 로직

**`java.security`** 패키지를 사용하여 구현한다. **Java의 보안 프레임워크의 핵심**

- 암호화, 해시, 키 생성, 인증서 처리, 서명 등 **암호학 기반 기능들을 제공**하는 클래스들의 모음이다.

```java
    private static final String INSTANCE_TYPE = "RSA";  
  
    // 2048bit RSA KeyPair 생성.  
    public static KeyPair generateKeypair() throws NoSuchAlgorithmException {  
  
        KeyPairGenerator keyPairGen = KeyPairGenerator.getInstance(INSTANCE_TYPE);  
        keyPairGen.initialize(2048, new SecureRandom());  
  
        return keyPairGen.genKeyPair();  
    }  
```

- KeyPairGenerator (공개키/개인키 쌍 생성) 을 사용한다.

- `NoSuchAlgorithmException` : 지정한 알고리즘 이름이 현재 JVM 환경에서 지원되지 않거나 잘못된 경우 발생하는 **체크 예외**다.

- `KeyPair`타입 : PrivateKey와 PublicKey로 이루어져있는 데이터 타입
- `SecureRandom`을 시드로 사용해 보안 수준 향상

2048bit로 RSA암호화 방식을 사용하여 **keyPair**를 생성하는 코드이다.

---

# 2.1 평문 + base64공개키 -> base64암호문 생성 로직 .Java

```java
	private static final String INSTANCE_TYPE = "RSA";
	
	// 평문 + 공개키 Base64로 암호문 생성  
	public static String rsaEncode(String plainText, String publicKey)  
	        throws InvalidKeyException, InvalidKeySpecException, NoSuchAlgorithmException, NoSuchPaddingException, IllegalBlockSizeException, BadPaddingException {  
	  
	    Cipher cipher = Cipher.getInstance(INSTANCE_TYPE);  
	    cipher.init(Cipher.ENCRYPT_MODE, convertPublicKey(publicKey));  
	  
	    byte[] plainTextByte = cipher.doFinal(plainText.getBytes());  
	  
	    return base64EncodeToString(plainTextByte);  
	}  
	//Base64 공개키 -> 공개키로 디코딩  
	public static PublicKey convertPublicKey(String publicKey)  
	        throws InvalidKeySpecException, NoSuchAlgorithmException {  
	  
	    KeyFactory keyFactory = KeyFactory.getInstance(INSTANCE_TYPE);  
	    byte[] publicKeyByte = Base64.getDecoder().decode(publicKey.getBytes());  
	  
	    return keyFactory.generatePublic(new X509EncodedKeySpec(publicKeyByte));  
	}
```

- `Cipher` : Java 보안 API에서 실제 _암호화/복호화_를 수행하는 핵심 클래스  
    AES, RAS, DES 같은 알고리즘을 직접 실행하는 암호 모듈 , 암호화 엔진 이다.
    
- `Cipher cipher = Cipher.getInstance(INSTANCE_TYPE);` : 타입에 따라, 암호화 모드와 패딩 방식이 결정된다.
    
- `convertPublicKey` : base64기반 코드를 실제 키 객체로 변환
    
- `KeyFactory` : 키 복원용 펙토리 객체
    
- `keyFactory.generate...` : 실제 키 객체 생성
    
- `X509EncodedKeySpec` → 공개키 표준 포맷 스펙
    

---

# 2.2 평문 + base64공개키 -> base64암호문 생성 로직 .JS

node-forge 패키지를 사용한다.

base64, encode등 TLS프로토콜(암호화 도구)를 구현한 패키지 이다.

```javascript
<script src="https://cdn.jsdelivr.net/npm/node-forge@1.3.1/dist/forge.min.js"></script>

<script>
/**
 * 서버에서 공개키를 받아서 RSA로 암호화하는 함수
 * @param {string} plainText - 암호화할 평문
 * @param {string} publicKeyBase64 - 서버로부터 받은 Base64 인코딩된 공개키
 * @returns {string} 암호화된 Base64 문자열
 */
function rsaEncryptWithBase64PublicKey(plainText, publicKeyBase64) {
  const forge = window.forge;

  // 1. Base64 디코딩 → DER 바이너리
  const der = forge.util.decode64(publicKeyBase64);

  // 2. DER → ASN.1 파싱 → PublicKey 객체
  const asn1 = forge.asn1.fromDer(der);
  const publicKey = forge.pki.publicKeyFromAsn1(asn1);

  // 3. RSA 암호화
  const encryptedBytes = publicKey.encrypt(plainText, 'RSAES-PKCS1-V1_5');

  // 4. 암호문을 Base64 인코딩해서 반환
  return forge.util.encode64(encryptedBytes);
}
</script>
```

- 클라이언트 단에서 request전달 전, 암호화 하기 위한 코드 / 동작 구성은 `2.1`과 동일 하다

---

# 3. base64암호문 + base64개인키 -> 평문 생성 로직

```java
	private static final String INSTANCE_TYPE = "RSA";

	// 암호문 + 개인키 Base64로 평문 생성  
	public static String rsaDecode(String encryptedPlainText, String privateKey)  
	        throws NoSuchAlgorithmException, NoSuchPaddingException, InvalidKeyException, InvalidKeySpecException, IllegalBlockSizeException, BadPaddingException {  
	  
	    byte[] encryptedPlainTextByte = Base64.getDecoder().decode(encryptedPlainText.getBytes());  
	  
	    Cipher cipher = Cipher.getInstance(INSTANCE_TYPE);  
	    cipher.init(Cipher.DECRYPT_MODE, convertPrivateKey(privateKey));  
	  
	    return new String(cipher.doFinal(encryptedPlainTextByte));  
	}  
	//Base64 개인키 -> 개인키로 디코딩  
	public static PrivateKey convertPrivateKey(String privateKey)  
	        throws InvalidKeySpecException, NoSuchAlgorithmException {  
	  
	    KeyFactory keyFactory = KeyFactory.getInstance(INSTANCE_TYPE);  
	    byte[] privateKeyByte = Base64.getDecoder().decode(privateKey.getBytes());  
	  
	    return keyFactory.generatePrivate(new PKCS8EncodedKeySpec(privateKeyByte));  
	}
```

- `Cipher` 객체를 사용하여 위와 동일하게 동작.
- `PKCS8EncodedKeySpec` → 개인키 표준 포맷 스펙

---

# + 바이너리 데이터를 Base64로 인코딩 .java

```java
  public static String base64EncodeToString(byte[] byteData) {
    return Base64.getEncoder().encodeToString(byteData);
  }
```

- cipher.doFinal(...) 과 같은 코드는 _바이너리 데이터_로 리턴값을 보낸다.

---

# 백로직 및 프론트 검증 완료

### 

_junit_과 _assertj_를 사용하여 검증

- `JUnit` → 테스트 프레임워크
- `AssertJ` → 테스트 결과를 검증(assert)할 때 쓰는 **강력한 assertion 도구**

JS코드는 제외

```java
private static final String PLAIN_TEXT = "키 암/복호화 테스트 123 abc !@#";

@Test  
@DisplayName("RSA 키쌍 생성 및 암/복호화 통합 테스트")  
public void testGenerateKeypairAndEncryptDecrypt() throws Exception {  
    // 키쌍 생성  
    KeyPair keyPair = rsaService.generateKeypair();  
    PublicKey publicKey = keyPair.getPublic();  
    PrivateKey privateKey = keyPair.getPrivate();  
  
    // 공개키, 개인키 → Base64 인코딩  
    String publicKeyBase64 = rsaService.base64EncodeToString(publicKey.getEncoded());  
    String privateKeyBase64 = rsaService.base64EncodeToString(privateKey.getEncoded());  
    System.out.println("공개키Base64 : " + publicKeyBase64);  
    System.out.println("개인키Base64 : " + privateKeyBase64);  
  
    // 암호화  
    String encryptedText = rsaService.rsaEncode(PLAIN_TEXT, publicKeyBase64);  
    System.out.println("RSA암호화 텍스트 : " + encryptedText);  
  
    // 복호화 (개인키 사용해야 함)  
    String decryptedText = rsaService.rsaDecode(encryptedText, privateKeyBase64);  
    System.out.println("RSA복호화 텍스트 : " + decryptedText);  
  
    // 검증  
    Assertions.assertThat(decryptedText).isEqualTo(PLAIN_TEXT);  
}
```

결과  
![Pasted image 20250325102427.png](https://lts.kr/%EC%82%AC%EC%A7%84-%EB%B0%8F-%EB%AC%B8%EC%84%9C/pasted-image-20250325102427.png)

- Base64기반 String 변환 및 암/복호화 테스트 완료
- js암호화도 동일하게 동작 확인


#  분산 서버 환경에서 고려하고있는 **Cache** 전략들

분산 서버란?

"여러 대의 서버(컴퓨터)가 **하나의 시스템처럼 동작**하도록 구성된 환경."

1. 사용자가 Web 브라우저로 접속
2. 로드밸런서가 여러 대 서버 중 하나로 요청 분배
3. 각 서버가 필요한 처리 수행
4. 공통 DB/캐시(Redis) 접근해서 데이터 읽기/쓰기

즉, Spring기반이라면 여러대의 JVM을 띄워놓고 사용자를 분배하는 환경이다!

## 1. 레디스(redis)

## 2. 스티키 세션 전략(Sticky Session)

## 3. 자바의 콘커런트 해쉬(ConcurrentHashMap)

---

# ▶ **분산 서버** 환경에서의 고려해야 할 점

|문제|설명|
|---|---|
|_데이터 일관성 문제_|여러 서버가 동시에 데이터 쓰면 충돌 가능 (ex: 동시성 제어 필요)|
|_네트워크 이슈_|서버 간 통신 실패, 지연(latency) 발생 가능|
|_세션/상태 관리_|Sticky Session, Redis 등 세션 상태관리 전략 필요|
|_장애 전파 위험성_|하나의 장애가 전체 시스템에 파급될 수 있음|

---

# ▶ 각 **Cache**전략 별 설명

## 1. Redis

> 분산 환경에서 공통으로 접근 가능한, 독립적인 "In-memory Key-Value 저장소"

```
Redis는 Remote Dictionary Server.  
메모리 기반의 초고속 데이터 저장소(DB처럼 사용 가능).
```

- **동작 원리:**
    - 서버 여러 대(Spring 인스턴스들)가 **공통된 Redis 서버**에 네트워크를 통해 접속.  
        
    - 각 인스턴스는 키를 통해 Redis에 데이터 저장/조회.  
        
    - Redis는 데이터를 메모리에 저장하여 매우 빠른 읽기/쓰기 성능 제공.  
        
    - 키에 TTL(Time To Live) 설정 가능: 자동 만료.  
        
- **적용 사례:**
    - 세션 저장소(Spring Session).  
        
    - JWT 키 관리.  
        
    - 캐시(Cache) 계층.  
        
    - 분산 락(distributed locking).  
        
- **특징:**
    - 인메모리 기반, 디스크에 영속화 옵션 가능(AOF, RDB).  
        
    - 복제(replication), 클러스터링 지원.  
        
    - Pub/Sub(메시지 브로커 기능)도 있음.

---

## 2. Sticky Session

> "사용자가 항상 같은 서버 인스턴스로 요청을 보내도록 강제하는" 로드 밸런싱 전략

```
Sticky Session은 로드밸런서가 요청을 서버에 무작위로 분산하지 않고,  
특정 사용자는 항상 같은 서버로 라우팅되게 만드는 방식.
```

- **동작 원리:**
    - 최초 요청 시, 서버가 세션 ID(cookie 또는 HTTP 헤더)를 발급.  
        
    - 로드밸런서는 이 세션 ID를 기억하여, 이후 요청을 같은 서버로 보냄.  
        
    - 서버 로컬 메모리에 세션 데이터 저장.  
        
- **적용 사례:**
    - 간단한 구조의 서비스.  
        
    - 세션 기반 인증(Login 상태 유지) 시스템에서 서버 확장이 필요할 때.  
        
    - 임시방편(quick and dirty)으로 분산처리 할 때.  
        
- **특징:**
    - 서버 죽으면 세션도 소멸.  
        
    - 서버 간 데이터 공유는 불가능.  
        
    - 스케일 아웃(확장)에 제약이 있음.

---

## 3. ConcurrentHashMap

> Java 내부에서 제공하는 "스레드-세이프(Thread-safe)한 해시 맵 구현체"

- **정체:**  
    Java Collections Framework에 포함된 동시성-safe한 HashMap.  
    Java 5부터 제공. 내부적으로 세그먼트를 락(lock) 걸어 다중 스레드 동시 접근 허용.  
    
- **어떻게 작동:**
    - 하나의 JVM 인스턴스(Spring 서버) 내에서, 메모리에 데이터를 저장.  
        
    - 다중 스레드 환경에서도 동기화(Concurrency Control)가 되어 있어 안정적.  
        
    - 외부 서버와는 전혀 공유되지 않음 → 오직 '자기 자신' JVM 메모리.  
        
- **적용 사례:**
    - 요청 중 공유되는 데이터 저장.  
        
    - 일시적 데이터 캐시.  
        
    - 스레드 간 상태 저장.  
        
- **특징:**
    - 진짜 빠르다 (로컬 메모리 접근).  
        
    - 서버 죽으면 데이터 사라진다.  
        
    - 여러 서버에 동일한 데이터 보장 불가.

---
# ▶ 정리

1. ConcurrentHashMap은 스레드 세이프하고 매우매우 빠르지만, 분산 서버 환경에서는 서버간 데이터 공유가 불가능하다. - (별도의 셋팅 필요)  
      
    
2. Sticky Session은 Spring의 셋팅을 별도로 변경하지 않고 적용가능하다는 장점이 있다.(로드 벨런서를 사용하여 특정 서버 고정) 하지만, 특정 서버에 트래픽이 몰릴 가능성이 있어 로드 벨런싱 효율이 떨어지게 된다. 또한 특정서버가 죽으면 해당 세션이 소멸되어 찾을 수 없게된다.  
      
    
3. Redis는 위의 단점을 모두 커버할 수 있는 방식이다. 하지만, 서버-서버간 통신이 필요하기 때문에 로컬보다는 느릴 수 있고, 추가적인 인프라 구축 비용이 필요하다. 또한 _SPOF (Single Point of Failure)_ 이 발생할 수 있기 때문에, Redis Cluster, Redis Replication 등의 전략이 필요하다.

SPOF (Single Point of Failure) 이란?

> **시스템에서 단 하나의 장애 지점이 전체 서비스 장애로 이어지는 것**

- 시스템의 특정 컴포넌트가 장애나면 전체가 멈춰버리는 상황.
- 고가용성(HA, High Availability)을 깨뜨리는 주범

---

# ✨ 결론

- Redis인프라가 구성 되어있는 상황에서, 분산 서버 캐시는 Redis가 가장 합리적으로 보인다.


# **Redis** 연결 및 구현 With. Spring

  

# 1. 환경 설정

**build.gradle에 의존성 추가**

```java
implementation 'org.springframework.boot:spring-boot-starter-data-redis'  
```

**yml redis 속성 추가**

```yml
cache:  
type: redis  
redis:  
cache-null-values: true

redis:  
host: `레디스 호스트`
port: `레디스 포트`
```

---

2. RedisConfiguration

- @Configuration으로 redis사용에 필요한 셋팅을 Bean으로 등록할 클래스.

```java
@Configuration  
public class RedisConfiguration {  
  
    @Value("${spring.redis.host}")  
    private String host;  
  
    @Value("${spring.redis.port}")  
    private int port;
}
```

  

---

**RedisConnectionFactory**

Redis 서버와 연결을 생성 및 관리해주는 인터페이스

```java
@Bean  
public RedisConnectionFactory redisConnectionFactory() {  
    return new LettuceConnectionFactory(host, port);  
}
```

- 어플리케이션 서버와 Redis 서버 간의 데이터 송수신을 하는 클라이언트
- 대표적으로 _Lettuce_와 _Jedis_, _Redisson_ 이 있다.

Lettuce

- 비동기 및 논블로킹 I/O를 기반으로 하여 고부하, 다중 스레드 환경에 적합

Jedis

- 블로킹 I/O(동기) 방식을 사용.
- 고부하나 비동기 처리가 중요한 환경에서는 효울이 떨어진다.

redisson

- 단순히 Redis 연결을 관리하는 것을 넘어 분산 락, 분산 컬렉션, 분산 캐시 등 고급 기능을 제공.
- 직접 RedisConnectionFactory로 사용하기보다는 _RedissonClient를 빈으로 등록_하고 이를 통해 분산 락이나 캐시 매니저를 구성.

따라서 해당 코드에는 비동기 성능이 높은 좋은 **Lettuce** 선택.

---


**RedisCacheDefaultConfiguration**

Redis에 저장될 캐시의 기본 직렬화 및 만료 시간 등의 설정을 담당.

```java
private RedisCacheConfiguration redisCacheDefaultConfiguration() {
    return RedisCacheConfiguration
            .defaultCacheConfig()
            .serializeKeysWith(RedisSerializationContext.SerializationPair
                    .fromSerializer(new StringRedisSerializer()))
            .serializeValuesWith(RedisSerializationContext.SerializationPair
                    .fromSerializer(new GenericJackson2JsonRedisSerializer());
}
```

- _serializeKeysWith_ : `StringRedisSerializer`를 사용하여 키를 문자열로 직렬화합니다.
- _serializeValuesWith_ : `GenericJackson2JsonRedisSerializer`와 `ObjectMapper`를 사용해 JSON 형식으로 직렬화
    
    > `GenericJackson2JsonRedisSerializer` : 직렬화 방식 중 하나로, JSON형식을 지원.
    

---

**redisCacheConfigurationMap**

여러 캐시 이름에 대해 각기 다른 TTL(Time To Live)을 동적으로 설정.

```java
private Map<String, RedisCacheConfiguration> redisCacheConfigurationMap() {  
    Map<String, RedisCacheConfiguration> cacheConfigurations = new HashMap<>();  
    for (Map.Entry<String, Long> cacheNameAndTimeout : cacheProperties.getTtl().entrySet()) {  
        cacheConfigurations  
                .put(cacheNameAndTimeout.getKey(), redisCacheDefaultConfiguration().entryTtl(  
                        Duration.ofSeconds(cacheNameAndTimeout.getValue())));  
    }  
    return cacheConfigurations;  
}
```

  

//cacheProperties.yml

```yml
cache:  
  ttl:  
    CacheName: 10 #만료 시간
```

- 외부 설정(`CacheProperties`)에서 캐시별 TTL 정보를 읽어와 각 캐시의 만료 시간을 지정  
    이를 통해 특정 캐시만 별도의 만료 정책 등을 적용할 수 있다.
- _entryTtl_ : 기본 만료시간 설정

---

## 

**RedisCacheManager**

Spring의 캐시 추상화에서 Redis를 캐시 저장소로 사용하기 위한 캐시 매니저를 생성  
위에서 설정한 `redisCacheDefaultConfiguration`과 `cacheConfigurations`(커스텀) 이 삽입된다.

```java
@Bean
public CacheManager redisCacheManager(RedisConnectionFactory redisConnectionFactory) {
    return RedisCacheManager.RedisCacheManagerBuilder
            .fromConnectionFactory(redisConnectionFactory)
            .cacheDefaults(redisCacheDefaultConfiguration())
            .withInitialCacheConfigurations(redisCacheConfigurationMap())
            .build();
}
```

- `withInitialCacheConfigurations` : _RedisCacheManager_를 생성할 때 미리 정의된 특정 캐시 이름에 대해 개별적인 설정을 적용할 수 있도록 해주는 메서드입니다.

---

# 3. Service Layer

_Bean 주입_

```java
@Service  
public class TestServiceImpl {

	private final CacheManager cacheManager;  
	private final RedisTemplate<String, Object> redisTemplate;

	public TestServiceImpl(CacheManager cacheManager, RedisTemplate<String, Object> redisTemplate) {  
	    this.cacheManager = cacheManager;  
	    this.redisTemplate = redisTemplate;  
	}
}
```

(생성자 주입)

- 위(_RedisConfiguration_)에서 생성한 CacheManager 및 RedisTemplate의 빈을 주입한다.

주의

만약, Bean으로 생성된 CacheManager객체나, RedisTemplate객체가 여러개라면,  
@Qualifier 어노테이션으로 Bean이름을 명시해야한다.

```
`ex) @Qualifier("CustomCacheManager") CacheManager cacheManager ...`
```

---

_Cache 삽입 / 꺼내기 / 삭제_

```java
	Cache privateKeyCache = cacheManager.getCache("CacheName");
	
	public void putCache() {
		
		if (privateKeyCache != null) {  
		    privateKeyCache.put(keyId, 벨류);  
		} else {  
		    // 캐시가 없으면 예외 처리 또는 로깅  
		    throw new IllegalStateException("privateKeyCache 가 유요하지 않습니다.");  
		}
	}

	public void getCache() {
		
		if (privateKeyCache == null) {  
			throw new IllegalStateException("rsaPrivateKeyCache 가 유요하지 않습니다.");  
		}  
		String privateKeyValue = privateKeyCache.get(keyId, String.class);  
		// 1회용 사용을 위해 조회 후 캐시에서 제거할 수 있다.
		rsaPrivateKeyCache.evict(keyId); // 1회용 사용: 캐시에서 제거
		
	}
```

- getCache.(_CacheName_)으로 캐쉬를 객체를 가져온다.
- `put(keyId, 벨류);` / `get(keyId, String.class);` 로 삽입 / 가져오기가 가능하다.
- `.evict(keyId)`로 삭제 ( 1회성 사용이 가능하다. )

1회성으로 사용하는 이유

나의 경우에 RSA키를 매번 발급 받기 때문에 값을 꺼냄과 동시에 해당 키벨류를 삭제한다.  
Exception이 터지더라도, cacheProperties 에 설정한 TTL이 초과되면 삭제된다.

---

Redis서버를 사용한 Key 관리로, 멀티 서버 환경에서 정합성과 안정성을 챙길 수 있었다.

+ TTL 체크

![Pasted image 20250328120415.png](https://lts.kr/%EC%82%AC%EC%A7%84-%EB%B0%8F-%EB%AC%B8%EC%84%9C/pasted-image-20250328120415.png)

# + TTL 설정

```c
application.yml / properties
      ↓
CacheProperties (ttl map 관리)
      ↓
redisCacheConfigurationMap() → 캐시별 TTL 매핑
      ↓
redisCacheDefaultConfiguration() → 기본 설정 (ex: serializer, 기본 TTL)
      ↓
redisCacheManager() → 최종 CacheManager 생성
```



출처 : https://lts.kr/%F0%9F%8F%A0-taesung's-blog.html

**협업하고 개발일지 정리해준 태성님께 감사드립니다**