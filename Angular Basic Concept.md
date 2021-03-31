# Angular Basic Concept

https://angular.kr/guide/architecture



[Module](#module)

[Component](#component)

[Template, Directive, Binding](#template,-directive,-binding)

[Service, Dependency Injection (DI)](#service,-dependency-injection-(di))

[Routing](#routing)



## Module

보통 AppModule라는 최상위 모듈이 존재. 여기서 부트스트랩 방법 설정

NgModule은 기능적으로 관련된 컴포넌트를 묶어서 선언한다. 컴포넌트 외에 서비스, 폼 기능을 포함하기도 한다.

필요할 때 모듈을 불러오는 lazy loading 활용가능



## Component

모든 컴포넌트는 *클래스* 와 *템플릿* 으로 구성되어있다.

클래스는 데이터와 로직을 처리, 템플릿은 HTML정의

@Component() 데코레이터를 사용해서 컴포넌트에 대한 메타테이더, 템플릿 지정



## Template, Directive, Binding

템플릿은 HTML과 Angular 마크업 문법을 조합해서 구성.

디렉티브를 이용해서 템플릿 동작 확장 (ex ngFor)

바인딩 마크업으로 데이터를 DOM과 연결

- Event binding 
- Property binding

Angular는 뷰가 화면에 ㅍ시되기 전에 템플릿에 사용된 디렉티브와 바인딩 문법을 모두 체크해서 HTML element 와 DOM을 변형한다.



## Service, Dependency Injection (DI)

어떤 데이터나 함수가 하나의 뷰에만 적용되지 않는 경우, 서비스 클래스로 활용하는 것이 좋다.

@Injectable 데코레이터로 정의함. 이 데코레이터를 쓰면 다른 컴포넌트나 서비스에서 의존성으로 주입하기 위해 먼저 처리됨.



## Routing

페이지 전환(애플리케이션 상태 변화)기능을 제공함. 기본적으로 브라우저의 페이지 전환 방식을 바탕으로 구현되어있다.

- 주소표시줄 URL 매핑
- 링크
- 브라우저 히스토리에 따른 뒤로/앞으로 가기

Angular 라우터는 페이지 대신 뷰를 URL과 매핑한다. 링크를 클릭한 경우 브라우저는 새로운 페이지로 전환을 시도하는데, 라우터가 이를 중지시키고 페이지 이동 없이 뷰만 전환한다.

아직 로드되지 않은 모듈로 전환을 시도하면, lazy loading을 사용해 모듈을 불러오고 난 후 뷰를 전환.

라우터와 브라우저는 히스토리를 공유한다.

