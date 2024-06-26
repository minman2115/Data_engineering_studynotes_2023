.

Data_Engineering_TIL(20230108)

### Youtube 생활코딩 "Amazon API Gateway" 자료를 공부하고 정리한 내용입니다.

** URL : https://youtu.be/60goWpADp-I

### "REST API 기본개념" 글에 이어서 공부한 내용입니다.

** URL : https://minman2115.github.io/DE_TIL379

## [공부한 내용]

- API Gateway란

단일 진입점으로 클라이언트의 API 호출을 기능과 상황에 맞게 뒷단의 백엔드에 라우팅(매핑)시켜주는 서비스

API Gateway 서비스는 세가지의 작은 제품으로 구성되어 있다.

- HTTP API

- REST API

- WebSocket API

일단 HTTP API와 REST API가 비슷한데 서로 상대적으로 비교하면 아래와 같다.

```text
HTTP API      REST API
단순함          복잡함
저렴함          비쌈
빠름           느림
```

- API Gateway의 필요성 예시

REST API와 같이 인터넷 기반의 API를 사용할때 여러 서버에 API가 분산되어 있는 경우가 있다.

그리고 각 서버들은 각각 도메인 주소가 다를수 있다. 클라이언트 측에서는 API 서버들을 호출하는 수많은 코드들이 존재할 수 있다. 

이때 만약에 example.org 서버가 example.com 서버로 통합되었다고 가정하자. 

example.org 서버를 호출하는 코드가 1억개가 있었다면 이 1억개 코드 모두 에러가 발생할 것이다.

<img width="1183" alt="1" src="https://user-images.githubusercontent.com/41605276/211182743-3df8f7b2-61e0-4056-b1ff-f78138c85774.png">

이거를 다시 동작하게 하려면 1억개의 exmaple.org 코드를 모두 example.com로 바꿔줘야 할 것이다.

이럴때 API Gateway를 사용하게 되면 일단 이 API Gateway는 자신만의 주소를 갖고 있는데 (example.io 라고 가정) 그리고 아래 그림과 같이 클라이언트에서 호출하는 example.io 의 하위 경로가 어떤건지에 따라 뒷단에 각각의 서버와 연결시킬 수 있다.

<img width="1138" alt="2" src="https://user-images.githubusercontent.com/41605276/211183690-0eb7c531-d371-4f04-a498-5f05baa49fc0.png">

그러면 위와 같은 상황처럼 example.org 서버가 example.com 서버로 통합되었다고 해도 아래 그림과 같이 API 게이트웨이의 설정을 바꿔주면 된다.

클라이언트의 어떤 코드도 변경하지 않고, 에러없이 동작이 가능하다.

<img width="1141" alt="3" src="https://user-images.githubusercontent.com/41605276/211183752-15ba8f03-9cb2-4765-b969-8dec3e81fa76.png">

뿐만 아니라 외부에서 서버들로 접근할때 유일한 접점이기 때문에 아래와 같이 얻을수 있는 장점들이 있다.

아래와 같은 기능을 API Gateway에서 통합적으로 설정할 수 있다.

```text
인증
액세스 제어
모니터링
로깅
```

또한 API Gateway 뒷단에 Lambda를 연결하여 서버리스 플랫폼을 만들수 있는 장점도 있다.