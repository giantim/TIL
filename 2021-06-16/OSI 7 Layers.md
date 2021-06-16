[[10분 테코톡\] 🔮 히히의 OSI 7 Layer](https://www.youtube.com/watch?v=1pfTxp25MA8) 정리하기

## Layer 1 - Physical Layer

0 과 1을 주고받는 계층. 실제로는 잘 동작하지 않는다.

![image](https://user-images.githubusercontent.com/39546083/122248423-04f5c800-cf03-11eb-9a02-17a6cf0a018b.png)

→ 시간 당 전압을 나타내는 그래프. 전자기파를 표현하는 함수.

위 그래프의 주파수를 구해보면? 주파수는 1초동안 진동한 횟수. 주파수는 4Hz.

![image](https://user-images.githubusercontent.com/39546083/122248439-07f0b880-cf03-11eb-8610-1ac69bc101ae.png)

→ 전선은 모든 주파수를 통과시키지 못한다.

위 주파수를 특정 전선으로 받으면 보낸 신호와 받은 신호가 달라질 수 있다.

![image](https://user-images.githubusercontent.com/39546083/122248547-1a6af200-cf03-11eb-9437-675dae103347.png)

→ 위와 같은 전기신호를 통과시킬 수 있는 전선은 없다.

→ 아날로그 신호로 바꿔서 전송해야 한다.

![image](https://user-images.githubusercontent.com/39546083/122248564-1e970f80-cf03-11eb-8e26-2db65eeecce7.png)

![image](https://user-images.githubusercontent.com/39546083/122248588-21920000-cf03-11eb-8787-2b37e067eee4.png)

이 기술은 물리적인 칩에 구현되어 있다. → 하드웨어적으로 구현되어 있다.

## Layer 2 - Data-Link Layer

### 여러 대의 컴퓨터 간의 통신

컴퓨터는 전선으로 연결되어 있다면 통신이 가능하다. 물리적 전선으로 연결되지 않은 컴퓨터와 데이터를 통신하는 방법을 알아야 한다.

![image](https://user-images.githubusercontent.com/39546083/122248604-248cf080-cf03-11eb-880d-020d01446e3e.png)

→ 전선 하나를 이용해 여러 대의 컴퓨터와 통신하는 방법을 모색해야 한다.

![7](https://user-images.githubusercontent.com/39546083/122248617-26ef4a80-cf03-11eb-93b7-60406cfee15b.png)

![8](https://user-images.githubusercontent.com/39546083/122248631-29ea3b00-cf03-11eb-96e7-02727c477dea.png)

→ 위와 같은 구조는 모든 컴퓨터가 동일한 데이터를 수신할 수 있다는 것이다. 메시지의 목적지를 파악할 수 있어야 한다.

→ 상자를 스위치로 바꾸어야 한다.

![9](https://user-images.githubusercontent.com/39546083/122248674-32db0c80-cf03-11eb-98a2-1ab37adbe075.png)

- 예림 → 혜림에게 데이터를 보내려고 할 때 어떻게 해야 할까
- 스위치와 스위치를 연결해야 한다. → (스위치 → 라우터) 로 변경해야 한다. 라우터끼리는 연결이 가능하다. 라우터는 스위치의 역할도 할 수 있다.

![10](https://user-images.githubusercontent.com/39546083/122248684-34a4d000-cf03-11eb-8395-aba11a09c528.png)

- 라우터와 라우터를 라우터를 이용해서 연결한다.
- 전 세계의 컴퓨터들을 연결한 것을 `인터넷`이라고 한다.

### Data Link Layer

![11](https://user-images.githubusercontent.com/39546083/122248696-379fc080-cf03-11eb-9752-5d2d73803df7.png)

![12](https://user-images.githubusercontent.com/39546083/122248708-39698400-cf03-11eb-8b53-b4ddddc9707e.png)

- 동시에 여러 데이터를 받았을 때 어떻게 올바른 데이터로 끊어서 읽을 수 있을까

![13](https://user-images.githubusercontent.com/39546083/122248715-3bcbde00-cf03-11eb-8be1-38b6159856b4.png)

![14](https://user-images.githubusercontent.com/39546083/122248731-3ec6ce80-cf03-11eb-966b-e46c680f4f82.png)

- 같은 네트워크에 있는 컴퓨터: 스위치로 연결된 컴퓨터들

2계층도 1계층 처럼 하드웨어에 구현되어 있다.

## Layer 3 - Network Layer

![15](https://user-images.githubusercontent.com/39546083/122248747-438b8280-cf03-11eb-9297-4746310475be.png)

![16](https://user-images.githubusercontent.com/39546083/122248754-45554600-cf03-11eb-9b1a-650b7869e71f.png)

- A는 데이터 앞에 목적지 주소, B의 주소를 붙인다.
- 각 컴퓨터들이 갖는 고유한 주소를 `IP 주소` 라고 한다.
- A는 B의 IP 주소를 알고 있어야 한다. → 어떻게 알 수 있을까?
- DNS 를 이용하면 연결된 컴퓨터의 IP 주소를 알 수 있다.

![17](https://user-images.githubusercontent.com/39546083/122248763-471f0980-cf03-11eb-810a-ad4e3da963dc.png)

이것은 패킷!

- 라우터는 패킷을 열어 목적지 주소(IP 주소)를 확인한다. 라우터는 확인한 주소가 본인이 담당하는 컴퓨터 중 해당하는 컴퓨터가 있는지 확인한다. 없다면 상위 라우터로 패킷을 전달한다.
- 위 그림에서 패킷을 전달할 때 `마 → 바` 로 어떻게 전달하는지 알 수 있을까? → 라우터에 대해 공부해 보기

![18](https://user-images.githubusercontent.com/39546083/122248778-4a19fa00-cf03-11eb-8b3c-464feeb38cca.png)

네트워크 기술이 구현된 것은 `운영체제 커널에 소프트웨어` 로 구현되어 있다.

## Layer 4 - Transport Layer

모든 컴퓨터끼리 데이터를 주고 받을 수 있다.

컴퓨터에는 여러개의 프로그램이 실행되고 있다.

실행중인 프로그램 → 프로세스

어떤 데이터를 무슨 프로세스에게 줄 지 어떻게 알 수 있을까?

프로세스는 `포트 번호`를 갖는다. 포트 번호는 하나의 컴퓨터에서 동시에 실행되고 있는 프로세스들이 서로 겹치지 않게 가져야하는 정수 값.

![19](https://user-images.githubusercontent.com/39546083/122248785-4b4b2700-cf03-11eb-9236-38487edae221.png)

미리 포트 번호를 알고 있어야 한다.

![20](https://user-images.githubusercontent.com/39546083/122248794-4dad8100-cf03-11eb-82f1-096cde97b0d8.png)

data → (4계층 인코더) → data + 포트 번호 → 3계층 인코더 → ip + data → 1 ~ 2 계층 인코더 → 아날로그 신호로 변환 → 역으로 디코딩

해당 기술은 `운영체제의 커널`에 소프트웨어로 구현되어 있다.

## Application Layer

![21](https://user-images.githubusercontent.com/39546083/122248808-4f774480-cf03-11eb-9150-71f87c231281.png)

현대의 인터넷은 OSI 모델이 아닌 TCP/IP 모델을 따르고 있다.

TCP/IP 모델도 네트워크 시스템에 대한 모델이다. → 시장 점유 싸움에서 이겼다.

![22](https://user-images.githubusercontent.com/39546083/122248816-51410800-cf03-11eb-8a77-a0cbd46bcaf9.png)

- TCP/IP 모델이 업데이트 되어서 바뀌었다.

![23](https://user-images.githubusercontent.com/39546083/122248823-530acb80-cf03-11eb-9756-bb9ef2864e54.png)

- 오늘날은 Updated 모델이 더 많이 사용되고 있다.

### TCP/IP 소켓 프로그래밍

운영체제의 `Transport Layer` 에서 제공하는 API 를 활용해서 통신 가능한 프로그램을 만드는 것. 또는 네트워크 프로그래밍이라고 한다. 소켓 프로그래밍 만으로도 클라이언트, 서버 프로그램을 따로 만들어서 동작이 가능하다. TCP/IP 소켓 프로그래밍을 통해서 누구나 `자신만의 Application Layer 인코더 / 디코더`를 만들 수 있다. → 자신만의 Application Protocol 을 만들 수 있다.

`HTTP`: 대표적인 Application Layer 프로토콜

![24](https://user-images.githubusercontent.com/39546083/122248829-54d48f00-cf03-11eb-871f-2119960cd5b7.png)

**네트워크 시스템 → Layerd Architecture → 하나의 커다란 소프트웨어**