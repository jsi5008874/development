  

Spring MVC에서 특정 컨트롤러나 전역적으로 발생할 수 있는 예외를 처리하기 위해

사용하는 어노테이션

  

1. **컨트롤러 내부에서 사용**
    
    특정 컨트롤러의 예외를 처리할 때 사용합니다.
    
    ```Java
    java
    코드 복사
    @Controller
    public class SampleController {
    
        @GetMapping("/sample")
        public String sampleMethod() {
    // 일부 로직에서 NullPointerException 발생 가능throw new NullPointerException("테스트 예외 발생");
        }
    
        @ExceptionHandler(NullPointerException.class)
        public ResponseEntity<String> handleNullPointerException(NullPointerException e) {
            return new ResponseEntity<>("NullPointerException 처리: " + e.getMessage(), HttpStatus.BAD_REQUEST);
        }
    }
    
    ```
    
2. **전역 예외 처리**
    
    모든 컨트롤러에서 발생하는 예외를 처리하려면 `**@ControllerAdvice**`와 함께 사용합니다.
    
    ```Java
    java
    코드 복사
    @ControllerAdvice
    public class GlobalExceptionHandler {
    
        @ExceptionHandler(IllegalArgumentException.class)
        public ResponseEntity<String> handleIllegalArgumentException(IllegalArgumentException e) {
            return new ResponseEntity<>("IllegalArgumentException 처리: " + e.getMessage(), HttpStatus.BAD_REQUEST);
        }
    
        @ExceptionHandler(Exception.class)
        public ResponseEntity<String> handleGenericException(Exception e) {
            return new ResponseEntity<>("기타 예외 처리: " + e.getMessage(), HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }
    ```
    

@ControllerAdvice와 함께 사용하면 전역적으로 적용된다.

  

@ExceptionHandler의 장점

1. 예외 처리 로직을 중앙 집중화하여 코드의 유지보수성을 향상시킴.
2. 공통 예외 처리 로직을 재사용 가능.
3. 예외에 따라 사용자 친화적인 메시지 응답 가능.