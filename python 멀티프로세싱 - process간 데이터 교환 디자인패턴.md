.

Data_Engineering & Python_TIL(20230312)

인프런 “고수가 되는 파이썬 : 동시성과 병렬성 문법 배우기 Feat. 멀티스레딩 vs 멀티프로세싱 (Inflearn Original)” 강의를 공부하고 정리한 내용입니다.

** URL : https://www.inflearn.com/course/프로그래밍-파이썬-완성-인프런-오리지널

## [공부한 내용]

** 파이썬 참고문서 : https://docs.python.org/3/library/multiprocessing.html#exchanging-objects-between-processes

### 디자인패턴 예시 1. Queue를 이용한 멀티프로세스간 데이터 교환 예시코드

```python
# 프로세스 통신 구현 - Queue

from multiprocessing import Process, Queue, current_process
import time
import os

# 실행 함수
def worker(id, baseNum, q):

    process_id = os.getpid()
    process_name = current_process().name

    # 누적
    sub_total = 0

    # 계산
    for i in range(baseNum):
        sub_total += 1

    # Produce
    q.put(sub_total)

    # 정보 출력
    print(f"Process ID: {process_id}, Process Name: {process_name}")
    print(f"Result : {sub_total}")

def main():

    # 부모 프로세스 아이디
    parent_process_id = os.getpid()
    # 출력
    print(f"Parent process ID {parent_process_id}")

    # 프로세스 리스트  선언
    processes = list()

    # 시작 시간
    start_time = time.time()

    # Queue 선언
    q = Queue()

     # 프로세스 생성 및 실행
    for i in range(5): # 1 ~ 100 적절히 조절
        # 생성
        t = Process(name=str(i), target=worker, args=(1, 100000000, q))

        # 배열에 담기
        processes.append(t)

        # 시작
        t.start()

    # Join
    for process in processes:
        process.join()

    # 순수 계산 시간
    print("--- %s seconds ---" % (time.time() - start_time))

    # 종료 플래그
    q.put('exit')

    total = 0

    # 대기
    while True:
        tmp = q.get()
        if tmp == 'exit':
            break
        else:
            total += tmp

    print()

    print("Main-Processing Total_count={}".format(total))
    print("Main-Processing Done!")

if __name__ == "__main__":
    main()
```

위에 코드를 실행하면 아래와 같이 출력됨

```text
Parent process ID 77373
Process ID: 77376, Process Name: 1
Result : 100000000
Process ID: 77375, Process Name: 0
Result : 100000000
Process ID: 77377, Process Name: 2
Result : 100000000
Process ID: 77379, Process Name: 4
Result : 100000000
Process ID: 77378, Process Name: 3
Result : 100000000
--- 3.417874813079834 seconds ---

Main-Processing Total_count=500000000
Main-Processing Done!
```

### 디자인패턴 예시 2. Pipe를 이용한 멀티프로세스간 데이터 교환 예시코드

```python
# 프로세스 통신 구현 - Pipe

from multiprocessing import Process, Pipe, current_process
import time
import os

# 실행 함수
def worker(id, baseNum, conn):

    process_id = os.getpid()
    process_name = current_process().name

    # 누적
    sub_total = 0

    # 계산
    for _ in range(baseNum):
        sub_total += 1

    # Produce
    conn.send(sub_total)
    conn.close()

    # 정보 출력
    print(f"Process ID: {process_id}, Process Name: {process_name}")
    print(f"Result : {sub_total}")

    return True

def main():

    # 부모 프로세스 아이디
    parent_process_id = os.getpid()
    # 출력
    print(f"Parent process ID {parent_process_id}")

    # 시작 시간
    start_time = time.time()

    # Pipe 선언
    parent_conn, child_conn = Pipe()

    processes = list()
    shared_result = 0

    # 프로세스 생성 및 실행
    for i in range(5):
        # 생성
        t = Process(target=worker, args=(1, 100000000, child_conn))
        
        # 배열에 담기
        processes.append(t)

        # 시작
        t.start()

    # Join
    for process in processes:
        shared_result += parent_conn.recv()
        process.join()

    # 순수 계산 시간
    print("--- %s seconds ---" % (time.time() - start_time))

    print("Main-Processing : {}".format(shared_result))
    print("Main-Processing Done!")

if __name__ == "__main__":
    main()
```

```text
Parent process ID 77328
Process ID: 77332, Process Name: Process-3
Result : 100000000
Process ID: 77334, Process Name: Process-5
Result : 100000000
Process ID: 77333, Process Name: Process-4
Result : 100000000
Process ID: 77330, Process Name: Process-1
Result : 100000000
Process ID: 77331, Process Name: Process-2
Result : 100000000
--- 3.3975470066070557 seconds ---
Main-Processing : 500000000
Main-Processing Done!
```