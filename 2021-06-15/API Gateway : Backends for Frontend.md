# API Gateway / Backends for Frontends 개념

[Microservices Pattern: API gateway pattern](https://microservices.io/patterns/apigateway.html)

지금 진행하는 프로젝트에서 Backends for Frontends 패턴을 적용한다고 합니다. 하지만 해당 패턴에 대한 이해도가 부족하여 위 문서를 번역하여 개념을 이해해 보도록 하겠습니다.

## 상황

MSA(https://microservices.io/patterns/microservices.html) 패턴을 적용한 온라인 쇼핑몰에서 제품 상세 페이지를 구현한다고 가정해보자. 다양한 버전의 상품 페이지를 위한 사용자 인터페이스를 구현해야 한다.

- HTML5 / Javascript 기반의 데스크탑, 휴대기기 브라우저 UI 를 구현한다.
- 네이티브 안드로이드 / iOS 기반의 클라이언트를 구축한다 - 클라이언트는 서버와 REST API 를 이용해 상호작용한다.

게다가, 해당 쇼핑몰은 REST API 를 활용해 3rd party applications 에게 상품 상세 정보를 제공한다.

다음과 같은 상품 상세 정보를 UI 에 포함한다고 가정해보자. 예를 들어, 아마존의 도서 구입 페이지를 생각하면 된다.

- 책의 제목, 작가, 가격등의 기본 정보.
- 사용자의 책 구매 이력
- 구입 가능 여부
- 구매 옵션
- 일반적으로 책과 함께 같이 구입한 추천 품목
- 책을 구입한 사람들이 일반적으로 같이 구입한 품목
- 고객 리뷰
- 판매 랭킹 등

쇼핑몰은 MSA 구조이기 때문에 제품 상세 정보는 다양한 서비스로 전파된다.

- Product Info Service - 제목, 작가 등과 같은 제품의 기본 정보
- Pricing Service - 제품 가격
- Order Service - 제품 구매 이력 서비스
- Inventory Service - 제품 구매 여부 서비스
- Review Service - 고객 리뷰 서비스 등

위 서비스들에 일관적으로 제품 상세 정보를 구성하기 위한 fetch 작업을 진행해야 한다.

## 문제

MSA 기반의 애플리케이션의 클라이언트는 각 서비스에 어떻게 접근할 수 있을까?

## 제한사항

- 마이크로서비스가 제공하는 API 의 세밀함은 클라이언트의 요구사항과 다를 수 있다. 마이크로서비스는 일반적으로 세분화된 API 를 제공하고 클라이언트는 요구사항에 맞게 다양한 서비스와 상호작용 해야한다.
- 클라이언트마다 필요로 하는 데이터가 다르다. 예를 들어 데스크탑 버전의 브라우저가 요구하는 데이터는 모바일 브라우저가 요구하는 데이터보다 더 정교하다.
- 클라이언트마다 네트워크 성능이 다르다. 예를 들어, 일반적인 모바일 네트워크 환경은 모바일 네트워크가 아닌 환경보다 레이턴시(지연)가 발생한다. WAN 은 LAN 보다 느리다.
- 서비스를 제공하는 인스턴스의 수와 인스턴스의 위치(host + port)는 가변적이다.
- 서비스가 파티셔닝 된 정보는 클라이언트에게 숨겨져있다.
- 서비스는 다양한 종류의 프로토콜을 사용할 수 있고 그 중에는 웹에 적합하지 않을 수 있다.

## 해결방법

모든 클라이언트를 위한 단일 엔트리 포인트인 API gateway 를 구현한다. API gateway 는 두 가지 방법 중 하나의 방법으로 요청을 처리한다. 일부 요청은 적절한 서버로 단순하게 프록시 / 라우팅 처리를 한다. 그것은 요청들을 다양한 서비스로 나눠준다.

![image](https://user-images.githubusercontent.com/39546083/122069247-fee2e700-ce2f-11eb-9fdd-4a1a0dc58c8b.png)

API 게이트웨이는 모든 유형의 단일 API를 제공하는 대신 각 클라이언트에 대해 다른 API를 표시할 수 있다. API 게이트웨이가 보안을 구현할 수 도 있다.

[Embracing the Differences : Inside the Netflix API Redesign](https://netflixtechblog.com/embracing-the-differences-inside-the-netflix-api-redesign-15fd8b3dc49d)

### 변동: 프론트엔드의 백엔드

이 패턴의 변형된 패턴은 Backends for frontends 패턴이다. 이것은 클라이언트의 다양한 종류마다 분리된 API 게이트웨이를 정의한다.

![image](https://user-images.githubusercontent.com/39546083/122069370-18842e80-ce30-11eb-8cfb-167a768b68ac.png)

위 그림에서 클라이언트는 Web application / Mobile App / 3rd party applications 세 종류다. 클라이언트 종류 만큼 API 게이트웨이가 구현되어 있고 클라이언트에 적합한 API 를 제공한다.

### 예시

예시 코드: [cer/event-sourcing-examples](https://github.com/cer/event-sourcing-examples)

### 결론

**장점**

- 클라이언트를 마이크로서비스와 격리해 애플리케이션의 구성을 감출 수 있다.
- 서비스 인스턴스의 위치를 결정하는 문제로부터 클라이언트를 보호한다.
- 각 클라이언트에 필요한 최적의 API 를 제공한다.
- 요청 / 응답을 최소화 할 수 있다. 예를 들어 API 게이트웨이는 다양한 서비스에서 구성된 데이터를 한번의 요청 / 응답으로 탐색할 수 있다. 적은 요청은 오버헤드를 감소시켜 사용자 경험을 증가시킨다. API 게이트웨이는 모바일 애플리케이션에서 필수 요소다.
- 클라이언트에서 API 게이트웨이로 다양한 서비스로의 API 호출 로직이 이동되므로 클라이언트가 단순해진다.
- 웹 친화적인 표준 API 프로토콜에서 내부적으로 사용하는 프로토콜로 변환할 수 있다.

**단점**

- 복잡도의 증가 - API 게이트웨이는 개발하고 관리되어야 할 부품이다.
- API 게이트웨이를 통한 추가 네트워크 홉(network hop)으로 인해 응답 시간이 증가했지만 대부분의 애플리케이션의 경우 추가 왕복 비용이 미미하다.

**이슈**

- API 게이트웨이를 구현하기 위해서는 이벤트 방식 / 리액티브로 접근해야 한다. 이 방식은 높은 부하를 감당할 수 있다.