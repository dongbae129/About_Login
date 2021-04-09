# About_Login
-목차-
1.쿠키
2.세션
3.passport(node에서 진행)

로그인으로 바로 넘어가기 전에 http와 로그인의 관계 및 왜 쿠키, 세션이 필요한지 알아보고 넘어가겠습니다.

웹은 클라이언트 <-> 서버 / request <-> response 로 이루어집니다. 그리고 이런 동작은 http라는 프로토콜을 이용하여 작동합니다.
그리고 http는 stateless라는 특징을 가지고 있는데 이는 request,response 할일 다하고 연결을 끊어버리는걸 의미합니다.

그렇다면 로그인 문제와 로그인 후 동작하는 세션들(회원정보 가져오는 것들)에 문제가 생깁니다. 이를 해결하고자 쿠키와 세션이 등장합니다.

-쿠키-
서버를 대신해서 각종 정보들을 웹 브라우저(사용자의 컴퓨터)에 저장합니다. request를 보낼때 header에 동봉하여 보내서 서버가 사용자를 식별할수 있도록 합니다.
인증 과정중 최초 request를 통해 서버가 client에게 cookie를 발급하며 다음 request부터 client가 server에게 cookie와 같이 request를 합니다.
쿠키는 key: value 형식으로 저장되며 유효기간을 설정할수도 있습니다. 유효기간이 설정되지 않았다면 웹 브라우저가 종료될때 사라지게 됩니다.
단점 : 매 request마다 header에 추가하기 때문에 트래픽 추가 발생하며 client쪽에 저장되기 때문에 보안적 문제점이 있습니다.
그렇기 때문에 쿠키는 세션관리, 개인화(광고나 장바구니 등등)를 위하여 사용됩니다.

-세션-

