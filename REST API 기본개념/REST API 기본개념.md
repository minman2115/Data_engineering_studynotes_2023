.

Data_Engineering_TIL(20230108)

### Youtube 생활코딩 "기계들의 대화법 - REST API" 자료를 공부하고 정리한 내용입니다.

** URL : https://youtu.be/PmY3dWcCxXI

## [공부한 내용]

기계와 기계가 규격화된 방식으로 인터넷의 웹을 이용해서 통신을 돕는 규칙인 REST API 라는 것이 있다.

REST API는 웹의 통신규약인 HTTP를 이용한다.

API라는 것은 컴퓨터의 기능을 실행시키는 방법을 의미한다.

예를 들어서 컴퓨터 화면에 "hello world"라는 글짜를 띄우는 방법은 프로그래밍 언어마다 제각각 다르다. 

python은 아래와 같이 표현하고, 

```python
print('hello world')
```

Java script는 아래와 같이 표현할 것이다.

```javascript
document.write('hello world')
```

이때 print, doucument, write 이 하나 하나가 API라고 할수 있다.

REST API도 마찬가지로 컴퓨터에 어떤 기능을 실행하는 명령이라고 할 수 있다. 그런데 REST API는 내 컴퓨터에 기능을 실행하는 명령을 하는게 아니라 다른 (사람, 회사, 서버) 컴퓨터에 기능을 실행하는 명령어다.

예를 들어서 나의 application이 아래와 같은 URL 주소로 접속하게 되면 구글캘린더에 등록되어 있는 헬스장 가는 일정을 출력해주기도 한다.

```console
$ curl https://www.googleapis.com/…/calendars/calendar_id
{
    "summar".:"minman 헬스장 가는 일정",
    "timeZone": "Asia/Seoul"
}
```

또는 아래와 같이 트위터의 어떤 글내용을 가져올수도 있다.

```console
$ https://api.twitter.com/2/tweets/1067094924124872705
{ 
    "data": {
        "id": "1067094924124872705",
        "text": "Just getting started with Twitter APIs"
    }
}
```

위와 같이 어떤 컨텐츠를 가져올수 있는것 뿐만 아니라 이 내용을 추가하고 삭제하고 수정하는것도 가능하다.

이렇게 인터넷과 웹을 통해서 나의 컴퓨터를 제어할때 어떻게하면 시행착오를 줄이고 더 좋은 API를 만들수 있을지에 대한 고민의 결과물이 REST API다.

REST API는 어떤 특정 기술을 의미하는 것이 아니다. HTTP를 이용해서 기계들이 통신을 할때 HTTP가 가지고 있는 장점과 기능을 최대한 이용할수 있도록 유도하기 위한 노하우 또는 모범사례라고 할 수 있다.

만약에 내가 블로그나 SNS를 운영한다고 가정하자. 

글 하나하나를 topic이라고 한다면 아래와 같은 테이블 형태로 데이터를 가지고 있을 것이다. 이 데이터들을 REST API에서는 Resource라고 부른다.

Resource를 REST API로 표현해보자.

Resource는 URI를 통해서 표현을 하게 된다.

이때 만약에 topic 전체를 식별하고 싶거나 여러개의 topic을 식별하고 싶다면 `http://example.com/topics` 이런식으로 URI를 사용하면 된다. 이러한 것을 Collection이라고 한다. Collection은 URI 뒷쪽에 topics라는것을 보면 짐작이 가능하겠지만 복수형을 사용하게 된다.

그리고 테이블에서 한건 한건의 레코드들을 Element라고 부른다. 즉 Collection은 Element들이 모여있는 것을 말하고, Collection의 하나하나의 데이터를 Element라고 말한다. 일반적으로는 id 값을 Element에서 구분자로 사용한다. 그런데 만약에 이름으로도 구분자로 활용이 가능하다면 이름으로도 구분자로 사용이 가능하다.

<img width="1054" alt="1" src="https://user-images.githubusercontent.com/41605276/211178210-b9753b32-4566-421a-a0a7-7687c2aa8ded.png">

그런데 Resource를 URI로 구분하는 것 만으로는 어떤 action도 할수가 없다. URI는 단지 그 정보를 식별하는 이름일 뿐이고 이 정보를 가공할수 있어야 한다.

Resource에 대한 가공은 CRUD라고 부르는 방법에 따라 가공할 수 있다. 이러한 가공 방법을 REST API에서는 METHOD 라고 부른다.

```text
C : Create
R : Read
U : Update
D : Delete
```

REST API는 웹의 통신규약인 HTTP를 이용하기 때문에 HTTP가 갖고 있는 메소드를 이용한다.

REST API에서 Create는 HTTP에서 POST 메소드로 하게 된다. 웹 어플리케이션은 POM을 이용해서 데이터를 전송할때 수정/생성/삭제할시 POST 메소드를 사용하지만 사실 POST는 본래 생성을 위해 만들어진 기능이다. REST API는 HTTP의 메소드들을 본래의 용도에 맞게 사용하자라는 것도 중요한 목표이기 때문에 POST는 Create를 위해 사용한다.

Read는 GET, 삭제는 DELETE 메소드를 사용한다.

Update는 전체 내용을 교체하는 PUT 메소드와, 부분 내용을 교체하는 PATCH 메소드 두개를 사용한다.

```text
 REST API       HTTP 메소드
C : Create  -->   POST
R : Read    -->   GET
U : Update  -->   PUT AND PATCH
D : Delete  -->   DELETE
```