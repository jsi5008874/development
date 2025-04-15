  

### **POODLE (Padding Oracle On Downgraded Legacy Encryption)**

공격자가 SSL 3.0의 CBC 모드 패딩 방식을 악용하여 **암호문을 해독하는 공격**

서버가 SSL 3.0을 지원하면 TLS를 사용하더라도 SSL 3.0으로 다운드레이드하여 공격 가능

(SSL 3.0 완전 차단)

  

### **BEAST 공격 (SSL/TLS 1.0 취약점)**

TLS 1.0 이하 버전의 CBC 모드 암호화 방식의 약점을 이용

특정한 방식으로 세션 쿠키 등의 중요한 데이터를 복호화 가능

  

### **DROWN (Decrypting RSA with Obsolete and Weakened eNcryption)**

SSL 2.0의 취약점을 이용해 TLS의 RSA 암호화를 해독하는 공격

  

### **Heartbleed (하트블리드, OpenSSL 취약점)**

OpenSSL의 **Heartbeat 기능**을 악용하여 **메모리 정보를 탈취**하는 공격

비밀번호, 세션 쿠키, 개인키 등의 **중요한 데이터 유출 위험**

  

### **MITM (Man-in-the-Middle, 중간자 공격)**

클라이언트와 서버 사이에서 데이터를 가로채고 변조하는 공격

SSL/TLS의 **핸드셰이크 과정에서 취약점이 있을 경우** 공격 가능