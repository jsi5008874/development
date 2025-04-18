
## DB Connection 이란?
애플리케이션과 데이터베이스간의 통신채널을 의미

애플리케이션 서버와 데이터베이스 서버 간의 TCP/IP 기반 네트워크 연결을 의미한다.
TCP 연결을 통해 SQL 쿼리와 그 결과를 주고 받는다.

**하지만 단순히 두 서버의 연결을 뜻하는게 아니고 인증 정보, 세션 상태, 데이터베이스 프로토콜 레이어 등**
**다양한 정보들이 추가된 보다 복잡한 개념**

### DB Connection 연결 과정
1. **연결 수립**: 애플리케이션은 데이터베이스 서버에 연결을 요청합니다. 이때 주소, 포트, 사용자 이름, 비밀번호 등의 인증 정보가 필요
2. **인증 과정**: 데이터베이스 서버는 제공된 정보를 검증하고, 유효하면 연결을 허용
3. **세션 생성**: 연결이 성공하면 서버는 클라이언트를 위한 세션을 생성하고 리소스를 할당
4. **통신**: 이 연결을 통해 SQL 쿼리를 보내고 결과를 받을 수 있습니다.
5. **연결 종료**: 작업이 완료되면 연결을 종료하여 리소스를 해제

### DB Connection을 여러 개 연결할 수 있는 이유
DB 서버에서 DB 어플리케이션에 할당된 포트는 한 개(ex. mysql 3306)인데 어떻게 여러 개의 커넥션을 연결할까?

애플리케이션 서버와 데이터 베이스 서버는 TCP/IP 구조로 연결되기 때문에 소켓통신을 한다.
소켓의 고유한 조합(클라이언트 IP + 클라이언트 포트 + 서버 IP + 서버 포트)을 통해 각 연결을 구분한다.

애플리케이션 서버에서 두 개의 커넥션 요청을 보내더라도 클라이언트 포트가 두 커넥션마다 다르기 때문에
데이터 베이스 서버에서는 이를 구분하여 인식한다.

ex)
- 연결 1: 클라이언트A(192.168.1.10:58432) → DB 서버(10.0.0.1:3306)
- 연결 2: 클라이언트A(192.168.1.10:58433) → DB 서버(10.0.0.1:3306)

**이처럼 클라이언트(애플리케이션 서버)의 포트 번호가 연결마다 달라서 DB 서버에서는 이를 기준으로**
**커넥션을 구분하고 커넥션 별 스레드를 생성해서 요청을 처리하는 방식이다.**


## DB Connection의 세션 관리
클라이언트와 데이터베이스 서버 간의 상태를 유지하고 관리하는 메커니즘

### 세션 생성 및 초기화
1. **세션 생성**: 클라이언트가 데이터베이스에 연결하면 서버는 새로운 세션을 생성
2. **인증 및 권한 부여**: 사용자 인증 후, 서버는 해당 사용자의 권한과 역할을 세션에 할당
3. **세션 ID 할당**: 각 세션에는 고유한 식별자가 부여
4. **환경 설정**: 기본 스키마, 문자셋, 타임존, 트랜잭션 격리 수준 등의 초기 세션 변수가 설정

### 세션 상태 유지
1. **트랜잭션 상태**: 현재 트랜잭션의 시작, 커밋 또는 롤백 상태
2. **임시 객체**:
    - 임시 테이블 (세션 종료 시 자동 삭제)
    - 커서 (쿼리 결과셋을 탐색하기 위한)
    - 준비된 문장(Prepared Statements)
3. **사용자 변수**: 세션 내에서 정의된 변수들
4. **락(Lock)**: 세션이 보유한 테이블이나 레코드 락
5. **캐시**: 쿼리 실행 계획이나 결과 캐시
6. **컨텍스트 정보**: 마지막 실행 쿼리, 영향받은 행 수 등
### 세션 종료
1.  커밋되지 않은 트랜잭션은 롤백
2. 임시 테이블과 객체가 삭제
3. 해당 세션이 보유한 락이 해제
4. 세션 자원이 해제


**커넥션 풀에서는 생성, 종료가 아닌 대여(borrow), 반납(return)의 개념이다.**
**이렇게 세션은 한 요청에 대해 상태를 유지하고 관리하는 수단이다.**


### Spring에서의 DB Connection
Spring에서는 Connection 객체를 통해 DB Connection을 유지한다.
DB 서버와 연결된 Connection 객체를 DBCP에 보관하면서 연결을 유지하는 방식이다.

```java
// DataSource는 일반적으로 애플리케이션 시작 시 구성되어 주입됨
public class UserRepository {
    private final DataSource dataSource;
    
    public UserRepository(DataSource dataSource) {
        this.dataSource = dataSource; // HikariCP 등으로 구성된 DataSource 주입
    }
    
    public User findUserById(Long userId) {
        // SQL 쿼리 정의
        String sql = "SELECT id, name, email FROM users WHERE id = ?";
        
        // try-with-resources를 사용해 자동으로 자원 해제
        try (
            // 1. 커넥션 풀에서 커넥션 객체 획득 (스레드에 할당)
            Connection connection = dataSource.getConnection();
            
            // 2. PreparedStatement 생성
            PreparedStatement pstmt = connection.prepareStatement(sql)
        ) {
            // 3. 파라미터 설정
            pstmt.setLong(1, userId);
            
            // 4. 쿼리 실행 및 결과 처리
            try (ResultSet rs = pstmt.executeQuery()) {
                if (rs.next()) {
                    User user = new User();
                    user.setId(rs.getLong("id"));
                    user.setName(rs.getString("name"));
                    user.setEmail(rs.getString("email"));
                    return user;
                }
                return null;
            }
        } catch (SQLException e) {
            // 예외 처리
            throw new RuntimeException("데이터베이스 조회 중 오류 발생", e);
        }
        // 5. try-with-resources 블록이 종료되면 자동으로 connection.close() 호출
        // 실제로는 커넥션이 종료되지 않고 풀에 반환됨
    }
}
```
이처럼 요청을 처리중인 애플리케이션 서버의 스레드가 커넥션 풀에서 커넥션을 할당받아 사용하고
사용 후 커넥션 풀에 반환하는 방식

## DB Connection 정리
DB Connection은 애플리케이션 서버와 DB 서버를 연결한 객체이다.
TCP/IP 기반으로 서로 연결하며 쿼리 실행 및 결과 반납 뿐만 아니라 사용자 인증 및 권한 부여, 세션관리 등 폭 넓은 기능을 수행한다.

Spring에서는 Connection 객체로 구현하고 커넥션 풀에서 이를 재활용한다.