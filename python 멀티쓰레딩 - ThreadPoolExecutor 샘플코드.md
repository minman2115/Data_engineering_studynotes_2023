.

Data_Engineering & Python_TIL(20230312)

인프런 “고수가 되는 파이썬 : 동시성과 병렬성 문법 배우기 Feat. 멀티스레딩 vs 멀티프로세싱 (Inflearn Original)” 강의를 공부하고 정리한 내용입니다.

** URL : https://www.inflearn.com/course/프로그래밍-파이썬-완성-인프런-오리지널

## [공부한 내용]

### 샘플코드

```python
import logging
from concurrent.futures import ThreadPoolExecutor
import time

# 스레드 실행 함수
def task(name):
    logging.info("Sub-Thread %s: starting", name)

    result = 0
    for i in range(10001):
        result = result + i

    logging.info("Sub-Thread %s: finishing result: %d", name, result)

    return True


# 메인 영역
def main():
    # Logging format 설정
    format = "%(asctime)s: %(message)s"
    logging.basicConfig(format=format, level=logging.INFO, datefmt="%H:%M:%S")

    logging.info("Main-Thread : before creating and running thread")

    # 실행 방법1
    # with context 구문 사용

    with ThreadPoolExecutor(max_workers=3) as executor:
        tasks = executor.map(task, ['First', 'Second'])
        
        # 결과 확인
        # print(list(tasks))

    logging.info("Main-Thread : all done")


    ################################################################
    # 실행 방법2
    # max_workers : 작업의 개수가 남어가면 직접 설정이 유리
    # executor = ThreadPoolExecutor(max_workers=3)
    
    # task1 = executor.submit(task, ('First',))
    # task2 = executor.submit(task, ('Second',))

    # 결과 값 있을 경우
    # print(task1.result())
    # print(task2.result())
    ################################################################

if __name__ == '__main__':
    main()
```

위에 코드를 실행하면 아래와 같이 출력됨

```text
11:48:39: Main-Thread : before creating and running thread
11:48:39: Sub-Thread First: starting
11:48:39: Sub-Thread First: finishing result: 50005000
11:48:39: Sub-Thread Second: starting
11:48:39: Sub-Thread Second: finishing result: 50005000
11:48:39: Main-Thread : all done
```