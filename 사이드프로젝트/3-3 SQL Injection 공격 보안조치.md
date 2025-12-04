## 개요
매칭이 되지 않아 redis의 매칭 대기열을 확인해보았다.
```
  1) "matching_queue:1cGLAvQhp'; waitfor delay '0:0:15' -- "
  2) "matching_queue:@@YU7zK"
  3) "matching_queue:1-1; waitfor delay '0:0:15' -- "
  4) "matching_queue:1W98xpzzq')) OR 945=(SELECT 945 FROM PG_SLEEP(15))--"
  5) "matching_queue:-1\" OR 2+54-54-1=0+0+0+1 -- "
```
매칭큐 자료구조에 sql injection 공격 패턴으로 보이는 내용들이 확인됐다.

아직 개발중이고 우리 시스템의 특성을 고려했을 때 개발이 완료되더라도 redis나 mysql에 털어갈 데이터가 없기 때문에 보안에 별 신경을 안쓴것은 사실이다.
하지만 그대로 둔다면 서비스 장애가 계속해서 발생할 것이기 때문에 보안 조치를 해야하며
단순히 "털릴게 없는데 굳이 해야하나?" 이렇게 안일하게 생각한 점을 반성하고 보안 공부도 할 겸 보안 조치를 시작했다.

또한 모니터링도 보완해야한다.
이상 징후가 발생하면 알림으로 바로 인지할 수 있어야 하는데 현실은 매칭을 잡아보다가 매칭이 안되서 알게된 사실이었다.
로깅과 모니터링을 보완해서 즉각 조치가 가능하도록 변경해야겠다.
### 침투 경로(예상)
burf suite나 SQLMap 같은 도구로 페이로드를 조작하여 보낸 것 같다.
```java
// MatchingSeviceImpl 일부
public QueueResponse enqueueUser(QueueRequest request, String ip, HttpSession session) {  
    String category = request.getCategory();  
    String queueKey = "matching_queue:" + category;  
    String sessionId = session.getId();  
  
    try {  
        ensureQueueExists(queueKey);  
  
        if(Boolean.TRUE.equals(redisTemplate.hasKey(sessionId))){  
            throw new IllegalStateException("이미 매칭 대기중입니다.");  
        }  
  
        Long queueSize = queueRedisTemplate.opsForList().size(queueKey);  
        queueRedisTemplate.opsForList().rightPush(queueKey, sessionId);
```
매칭큐에 삽입하는 서비스 로직을 보면 request > category > queueKey = "matching_queue:" + category로 가공 후 
queueKey 매칭큐 명, sessionId를 데이터로 매칭큐에 저장한다.

공격자가 request의 값을 악의적으로 변경하여 요청을 보냈고 검증 없이 그대로 redis 매칭큐에 삽입이 된 것이다.
```
ex)
POST /api/matching/request
 { "category": 1cGLAvQhp'; waitfor delay '0:0:15' -- "}
```
예시와 같이 요청 페이로드의 데이터를 수동으로 바꿔서 공격을 시도


### 조치 내용
1. 프론트엔드 검증
	허용된 값만 전송
2. nginx 필터링
	SQL Injection 패턴 차단
3. 백엔드 검증
	validation 추가
4. 로깅 및 모니터링 강화
	공격 시도 기록, 알림 발송

## 조치

### 프론트 엔드 검증 추가
해킹 시도에서는 크게 의미가 없지만 프론트 엔드에도 검증 단계를 추가하기로 했다.
(burf suite 같은 툴로 요청을 변경하면 쉽게 우회가 가능하지만 사용자 실수로 요청이 잘못 전송되는 경우를 방지하기 위해 추가)

현재는 매칭 요청 시 요청에 들어가는 카테고리를 검증하지 않고 그대로 요청을 보내고 있다.
그래서 요청을 보낼 때 카테고리 타입을 검증하고 요청을 보내는 방식으로 변경했다.

#### Category 타입 정의
```node
// utils/categiries.ts
* 카테고리 인터페이스
export interface Category {
id: CategoryId;
name: string;
icon: string;
displayName: string;
}

* 전체 카테고리 목록 (UI 표시용)
export const CATEGORIES: Category[] = [
{ id: 'sports', name: '스포츠', icon: '🏈', displayName: '스포츠' },
{ id: 'game', name: '게임', icon: '🎮', displayName: '게임' },
{ id: 'travel', name: '여행', icon: '✈️', displayName: '여행' },
{ id: 'music', name: '음악/영화', icon: '🎵', displayName: '음악/영화' },
{ id: 'hobby', name: '일상/취미', icon: '🎨', displayName: '일상/취미' },
{ id: 'love', name: '연애/썸', icon: '💕', displayName: '연애/썸' },
{ id: 'free', name: '자유주제', icon: '💬', displayName: '자유주제' },
];
```

#### Category 타입 검증
```node
// utils/categiries.ts
* 카테고리 ID 유효성 검증 (Type Guard)
export function isValidCategoryId(value: unknown): value is CategoryId {
if (typeof value !== 'string') {
return false;
}
return ALLOWED_CATEGORY_IDS.includes(value as CategoryId);
}

* 카테고리 객체 유효성 검증
export function isValidCategory(category: unknown): category is Category {
if (!category || typeof category !== 'object') {
return false;
}
const cat = category as Category;
return (
'id' in cat &&
isValidCategoryId(cat.id) &&
typeof cat.name === 'string' &&
typeof cat.icon === 'string' &&
typeof cat.displayName === 'string'
);
}
```

utils/categiries.ts에 카테고리 타입을 정의하고 유효성을 검증하는 함수를 정의했다.

#### 검증 로직 사용
```node
// page.tsx

import {
CATEGORIES,
Category,
isValidCategory,
validateCategory
} from '@/utils/categories'; // categiries.ts import

const handleCategoryClick = (category: Category) => {
try {
validateCategory(category);
setActiveCategory(category);
} catch (error) {
console.error('[Security] Invalid category selection:', error);
setActiveCategory(CATEGORIES[0]);
}
};


const handleStartChat = async () => {
try {
// 카테고리 유효성 재검증
if (!isValidCategory(activeCategory)) {
throw new Error('유효하지 않은 카테고리입니다.');
}
// ...... 기타 코드
```
카테고리 선택시 카테고리 타입을 검증하고 매칭 등록 시 카테고리 타입인지 검증 후 매칭 대기열 등록으로 이어진다.
이젠 지정된 카테고리(sports, game, travel 등) 외에는 요청이 불가하게 변경되었다.

비록 해킹 시도에 큰 의미는 없는 개선이지만 서비스 안정성에 도움이 되기 때문에 추가한 사항이다.

### Nginx 설정 추가
Nginx는 단순한 공격 차단만 설정으로 추가 했다.
Nginx의 특성상 복잡한 검증 로직은 백엔드에서 다뤄야하고 Nginx는 패턴 감지, rate limiting 등 단순한 검증만 실행하도록 설정했다.

#### 패턴 감지 설정
```
// app.conf
# SQL Injection 차단
        if ($request_uri ~* "(union|select|insert|drop|delete|update|--|/\*|#)") {
            return 403;
        }
```
nginx에서 요청 uri를 검사해 sql injection 의심 요청을 차단
쿼리 조작 명령어 패턴이 포함된 요청들을 nginx가 감지 > 403으로 차단

```
// app.conf
# XSS 차단
        if ($request_uri ~* "(<script|javascript:|onerror=|onload=)") {
            return 403;
        }
```
Cross-Site Scripting 공격을 차단
javascript 패턴이 포함된 요청을 감지 > 403으로 차단

**테스트**
```
docker-compose exec -T nginx curl -k -s -o /dev/null -w "DROP TABLE: %{http_code}\n" "https://localhost/api/test?id=DROP%20TABLE"
>> DROP TABLE 명령어가 들어간 uri 요청(Sql Injection)
DROP TABLE: 403
>> 403 응답으로 차단

docker-compose exec -T nginx curl -k -s -o /dev/null -w "<script> tag: %{http_code}\n" "https://localhost/api/search?q=<script>alert(1)</script>"
>> <script>가 들어간 uri 요청(xss 공격)
<script> tag: 403 
>> 403 응답으로 차단
```


#### Rate Limiting 설정
Rate Limiting은 IP를 기준으로 초당 요청 횟수를 제한하는 것으로 nginx에서 설정 가능하다.
보통 해킹은 봇으로 자동화해서 대량의 요청을 짧은 시간에 보내기 때문에 nginx에서 차단이 가능하다.
또한 DDOS 공격을 차단하는데도 유용한 기능이다.

```
// nginx.conf
http {
# 인증 API
limit_req_zone $binary_remote_addr zone=auth_limit:10m rate=3r/s; 
# 매칭/채팅 API
limit_req_zone $binary_remote_addr zone=matching_limit:10m rate=5r/s; 
# 일반 API
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s; 
# 메인 페이지
limit_req_zone $binary_remote_addr zone=general_limit:10m rate=20r/s; 
# 정적 파일
limit_req_zone $binary_remote_addr zone=static_limit:10m rate=50r/s; 
# 동시 연결 제한 
limit_conn_zone $binary_remote_addr zone=conn_limit:10m; 
limit_conn_zone $binary_remote_addr zone=ws_conn_limit:10m;
}
```
http 섹션 바로 아래에 rate limiting 설정이 들어간다.
zone = 설정 이름
rate = 초당 허용 요청 횟수이다.

```
// app.conf

location /api/matching/ {
limit_req zone=matching_limit burst=10 nodelay;
limit_conn conn_limit 5;
... 다른 설정들
}
>> /api/matching으로 들어오는 요청은 matching_limit의 rate=5r/s, burst=10을 더해서
>> 초당 최대 15개의 요청까지 허용

location /api/ {
limit_req zone=api_limit burst=20 nodelay;
limit_conn conn_limit 5;
... 다른 설정들
}
>> /api로 들어오는 요청은 api_limit의 rate=10r/s, burst=20을 더해서
>> 초당 최대 30개의 요청을 허용
```
nginx.conf에서 설정한 rate limit의 값들을 app.conf에서 가져와 사용한다.
location 별로 지정 가능하며 rate외에 burst를 지정해서 최대치를 더 높여서 조정할 수 있다.
burst는 순간적으로 허용해주는 추가 요청량이며 일시적인 트래픽 폭주 시 어느 정도까지 허용해줄지 결정하는 옵션

```
nginx.conf와 app.conf의 차이점

nginx.conf는 nginx 설정의 뼈대이며 실제로 nginx가 바라보는 설정 파일
app.conf는 사용자가 커스터 마이징해서 사용하는 설정으로 대부분 app.conf에서 설정을 완료했다.
(https 설정, location 등 대부분 커스터 마이징한 설정들이 app.conf에 포함)
nginx.conf에서 app.conf를 include해서 두 설정 파일을 합친다.

ex)
include /Anonichat/elk-stack/nginx/conf.d/app.conf
>> nginx.conf에 들어가는 include 문
```

**테스트**
```
동시 요청 shell script 작성
# 임시 파일
tmp_file="/tmp/rate_test_results.txt"
> $tmp_file
> 
# 100개 동시 요청
for i in {1..100}; do
    {
        code=$(docker-compose exec -T nginx curl -k -s -o /dev/null -w "%{http_code}" https://localhost/)
        echo "$code" >> $tmp_file
    } &
done

# 모든 요청 완료 대기
wait

echo "결과 집계:"
total=$(wc -l < $tmp_file)
success=$(grep -c "200" $tmp_file)
limited=$(grep -c "503" $tmp_file)
other=$(grep -v "200\|503" $tmp_file | wc -l)
  
echo "  총 요청: $total"
echo "  성공 (200): $success"
echo "  차단 (503): $limited ✅"
echo "  기타: $other"
echo ""
```
shell script를 실행 시키면 동시 요청을 하고 결과가 나올 것이다.

```
/tmp/test_concurrent.sh

메인 페이지 (rate=20, burst=30 = 50개)
   100개 동시 요청 보내기...
  
결과 집계:
  총 요청: 100
  성공 (200): 85
  차단 (503): 15 ✅
  기타: 0
```
차단에 성공한 것을 볼 수 있다.
기대치는 50개가 차단이 되어야 하지만 1초에 100개를 모두 못보내서 기대치만큼 차단이 되지 않은것 같다.

### 백엔드 검증 로직 추가
백엔드에서는 category Enum 타입을 만들어 정해진 카테고리 외에 다른 입력이 들어오면 차단하도록 설정할 것이다.

#### gradle 의존성 추가
```
dependencies {
// Bean Validation 추가 
implementation 'org.springframework.boot:spring-boot-starter-validation'
}

@NotNull        // null 체크
@NotBlank       // 빈 문자열 체크
@Size           // 길이 체크
@Min, @Max      // 숫자 범위
@Email          // 이메일 형식
@Pattern        // 정규식 검증
```
validation 의존성을 추가해서 @NotNull, @Pattern 등 어노테이션을 사용 가능하도록 세팅한다.

#### Enum 생성
```java
public enum CategoryType {  
    LOVE("연애", "love"),  
    FREE("자유", "free"),  
    GAME("게임", "game"),  
    SPORTS("스포츠", "sports"),  
    MUSIC("음악", "music"),  
    TRAVEL("여행", "travel"),  
    HOBBY("취미", "hobby");  
  
    private final String displayName;  
    private final String key;  
  
    CategoryType(String displayName, String key) {  
        this.displayName = displayName;  
        this.key = key;  
    }  
  
    /**  
     * JSON으로 변환 시 key 값 반환  
     * 예: LOVE -> "love"  
     */    @JsonValue  
    public String getKey() {  
        return this.key;  
    }  
  
    /**  
     * JSON에서 Enum으로 변환  
     */  
    @JsonCreator  
    public static CategoryType from(String value) {  
        if (value == null || value.isBlank()) {  
            throw new IllegalArgumentException("카테고리는 필수입니다");  
        }  
  
        return Arrays.stream(values())  
                .filter(category -> category.key.equalsIgnoreCase(value) ||  
                        category.name().equalsIgnoreCase(value))  
                .findFirst()  
                .orElseThrow(() -> new IllegalArgumentException(  
                        "유효하지 않은 카테고리: " + value  
                ));  
    }  
}
```
1. 고정된 카테고리 명을 Enum으로 선언하여 비교할 때 사용
2. from()으로 요청에 있는 카테고리명과 Enum에서 선언한 카테고리명이 일치하는지 확인하는 과정

즉 고정된 카테고리명을 가지고 요청으로 들어온 카테고리명이 일치하는지 확인하여 일치하지 않는 요청은 차단한다.

#### DTO 수정
```java
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class QueueRequest {
    
    @NotNull(message = "카테고리는 필수입니다")
    private CategoryType category;  // ← String에서 CategoryType으로 변경
    
    private String userIp;
    private Long score;
    
    /**
     * Redis 키 생성용
     * 예: LOVE -> "love"
     */
    public String getCategoryKey() {
        return category.getKey();
    }
}

```
기존에 String 타입이었던 category를 CategoryType(Enum)으로 변경

getCategoryKey() 메서드를 통해 CategoryType(Enum)을 key로 변경한다.

#### Controller 수정
```java
@PostMapping("/queue") 
public ResponseEntity<QueueResponse> matchingStart( @Valid @RequestBody QueueRequest request, // ← @Valid 추가 HttpServletRequest httpRequest, HttpSession session) {
//... 이하 코드는 동일


@DeleteMapping("/cancel") 
public ResponseEntity<DeleteQueueResponse> deleteQueue( @RequestParam("category") String categoryParam, HttpSession session) { 
	try { 
	// String을 Enum으로 변환 (자동 검증) 
	CategoryType category = CategoryType.from(categoryParam);
	// ... 이하 코드는 동일
```
**/queue**
@Valid를 추가하여 유효성 검사를 실행
```java
// QueueRequest DTO
@NotNull(message = "카테고리는 필수입니다")
private CategoryType category;
```
QueueRequest DTO에서 선언했던 @NotNull을 바로 체크
만약 null인 경우에는 예외를 던져서 차단한다.

**/cancle**
CategoryType.from()을 활용하여 카테고리가 Enum에 선언된 카테고리와 일치하는지 확인


**유의사항**
/queue에서 컨트롤러 도달전에 이미 Enum으로 카테고리명을 검증한다.
```
클라이언트 요청
    ↓
{"category": "love"}
    ↓
[1] Spring이 JSON → Object 변환 시도
    ↓
[2] Jackson이 CategoryType.from("love") 호출 ← 검증 여기서!
    ↓
    ├─ 성공 → Controller로 전달
    └─ 실패 → 400 에러 즉시 반환 (Controller 도달 못함)
    ↓
[3] Controller 실행
```
Spring에서 json 형태를 QueueRequest 객체로 변환할 때 CategoryType.from()을 활용해 검증을 실행
통과하지 못하면 차단된다.
#### Service 수정
```java
@Override 
public QueueResponse enqueueUser(QueueRequest request, String ip, HttpSession session) { 
// Enum에서 안전한 키 추출 
String categoryKey = request.getCategoryKey(); // ← "romance", "car" 등

// ... 이하 코드 동일
```
기존에 String 형태의 category를 검증된 Enum 형태로 변경
getCategoryKey() 메서드를 활용해서 Enum의 key 값으로 변환하여 사용

#### 예외 처리 추가
```java
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {

    /**
     * Bean Validation 실패 처리
     * 예: @NotNull, @Pattern 위반
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidationException(
            MethodArgumentNotValidException e) {
        
        Map<String, String> errors = new HashMap<>();
        e.getBindingResult().getAllErrors().forEach(error -> {
            String fieldName = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
            log.warn("Validation 실패 - Field: {}, Message: {}", fieldName, errorMessage);
        });
        
        return ResponseEntity.badRequest().body(errors);
    }

    /**
     * Enum 변환 실패 처리
     * 예: "invalid_category" -> CategoryType 변환 실패
     */
    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<Map<String, String>> handleIllegalArgumentException(
            IllegalArgumentException e) {
        
        log.warn("잘못된 요청: {}", e.getMessage());
        
        Map<String, String> error = new HashMap<>();
        error.put("error", e.getMessage());
        
        return ResponseEntity.badRequest().body(error);
    }

    /**
     * 예상치 못한 오류 처리
     */
    @ExceptionHandler(Exception.class)
    public ResponseEntity<Map<String, String>> handleGeneralException(Exception e) {
        log.error("예상치 못한 오류 발생", e);
        
        Map<String, String> error = new HashMap<>();
        error.put("error", "서버 오류가 발생했습니다");
        
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```
GlobalExceptionHandler는 오류가 발생하면 처리하여 클라이언트로 응답을 보내는 Hadler다.

```
 예외 발생
    ↓
Controller까지 예외 전파
    ↓
DispatcherServlet이 예외 감지
    ↓
@ExceptionHandler 찾기 시작
    ↓
├─ 1순위: Controller 내부의 @ExceptionHandler
└─ 2순위: @ControllerAdvice/@RestControllerAdvice의 @ExceptionHandler ← GlobalExceptionHandler
    ↓
예외 처리 후 응답 반환
    ↓
클라이언트
```
스프링 내부적으로는 위와 같이 동작하며 예외 종류별 어떻게 처리할 것인지 지정해놓은 클래스라고 생각하면 된다.

Enum 검증을 추가하면서 MethodArgumentNotValidException, IllegalArgumentException을 추가했기 때문에
GlobalExceptionHandler에 해당 예외에 대한 처리 방법을 정의했다.

#### 테스트
**정상인 경우**
```
curl -X POST http://localhost:8080/api/matching/queue \
  -H "Content-Type: application/json" \
  -d '{"category": "love"}'
>> love 카테고리로 matching/queue로 요청

{"sessionId":"0978f0ed-bb58-4f4f-bc83-35e5f7fda39c","queueSize":0,"category":"love",
"message":"매칭 대기열에 등록되었습니다.","status":"SUCCESS"}
>> 정상적으로 등록된 것을 볼 수 있다.
```

![[스크린샷 2025-12-04 오후 6.05.23.png]]
redis insight로 확인했을 때 love 매칭큐에 해당 session이 정상적으로 등록된 것을 확인할 수 있다.

**비정상인 경우**
```
curl -X POST http://localhost:8080/api/matching/queue \
  -H "Content-Type: application/json" \
  -d '{"category": "romance'\'' ; waitfor delay '\''0:0:15'\'' --"}'
 >> 비정상적인 카테고리명을 요청

{"error":"서버 오류가 발생했습니다"}
>> 오류 발생
```

![[Pasted image 20251204180842.png]]
에러 로그가 생기는 것을 확인할 수 있다.
