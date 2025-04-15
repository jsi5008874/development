**연동 Flow**
1. 대출계약완료 후 대출 실행 전에 플랫폼 수수료를 먼저 납부
2. 카드결제 버튼 클릭 시 카드결제 화면으로 넘어감
3. 이 때 결제창은 iframe으로 페이스토리에서 만든 결제창으로 넘어가게 됨
4. 카드사 선택 > 카드 인증 > 결제하기를 누르면 인증(1단계)요청을 함
5. 인증결과를 받아서 app프로젝트의 PayStoryController > getPayResult 메서드로 넘겨줌
6. 해당 메서드에서 넘어온 인증결과 데이터들을 PayStoryService로 넘겨줌
7. PayStroyService에서는 인증결과 데이터를 토대로 승인요청 데이터를 작성
8. 승인요청 데이터를 api 프로젝트로 보냄
9. api프로젝트에서 페이스토리로 승인요청 데이터를 보내고 응답 데이터를 받아옴
10. 승인응답 데이터를 app프로젝트로 재송신
11. app 프로젝트에서 데이터를 받으면 controller에서 승인 성공이면 payResult.html로
12. 승인 실패면 PayFail.html로 보내서 뷰를 출력
13. payAuthFail은 인증(1단계)실패 시 넘어가는 페이지


내가 맡은 파트는 6~13번 

Controller
1. Controller에서 인증결과 데이터를 받아온 후 유효성 검사 시행
2. resultCode이 0000이면 성공이므로 request 데이터를 service로 보낸다.
3. service에서 API를 통해 응답을 받으면 Controller로 ResponseModel 형태로 응답을 줌
4. payApprovalResponseDto에 응답 데이터가 담기면 유효성 검사
5. 성공 시 응답 데이터를 payResult 페이지로
6. 실패 시 응답 데이터를 payFail 페이지로


![[Pasted image 20241121180559.png]]

Serive
1. service에서는 받은 데이터를 api 연동 규격서에 맞춰 DTO에 세팅
2. 이후에 apiService.sendPayStroyApprooval 메서드를 통해 api 프로젝트로 내부 통신
3. api 프로젝트에서 payStory와 외부 통신 후 ApiResponseModel 형태로 응답을 줌
4. model 객체에 응답이 담기면 성공, 실패 여부 유효성 검사를 통해 로직 처리
     성공 시 : 할부 개월 수 문구 추가, 승인 성공 실패 시 : 승인 실패 문구 추가
![[Pasted image 20241121180923.png]]
