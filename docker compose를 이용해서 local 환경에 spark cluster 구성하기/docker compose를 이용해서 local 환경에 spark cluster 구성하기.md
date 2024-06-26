Data_Engineering_TIL(20240523)

### 학습시 참고자료
[Spark] 클러스터 구축 docker-compose, standalone (2/5) 블로그 글 자료

URL : https://ampersandor.tistory.com/11?category=1170869

### 개요
Mac OS local 환경에 docker compose를 이용해서 spark cluster를 구성하는 방법

![20](https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/562c910b-4ae4-4efd-b5d1-e4268cf77619)


### 구성방법 요약
STEP 1) 작업 folder 생성

STEP 2) custom image Dockerfile 작성
- cluster-base.Dockerfile
- spark-base.Dockerfile
- spark-master.Dockerfile
- spark-worker.Dockerfile
- jupyterlab.Dockerfile

STEP 3) image build shell script (build.sh) 작성

STEP 4) docker-compose.yaml 작성

STEP 5) docker image build

STEP 6) docker compose up

STEP 7) Spark 관련 UI 접속해보기

STEP 8) Spark 실행 테스트

### 구성내용

#### STEP 1) 작업 folder 생성
아래와 같이 작업 폴더와 file들이 구성될 것이다.
그래서 먼저 작업 folder를 생성해준다.
```bash
$ pwd
/Users/minman/spark_test

$ tree
.
├── build.sh
├── cluster-base.Dockerfile
├── docker-compose.yaml
├── jupyterlab.Dockerfile
├── spark-base.Dockerfile
├── spark-master.Dockerfile
└── spark-worker.Dockerfile

1 directory, 7 files
```

#### STEP 2) custom image Dockerfile 작성

[cluster-base.Dockerfile]

<img width="565" alt="12" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/4eb214ae-5d46-4142-9726-dee63c589cce">

- 가장 기본이 되는 cluster-base 이미지
- 이 이미지를 활용하여 단순히 shared_workspace를 docker volume으로 공유를 하여 이 위에 spark 이미지들과 jupyter 이미지를 띄우는 구조
- 마치 실제 운영환경에서의 HDFS 위에 spark를 올리듯이 구성을 하는 구조
- cluster의 base image는 8-jre-slim
- spark 가 jvm 환경에서 돌아가며, java 를 linux os 이미지에서 설치하는 것보다 조금이라도 간편하게 docker 공식 사이트에서 배포되는 java runtime environment 이미지를 사용

```Dockerfile
ARG debian_buster_image_tag=8-jre-slim
FROM openjdk:${debian_buster_image_tag}

ARG shared_workspace=/opt/workspace

RUN mkdir -p ${shared_workspace} && \
    apt-get update -y && \
    apt-get install -y python3 && \
    ln -s /usr/bin/python3 /usr/bin/python && \
    rm -rf /var/lib/apt/lists/*

ENV SHARED_WORKSPACE=${shared_workspace}

VOLUME ${shared_workspace}
CMD ["bash"]
```

[spark-base.Dockerfile]

<img width="576" alt="13" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/d6793562-da49-4940-9d3f-9e2f81a45a9f">

- cluster base image를 기반으로 spark를 다운받고, 압축을 해제하고, 환경변수를 셋팅
- cluster-base가 jre를 기반으로 한 이미지이기 때문에 jvm설치가 필요없음

```Dockerfile
FROM cluster-base

ARG spark_version
ARG hadoop_version


RUN apt-get update -y && \
    apt-get install -y curl && \
    curl https://archive.apache.org/dist/spark/spark-${spark_version}/spark-${spark_version}-bin-hadoop${hadoop_version}.tgz -o spark.tgz && \
    tar -xf spark.tgz && \
    mv spark-${spark_version}-bin-hadoop${hadoop_version} /usr/bin/ && \
    mkdir /usr/bin/spark-${spark_version}-bin-hadoop${hadoop_version}/logs && \
    rm spark.tgz

ENV SPARK_HOME /usr/bin/spark-${spark_version}-bin-hadoop${hadoop_version}
ENV SPARK_MASTER_HOST spark-master
ENV SPARK_MASTER_PORT 7077
ENV PYSPARK_PYTHON python3

WORKDIR ${SPARK_HOME}
```

[spark-master.Dockerfile]

![14](https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/ca66420c-2be6-4b8e-a465-28ef58a5dc72)

```Dockerfile
FROM spark-base

ARG spark_master_web_ui=8080
EXPOSE ${spark_master_web_ui}
EXPOSE ${SPARK_MASTER_PORT}
CMD bin/spark-class org.apache.spark.deploy.master.Master >> logs/spark-master.out
```

[spark-worker.Dockerfile]

<img width="562" alt="15" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/7fe7abdf-3b21-442b-a7f6-21e65a885c79">

```Dockerfile
FROM spark-base

ARG spark_worker_web_ui=8081
EXPOSE ${spark_worker_web_ui}
CMD bin/spark-class org.apache.spark.deploy.worker.Worker spark://${SPARK_MASTER_HOST}:${SPARK_MASTER_PORT} >> logs/spark-worker.out
```

[jupyterlab.Dockerfile]

<img width="577" alt="16" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/13edcf43-4326-46c6-8b64-cd16ff172284">

```Dockerfile
FROM cluster-base

ARG spark_version
ARG jupyterlab_version

RUN apt-get update -y && \
    apt-get install -y python3-pip && \
    pip3 install wget pyspark==${spark_version} jupyterlab==${jupyterlab_version}

EXPOSE 8888
WORKDIR ${SHARED_WORKSPACE}
CMD jupyter lab --ip=0.0.0.0 --port=8888 --no-browser --allow-root --NotebookApp.token=
```

#### STEP 3) image build shell script (build.sh) 작성

- build.sh

```bash
# -- Software Stack Version
SPARK_VERSION="3.0.0"
HADOOP_VERSION="2.7"
JUPYTERLAB_VERSION="2.1.5"

# -- Building the Images
echo "############################################# building cluster-base #############################################"
docker build \
  -f cluster-base.Dockerfile \
  -t cluster-base .

echo "############################################# building spark-base #############################################"
docker build \
  --build-arg spark_version="${SPARK_VERSION}" \
  --build-arg hadoop_version="${HADOOP_VERSION}" \
  -f spark-base.Dockerfile \
  -t spark-base .

echo "############################################# building spark-master #############################################"
docker build \
  -f spark-master.Dockerfile \
  -t spark-master .

echo "############################################# building spark-worker #############################################"
docker build \
  -f spark-worker.Dockerfile \
  -t spark-worker .

echo "############################################# building jupyter #############################################"
docker build \
  --build-arg spark_version="${SPARK_VERSION}" \
  --build-arg jupyterlab_version="${JUPYTERLAB_VERSION}" \
  -f jupyterlab.Dockerfile \
  -t jupyterlab .
```

#### STEP 4) docker-compose.yaml 작성

- docker-compose.yaml
```yaml
version: "3.6"
volumes:
  shared-workspace:
    name: "hadoop-distributed-file-system"
    driver: local
services:
  jupyterlab:
    image: jupyterlab
    container_name: jupyterlab
    ports:
      - 8888:8888
    volumes:
      - shared-workspace:/opt/workspace
  spark-master:
    image: spark-master
    container_name: spark-master
    ports:
      - 4040:8080
      - 7077:7077
    volumes:
      - shared-workspace:/opt/workspace
  spark-worker-1:
    image: spark-worker
    container_name: spark-worker-1
    environment:
      - SPARK_WORKER_CORES=1
      - SPARK_WORKER_MEMORY=512m
    ports:
      - 4041:8081
    volumes:
      - shared-workspace:/opt/workspace
    depends_on:
      - spark-master
  spark-worker-2:
    image: spark-worker
    container_name: spark-worker-2
    environment:
      - SPARK_WORKER_CORES=1
      - SPARK_WORKER_MEMORY=512m
    ports:
      - 4042:8081
    volumes:
      - shared-workspace:/opt/workspace
    depends_on:
      - spark-master
```

#### STEP 5) docker image build

아래와 같이 작업폴더에서 명령어를 실행하여 image를 생성한다.

```bash
$ chmod +x ./build.sh

$ ./run_build.sh                                                
############################################# building cluster-base #############################################
[+] Building 1.6s (7/7) FINISHED                                                                                                                                            
 => [internal] load build definition from cluster-base.Dockerfile                                                                                                      0.0s
 => => transferring dockerfile: 451B                                                                                                                                   0.0s
 => [internal] load .dockerignore                                                                                                                                      0.0s
 => => transferring context: 2B                                                                                                                                        0.0s
 => [internal] load metadata for docker.io/library/openjdk:8-jre-slim                                                                                                  1.5s
 => [auth] library/openjdk:pull token for registry-1.docker.io                                                                                                         0.0s
 => [1/2] FROM docker.io/library/openjdk:8-jre-slim@sha256:xxxxxxxxxxxxxxxxxxxxxx                                            0.0s
 => CACHED [2/2] RUN mkdir -p /opt/workspace &&     apt-get update -y &&     apt-get install -y python3 &&     ln -s /usr/bin/python3 /usr/bin/python &&     rm -rf /  0.0s
 => exporting to image                                                                                                                                                 0.0s
 => => exporting layers                                                                                                                                                0.0s
 => => writing image sha256:xxxxxxxxxxxxxxxxxxxxxx                                                                           0.0s
 => => naming to docker.io/library/cluster-base                                                                                                                        0.0s

Use 'docker scan' to run Snyk tests against images to find vulnerabilities and learn how to fix them
############################################# building spark-base #############################################
[+] Building 0.1s (7/7) FINISHED                                                                                                                                            
 => [internal] load build definition from spark-base.Dockerfile                                                                                                        0.0s
 => => transferring dockerfile: 816B                                                                                                                                   0.0s
 => [internal] load .dockerignore                                                                                                                                      0.0s
 => => transferring context: 2B                                                                                                                                        0.0s
 => [internal] load metadata for docker.io/library/cluster-base:latest                                                                                                 0.0s
 => [1/3] FROM docker.io/library/cluster-base                                                                                                                          0.0s
 => CACHED [2/3] RUN apt-get update -y &&     apt-get install -y curl &&     curl https://archive.apache.org/dist/spark/spark-3.0.0/spark-3.0.0-bin-hadoop2.7.tgz -o   0.0s
 => CACHED [3/3] WORKDIR /usr/bin/spark-3.0.0-bin-hadoop2.7                                                                                                            0.0s
 => exporting to image                                                                                                                                                 0.0s
 => => exporting layers                                                                                                                                                0.0s
 => => writing image sha256:xxxxxxxxxxxxxxxxxxxxxx                                                                           0.0s
 => => naming to docker.io/library/spark-base                                                                                                                          0.0s

Use 'docker scan' to run Snyk tests against images to find vulnerabilities and learn how to fix them
############################################# building spark-master #############################################
[+] Building 0.1s (5/5) FINISHED                                                                                                                                            
 => [internal] load build definition from spark-master.Dockerfile                                                                                                      0.0s
 => => transferring dockerfile: 257B                                                                                                                                   0.0s
 => [internal] load .dockerignore                                                                                                                                      0.0s
 => => transferring context: 2B                                                                                                                                        0.0s
 => [internal] load metadata for docker.io/library/spark-base:latest                                                                                                   0.0s
 => CACHED [1/1] FROM docker.io/library/spark-base                                                                                                                     0.0s
 => exporting to image                                                                                                                                                 0.0s
 => => exporting layers                                                                                                                                                0.0s
 => => writing image sha256:xxxxxxxxxxxxxxxxxxxxxx                                                                           0.0s
 => => naming to docker.io/library/spark-master                                                                                                                        0.0s

Use 'docker scan' to run Snyk tests against images to find vulnerabilities and learn how to fix them
############################################# building spark-worker #############################################
[+] Building 0.1s (5/5) FINISHED                                                                                                                                            
 => [internal] load build definition from spark-worker.Dockerfile                                                                                                      0.0s
 => => transferring dockerfile: 279B                                                                                                                                   0.0s
 => [internal] load .dockerignore                                                                                                                                      0.0s
 => => transferring context: 2B                                                                                                                                        0.0s
 => [internal] load metadata for docker.io/library/spark-base:latest                                                                                                   0.0s
 => CACHED [1/1] FROM docker.io/library/spark-base                                                                                                                     0.0s
 => exporting to image                                                                                                                                                 0.0s
 => => exporting layers                                                                                                                                                0.0s
 => => writing image sha256:xxxxxxxxxxxxxxxxxxxxxx                                                                           0.0s
 => => naming to docker.io/library/spark-worker                                                                                                                        0.0s

Use 'docker scan' to run Snyk tests against images to find vulnerabilities and learn how to fix them
############################################# building jupyter #############################################
[+] Building 0.1s (7/7) FINISHED                                                                                                                                            
 => [internal] load build definition from jupyterlab.Dockerfile                                                                                                        0.0s
 => => transferring dockerfile: 407B                                                                                                                                   0.0s
 => [internal] load .dockerignore                                                                                                                                      0.0s
 => => transferring context: 2B                                                                                                                                        0.0s
 => [internal] load metadata for docker.io/library/cluster-base:latest                                                                                                 0.0s
 => [1/3] FROM docker.io/library/cluster-base                                                                                                                          0.0s
 => CACHED [2/3] RUN apt-get update -y &&     apt-get install -y python3-pip &&     pip3 install wget pyspark==3.0.0 jupyterlab==2.1.5                                 0.0s
 => CACHED [3/3] WORKDIR /opt/workspace                                                                                                                                0.0s
 => exporting to image                                                                                                                                                 0.0s
 => => exporting layers                                                                                                                                                0.0s
 => => writing image sha256:xxxxxxxxxxxxxxxxxxxxxx                                                                           0.0s
 => => naming to docker.io/library/jupyterlab                                                                                                                          0.0s

Use 'docker scan' to run Snyk tests against images to find vulnerabilities and learn how to fix them

$ docker images       
REPOSITORY     TAG       IMAGE ID       CREATED          SIZE
jupyterlab     latest    xxxxxxxxxxxx   53 minutes ago   1.43GB
spark-worker   latest    xxxxxxxxxxxx   54 minutes ago   483MB
spark-base     latest    xxxxxxxxxxxx   54 minutes ago   483MB
spark-master   latest    xxxxxxxxxxxx   54 minutes ago   483MB
cluster-base   latest    xxxxxxxxxxxx   55 minutes ago   217MB
```

#### STEP 6) docker compose up

아래와 같이 명령어를 실행하여 docker compose를 구동한다.

```bash
$ docker-compose up -d
[+] Running 6/6
 ⠿ Network spark_test_default               Created                                                                                                                    0.0s
 ⠿ Volume "hadoop-distributed-file-system"  Created                                                                                                                    0.0s
 ⠿ Container spark-master                   Started                                                                                                                    0.7s
 ⠿ Container jupyterlab                     Started                                                                                                                    0.7s
 ⠿ Container spark-worker-2                 Started                                                                                                                    1.4s
 ⠿ Container spark-worker-1                 Started                                                                                                                    1.4s

$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                                                      NAMES
xxxxxxxxxxxx   spark-worker   "/bin/sh -c 'bin/spa…"   12 seconds ago   Up 10 seconds   0.0.0.0:4041->8081/tcp                                     spark-worker-1
xxxxxxxxxxxx   spark-worker   "/bin/sh -c 'bin/spa…"   12 seconds ago   Up 10 seconds   0.0.0.0:4042->8081/tcp                                     spark-worker-2
xxxxxxxxxxxx   spark-master   "/bin/sh -c 'bin/spa…"   12 seconds ago   Up 11 seconds   7077/tcp, 0.0.0.0:8077->8077/tcp, 0.0.0.0:4040->8080/tcp   spark-master
xxxxxxxxxxxx   jupyterlab     "/bin/sh -c 'jupyter…"   12 seconds ago   Up 11 seconds   0.0.0.0:8888->8888/tcp                                     jupyterlab
```

반대로 docker compose를 내리고 싶으면 `docker compose down` 명령어를 실행하여 종료
```bash
$ docker compose down 
[+] Running 5/5
 ⠿ Container spark-worker-1    Removed                                                                                                                                10.5s
 ⠿ Container spark-worker-2    Removed                                                                                                                                10.6s
 ⠿ Container jupyterlab        Removed                                                                                                                                10.7s
 ⠿ Container spark-master      Removed                                                                                                                                10.3s
 ⠿ Network spark_test_default  Removed                                                                                                                                 0.1s
```


#### STEP 7) Spark 관련 UI 접속해보기
아래의 링크를 통해서 jupyterlab 과 열어놓은 spark web ui 들을 볼 수 있으며, spark master 웹 UI 를 보면 worker 들이 잘 등록 되어있는 것을 확인할 수 있다.
- jupyterlab: http://localhost:8888
- 마스터 노드: http://localhost:4040
- 워커 노드 1: http://localhost:4041
- 워커 노드 2: http://localhost:4042

<img width="1670" alt="1" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/8421176a-bfdd-46e0-a7f1-7f76195ffbcd">

<img width="1669" alt="2" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/2814a978-b98d-4f8d-901e-f40bd6826c51">

<img width="1638" alt="3" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/e242b90d-96b5-4b8b-a8e5-7d8e7d4b46b1">

<img width="1614" alt="4" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/53440607-3d5f-483f-8dd2-e1cedf070c3f">

#### STEP 8) Spark 실행 테스트

jupyterlab에서 아래에 코드를 입력하여 실행해본다.

```python
from pyspark.sql import SparkSession

spark = SparkSession.\
        builder.\
        appName("pyspark-notebook").\
        master("spark://spark-master:7077").\
        config("spark.executor.memory", "512m").\
        getOrCreate()
        
import wget

url = "https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data"
wget.download(url)

data = spark.read.csv("iris.data")
data.show(n=5)
```

<img width="1666" alt="5" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/43327cc0-aec1-41b8-9a02-cfcf98593173">

<img width="1659" alt="6" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/0a7322dd-00e5-4979-bf23-470957ad3cb7">

<img width="1648" alt="7" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/1806540d-bfa1-48e8-a78f-9bace43d4c33">

<img width="1665" alt="8" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/a9bdccf5-8b2f-4f29-98d9-94591a529e81">