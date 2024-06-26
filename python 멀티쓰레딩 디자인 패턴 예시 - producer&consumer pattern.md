.

Data_Engineering & Python_TIL(20230311)

인프런 “고수가 되는 파이썬 : 동시성과 병렬성 문법 배우기 Feat. 멀티스레딩 vs 멀티프로세싱 (Inflearn Original)” 강의를 공부하고 정리한 내용입니다.

** URL : https://www.inflearn.com/course/프로그래밍-파이썬-완성-인프런-오리지널

## [공부한 내용]

### 실제 코드 예시

```python
import concurrent.futures
import logging
import queue
import random
import threading
import time

# producer
def producer(queue, event):
    """네트워크 대기 상태라 가정(서버)"""
    while not event.is_set():
        message = random.randint(1, 11)
        logging.info("Producer got message: %s", message)
        queue.put(message)

    logging.info("Producer received event. Exiting")

# consumer
def consumer(queue, event):
    """응답 받고 소비하는 것으로 가정 or DB 저장"""
    while not event.is_set() or not queue.empty():
        message = queue.get()
        logging.info("Consumer storing message: %s (size=%d)", message, queue.qsize())

    logging.info("Consumer received event. Exiting")

if __name__ == "__main__":
    # Logging format 설정
    format = "%(asctime)s: %(message)s"
    logging.basicConfig(format=format, level=logging.INFO, datefmt="%H:%M:%S")

    # 사이즈 중요
    pipeline = queue.Queue(maxsize=10)

    # 이벤트 플래그 초기 값 0
    # Python Event 객체
    # (1). Flag 초기값(0)
    # (2). Set() -> 1, Clear() -> 0, Wait(1 -> 리턴, 0 -> 대기), isSet() -> 현 플래그 상태
    event = threading.Event()

    # With Context 시작
    with concurrent.futures.ThreadPoolExecutor(max_workers=2) as executor:
        executor.submit(producer, pipeline, event)
        executor.submit(consumer, pipeline, event)

        # 실행 시간 조정
        time.sleep(0.1)

        logging.info("Main: about to set event")
        
        # 프로그램 종료
        event.set()
```

위에 코드 실행결과는 아래와 같음

```text
19:34:22: Producer got message: 8
19:34:22: Producer got message: 11
19:34:22: Producer got message: 1
19:34:22: Producer got message: 8
19:34:22: Producer got message: 8
19:34:22: Producer got message: 7
19:34:22: Producer got message: 11
19:34:22: Producer got message: 7
19:34:22: Producer got message: 2
19:34:22: Consumer storing message: 10 (size=9)
19:34:22: Consumer storing message: 2 (size=8)
19:34:22: Consumer storing message: 8 (size=7)
19:34:22: Consumer storing message: 11 (size=6)
19:34:22: Consumer storing message: 1 (size=5)
19:34:22: Consumer storing message: 8 (size=4)
19:34:22: Consumer storing message: 8 (size=3)
19:34:22: Consumer storing message: 7 (size=2)
19:34:22: Consumer storing message: 11 (size=2)
19:34:22: Producer got message: 4
19:34:22: Producer got message: 5
19:34:22: Producer got message: 1
19:34:22: Producer got message: 6
19:34:22: Producer got message: 3
19:34:22: Producer got message: 2

...

19:34:22: Producer got message: 8
19:34:22: Producer got message: 2
19:34:22: Producer got message: 2
19:34:22: Producer got message: 3
19:34:22: Producer got message: 4
19:34:22: Main: about to set event
19:34:22: Consumer storing message: 4 (size=9)
19:34:22: Consumer storing message: 5 (size=8)
19:34:22: Consumer storing message: 6 (size=7)
19:34:22: Consumer storing message: 3 (size=6)
19:34:22: Consumer storing message: 10 (size=5)
19:34:22: Consumer storing message: 4 (size=4)
19:34:22: Consumer storing message: 8 (size=3)
19:34:22: Consumer storing message: 2 (size=2)
19:34:22: Consumer storing message: 2 (size=1)
19:34:22: Consumer storing message: 3 (size=0)
19:34:22: Consumer received event. Exiting
19:34:22: Producer received event. Exiting
```