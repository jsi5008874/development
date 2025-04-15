PETI(공직자윤리위원회)에서 공직자의 재산수준을 파악하기위해 각 금융기관으로 조회 대상자의 대출, 예치금 보유현황 요청을 보냄

![[Pasted image 20241122094652.png]]
양식은 이렇게 SAM 형식으로 보내줌
순번|주민번호|이름|조회기준일자|요청지역|담당자|담당자번호

자동화 전의 프로세스는(쿼리는 너무 길어서 생략, 각 step마다 쿼리가 있음)
step1 sam 파일에 있는 조회 대상자가 우리 회원인지 DB에서 쿼리로 조회
       > 조회결과 회원이 아닐 시 조회결과 없음으로 보고

step2 조회결과 우리 회원이면 조회기준일자와 회원의 user_no를 기준으로 해당 회원의
대출계좌, 예치금 계좌를 조회해서 대출 또는 예치금, 투자금이 얼마인지 쿼리로 조회

step3 step2 조회 결과를 PETI_VBAL 테이블에 insert

step4 PETI에서 받은 SAM파일을 PETI_BALANCE 테이블에 insert

step5 쿼리를 통해 조회하고 나온 데이터를 회신용 SAM FILE로 변환하여 PETI에 보고


**이렇게 다섯 번의 쿼리를 통해 데이터를 추출하고 다시 insert하고 하는 과정을 반복했음**


해당 프로세스를 자동화 시키기 위해 전체적인 로직은

1. 텍스트파일 서밋으로 서버로 전송(PETI에서 보내준 SAM 파일)
2. 받은 텍스트파일 정규표현식(연속된 13자리숫자)으로 주민번호만 추출
3. 추출한 데이터로 step1 조회
4. 조회 결과 없으면 조회결과 없음 출력(해당 건 종료)
5. 조회결과 있으면 step2부터 쿼리 실행
6. step2 결과를 PETI_VBAL 테이블에 insert(step3)
7. 1번에서 받은 파일의 내용을 파싱해서 PETI_BALANCE 테이블에 insert(step4)
8. step5 쿼리로 조회된 데이터를 데이터 회신용 SAM 양식에 맞게 변환하여 txt 파일로 사용자가 다운로드



1. 텍스트파일 서밋으로 서버로 전송(PETI에서 보내준 SAM 파일)
![[Pasted image 20241122114005.png]]
파일선택 버튼으로 업로드 할 파일을 선택, 검사 버튼 클릭 시 파일이 서버로 넘어가서
로직을 처리

![[Pasted image 20241122115426.png]]

버튼 클릭 시 ajax로 파일 전송

2. 받은 텍스트파일 정규표현식(연속된 13자리숫자)으로 주민번호만 추출
![[Pasted image 20241122120120.png]]
controller에서 받은 file을 service로 넘겨준다.

![[Pasted image 20241122122320.png]]
SAM 파일은 BufferedReader와 Patter, Matcher로 주민번호, 조회기준일만 추출해서
petiInfo(Map<String, String> 형태)에 삽입 후 return

3. 추출한 데이터로 step1 조회
![[Pasted image 20241122122929.png]]
petiInfo의 키 값(주민번호)만 keySet으로 step1 조회
    >>주민번호로 회사의 고객이면 user_no와 주민번호를 fileMap에 삽입


4. 조회 결과 없으면 조회결과 없음 출력(해당 건 종료)


5. 조회결과 있으면 step2부터 쿼리 실행
![[Pasted image 20241126165526.png]]
step2는 우리 회사의 회원이면서 PETI에서 확인 요청한 공직자 및 공직자 가족인 사람을
unionList에 담아서 해당 회원이 우리 회사에서 대출 또는 투자한 현황을 조회한다.
여기서 필요한 데이터는 조회기준일과 user_no인데
user_no는 fileMap에 있고 조회기준일은 petiInfo에 있음

![[Pasted image 20241126165842.png]]
petiInfo, fileMap은 동일하게 주민번호를 key로 가지고 있기 때문에
주민번호를 기준으로 같은 주민번호를 가진 데이터끼리 unionList에 담아서 DB 조회 시 활용한다.


6. step2 결과를 PETI_VBAL 테이블에 insert(step3)
![[Pasted image 20241126171133.png]]
step2 결과를 기준으로 insert

7. 1번에서 받은 파일의 내용을 파싱해서 PETI_BALANCE 테이블에 insert(step4)
![[Pasted image 20241126171202.png]]
최초에 받은 SAM파일을 파싱 후 데이터를 List에 담아서 insert

8. step5 쿼리로 조회된 데이터를 데이터 회신용 SAM 양식에 맞게 변환하여 txt 파일로 사용자가 다운로드
![[Pasted image 20241126171546.png]]
step5 쿼리로 추출한 finalResult를 txt 형태로 변환

![[Pasted image 20241126171636.png]]
변환 후 controller에서 뷰로 전송해서 사용자가 다운로드