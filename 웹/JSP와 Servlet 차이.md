# 🎡 JSP와 Servlet 차이

## Servlet이란
- 웹 기반의 요청에 대한 동적인 처리가 가능한 Server Side에서 돌아가는 Java Program
- Java 코드 안에 HTML 코드 (하나의 클래스)
- 웹 개발을 위해 만든 표준

## JSP란
- Java 언어를 기반으로 하는 Server Side 스크립트 언어
- HTML 코드 안에 Java 코드
- Servlet를 보완하고 기술을 확장한 스크립트 방식 표준
- Servlet의 모든 기능 + 추가적인 기능
  
---
## Servlet
- Java 코드 안에 HTML 코드 (하나의 클래스)
- data processing(Controller)에 좋다.
- 즉 DB와의 통신, Business Logic 호출, 데이터를 읽고 확인하는 작업 등에 유용하다.
- Servlet이 수정된 경우 Java 코드를 컴파일(.class 파일 생성)한 후 동적인 페이지를 처리하기 때문에 전체 코드를 업데이트하고 다시 컴파일한 후 재배포하는 작업이 필요하다. (개발 생산성 저하)  

## JSP
- HTML 코드 안에 Java 코드
- presentation(View)에 좋다.
- 즉 요청 결과를 나타내는 HTML 작성하는데 유용하다.
- JSP가 수정된 경우 재배포할 필요가 없이 WAS가 알아서 처리한다. (쉬운 배포)
