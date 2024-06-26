.

Data_Engineering & Python_TIL(20230311)

인프런 “고수가 되는 파이썬 : 동시성과 병렬성 문법 배우기 Feat. 멀티스레딩 vs 멀티프로세싱 (Inflearn Original)” 강의를 공부하고 정리한 내용입니다.

** URL : https://www.inflearn.com/course/프로그래밍-파이썬-완성-인프런-오리지널

## [공부한 내용]

### 용어 기본개념

```text
1. 세마포어(Semaphore) : 프로세스간 공유된 자원에 접근 시 문제 발생 가능성이 있기 때문에 한 개의 프로세스만 접근하는 개념(경쟁상태 예방을 목적으로함)
2. 뮤텍스(Mutex) : 공유된 자원의 데이터를 여러 스레드가 접근하는 것을 막는 것. (경쟁상태 예방을 목적으로함)
3. Lock : 상호 배제를 위한 잠금(Lock)처리(데이터 경쟁)
4. 데드락(Deadlock) : 프로세스가 자원을 획득하지 못해 다음 처리를 못하는 무한 대기 상황(교착상태)
5. Thread synchronization(스레드 동기화)를 통해서 안정적으로 동작하게 처리한다.(동기화메소드, 동기화 블록)
6. Semaphore와 Mutex의 공통점과 차이점
   공통점
      - 세마포어와 뮤텍스 개체는 모두 병렬 프로그래밍 환경에서 상호 배제를 위해 사용 
   차이점
      - 뮤텍스 개체는 단일 스레드가 리소스 또는 중요 섹션을 소비 허용
      - 세마포어는 리소스에 대한 제한된 수의 동시 액세스를 허용
```

### 실제 코드 예시

```python
import logging
from concurrent.futures import ThreadPoolExecutor
import time
import threading


class FakeDataStore:
    # 공유 변수(value)
    def __init__(self):
        self.value = 0
        # Lock 선언
        self._lock = threading.Lock()

    # 변수 업데이트 함수
    def update(self, n):
        logging.info("Thread %s: starting update", n)

        # 뮤텍스 & Lock 등 동기화(Thread synchronization) 필요

        # Lock 획득(방법1)
        with self._lock:
             logging.info("Thread %s has lock", n)

             local_copy = self.value
             local_copy += 1
             time.sleep(0.1)
             self.value = local_copy

             logging.info("Thread %s about to release lock", n)

        logging.info("Thread %s: finishing update", n)

        ############################################################
        # Lock 획득(방법2)
        #self._lock.acquire()
        #logging.info("Thread %s has lock", n)
        
        #local_copy = self.value
        #local_copy += 1
        #time.sleep(0.1)
        #self.value = local_copy

        #logging.info("Thread %s about to release lock", n)

        # Lock 반환
        #self._lock.release()
        ############################################################

if __name__ == "__main__":
    # Logging format 설정
    format = "%(asctime)s: %(message)s"
    logging.basicConfig(format=format, level=logging.INFO, datefmt="%H:%M:%S")

    # 클래스 인스턴스화
    store = FakeDataStore()

    logging.info("Testing update. Starting value is %d.", store.value)

    # With Context 시작
    with ThreadPoolExecutor(max_workers=2) as executor:
        for n in ['First', 'Second', 'Third']:
            executor.submit(store.update, n)

    logging.info("Testing update. Ending value is %d.", store.value)
```

위에 코드를 실행하면 아래와 같이 출력된다.

```text
20:36:15: Testing update. Starting value is 0.
20:36:15: Thread First: starting update
20:36:15: Thread First has lock
20:36:15: Thread Second: starting update
20:36:15: Thread First about to release lock
20:36:15: Thread First: finishing update
20:36:15: Thread Third: starting update
20:36:15: Thread Second has lock
20:36:16: Thread Second about to release lock
20:36:16: Thread Second: finishing update
20:36:16: Thread Third has lock
20:36:16: Thread Third about to release lock
20:36:16: Thread Third: finishing update
20:36:16: Testing update. Ending value is 3.
```