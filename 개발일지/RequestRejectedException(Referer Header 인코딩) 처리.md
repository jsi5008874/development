![[Pasted image 20241220105112.png]]

# 분석

최근 해당 에러가 지속적으로 발생해서 에러를 해결하기 위해 분석을 했다.

우선 RequestRejectedException은 Spring Security의 StrictHttpFireWall에서 발생하는
예외로 요청에 올바르지 않은 문자열이 포함될 경우 생기는 예외이다.

에러 로그를 보면 인코딩이 제대로 되지 않은 부분이 보인다.
해당 부분이 문제가 되어 예외가 발생했다고 생각했다.

그런데 자세히 보니 header에 url 형식으로 들어가는 속성이 있었나? 생각이 들었고
찾아보니 request header에 referer 라는 속성이 있고
referer는 해당 요청을 보낸 URL을 알려주는 속성이었다.
ex) 네이버에서 우리 회사를 검색하고 검색 결과창에서 우리 회사 링크를 클릭하면
해당 요청에는 referer: naver.com/search_keyword=헬로펀딩
이런 형태로 referer 속성이 지정되어서 보내지는 것이다.

그런데 referer header 어떤 부분에서 인코딩이 잘못되었는지 몰랐다.
해당 부분을 찾아보니 spring에서는 기본적으로 ISO-8859-1로 인코딩을 한다.
그런데 referer header에 한글이 들어가 있었고 한글은 UTF-8로 인코딩을 하므로
서로 인코딩 타입이 달라 글자가 깨진 것이다.

서버 설정이나 tomcat 설정에서 인코딩 타입을 UTF-8로 바꿔서 처리를 할 수 있지만
전체 설정을 바꿔버리면 서버에 어떤 영향이 생길지 모르니 다른 방법을 찾아보자는
의견이 있어서 다른 방법을 찾아보았다.

내가 생각한 방법은 Spring Security에서 발생하는 예외이므로
Spring Security보다 더 우선 순위가 높은 Servlet Filter를 생성하고
해당 Filter에서 Referer Header를 UTF-8로 디코딩하는 것이다.
그러면 요청이 Spring Security에 도달하기 전에 올바르게 디코딩된 문자열을 가지게되어
예외가 발생하지 않을 것이다.


# Filter 클래스 생성

![[Pasted image 20241220120710.png]]
1. ServletRequest를 HttpServletRequest 객체로 변환
	getHeader, getCookie등의 메서드는 HttpServletRequest 객체에서 지원해서 변환한다.
2. String referer에 Referer header 값을 할당
3. referer가 null이 아니거나 %를 포함할 경우 UTF-8로 디코딩
	 %를 포함할 경우가 있는 이유 : 브라우저에서 서버로 요청을 전송할 때 UTF-8로
	 인코딩 해야 하는 문자열이 있다면(ex.한글) 브라우저에서 URL 인코딩(%인코딩)을
	 해서 서버로 요청을 전송하기 때문이다.
	 즉 요청에 한글이 포함되어 있다면 항상 %라는 문자열을 가지고 있기 때문에
	 referer에 %가 포함되어 있다면 referer를 UTF-8로 디코딩해주는 것이다.
4. 디코딩 후 chain.doFilter 메서드로 필터체인의 다음 단계로 요청 진행


# HttpServletRequestWrapper 클래스 생성

![[Pasted image 20241220121849.png]]
wrapper 클래스를 만드는 이유는 HttpServletRequest 객체가 불변객체이므로
Filter 클래스에서 referer header를 UTF-8로 디코딩한다 해도 referer header에 해당 값을
할당할 수 없다.
따라서 Wrapper 클래스를 만들어서 디코딩 된 referer header를 주입하여 처리한다.

실제로 HttpServletRequest 객체는 기존의 referer header를 가지고 있는 상태지만
그걸 감싸고 있는 wrapper 클래스가 변경된 referer header를 가지고 요청에 대한 처리를 할 때 referer 값을 대신 내어준다.

# Filter Config 클래스 생성

![[Pasted image 20241220122502.png]]
Filter를 설정하는 클래스로 servlet Filter에 어떤 필터 클래스를 넣을 것인지
우선순위는 어떻게 지정할 것인지 정하는 configuration 클래스이다.

setFilter를 통해 위에서 만든 RefererDecodingFilter를 servlet 필터로 지정하고

serOrder를 통해 해당 필터의 우선순위를 지정한다.
Integer.Min_VALUE(-2,147,483,648)로 선언해서 가장 높은 우선순위로 지정
이렇게 되면 Servlet Filter 중 가장 먼저 Filter 역할을 하게된다.



getHeader가 ISO-8859-1로 디코딩을 해주는건 맞는데 URL인코딩이 된 Referer라면 ISO-8859-1로 디코딩해도 Referer가 가지고 있던 문자열을 그대로 출력한다 이유는 URL 인코딩에 포함된 문자열들은 모두 ISO-8859-1에서 지원하는 문자열들이기 때문에 깨지는 문자열 없이 본래 가지고 있던 문자열을 그대로 출력한다 하지만 한글로 온 경우에는 ISO-8859-1에서 지원하지 않는 문자열들이어서 getHeader로 불러온다면 문자열이 깨진다