.

Data_Engineering_TIL(20230528)

### Youtube "데브시스터즈 - 게임의 성공을 위한 Scalable 한 데이터 플랫폼 사례" 자료를 공부하고 정리한 내용입니다.

** URL : https://youtu.be/k72umNnHHYo

## [목차]

1 ) 데브시스터즈의 데이터플랫폼 소개

2 ) 분석환경의 Scalability 높이기

3 ) 데이터 인프라의 Scalability 높이기

4 ) scalable한 데이터플랫폼으로 게임의성공에 직접적으로 기여하기

** 두가지 Scalability 관점으로 이야기함

1 ) 인프라/서비스의 Scalability 확보 관점 n00+배의 트래픽에 대응하기 위한 아키틱처

2 ) 사람의 Scalability 확보 관점 사람이 가장 scalable 하지 않은 요소

## [공부한 내용]

### 1 ) 데브시스터즈의 데이터플랫폼 소개

#### 사람의 Scalability 를 확보하기 위한 노력

인력이 scalability를 결정하는데 가장 중요한 요소중에 하나지만 사람을 뽑아서 조직이나 플랫폼의 규모를 키우는게 제일 힘들다.

또한 반복작업이나 잡일을 줄여보려는 관점에서 기술적으로나 문화적으로 개선을 해보려고 노력을 했다.

결론적으로 소수의 인원으로 scalable한 플랫폼을 운영할 수 있도록 노력하였다.

#### 게임회사에서 데이터 플랫폼이란

게임서버의 로그, DB, 클라이언트, 서드파티 등으로부터 다양한 데이터를 수집하고, 활용가능하도록 적절하게 데이터를 가공한 다음에 다양한 니즈에 맞게 활용할 수 있는 방법을 제공하여 게임의 개발과 운영, 성장에 필수적인 다양한 문제를 해결하는 플랫폼을 말한다.

### 데이터 플랫폼 오버뷰

<img width="815" alt="1" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/ff32012a-aca3-48d8-8621-1dd8c2fc2bf4">

** 일라스틱서치를 이용해서 로그 검색을 하고 있다.

### 2 ) 분석환경의 Scalability 높이기

#### 레거시 분석환경 플랫폼 (2018년)

<img width="1174" alt="2" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/b7b5092a-cf1a-4180-a4a1-043a04947754">

카프카에서 로그를 수집해서 용도에 따른 s3 버킷에 적재하고, 또한 데이터 수명주기 관리를 위해 버킷 티어릴도 활용했다.

- Kafka 로 로그 수집

- Spark (Scala) 을 통해 전처리해 S3 적재 (Parquet)

- PySpark 을 통해 Data Warehouse, Data Mart Table 적재 (Parquet)

- Athena 로 Parquet Table 을 쿼리하여 지표 대시보드에 사용

- Airflow & Spark Job Scheduling

#### Python/Spark 기반 분석 환경의 한계

<img width="1209" alt="3" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/381df027-62f4-4f50-bc7a-56c8848b633a">

결론적으로는 비지니스 환경이 복잡해져서 BI, DW(Data Gorvenance) 엔지니어들이 같은 팀으로 합류하였는데 이들이 SQL은 잘하는데 파이썬과 스팍에 익숙하지가 않은 문제가 발생했다.

그래서 매번 배포를 할때마다 데이터 플랫폼 팀에서 번거롭게 다 개입을 하였다. 서로 답답한 현상이 발생했다. BI, DW 엔지니어들 본인들은 직접 개발해서 직접 배포까지 하고 싶은데 그게 안되어서 답답하고, 데이터 플랫폼팀 입장에서도 불필요하게 개입을 하는 비효율성이 발생한것이다.

#### 해결방안 1: Spark SQL 기반으로 SQL 분석 환경을 지원

<img width="1178" alt="4" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/a86a5d6e-7e60-488d-957e-69c5f2516f0b">

<img width="1168" alt="5" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/94ee4ce7-a131-49f2-a03e-ffb2f84ad0e6">

--> 그래서 BI, DW 엔지니어들이 파이썬에 너무 종속되지 않고 최대한 SQL을 활용하도록 지원하였음.

<img width="1025" alt="6" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/c194b09d-cdd5-42e9-a995-cad6ba362f51">

그러나 아래와 같이 여전히 아쉬운 부분들이 있었다.

<img width="1167" alt="7" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/ac586e4e-d11b-4a88-b7e7-8483a97d418a">

결론적으로 spark SQL 특성상 기존에 DW 솔루션들에서 지원하는 기능들을 일부 있었고, 파케이 특성상 데이터 레코드의 업데이트나 삭제가 불가하다는 점이 한계점이었다.

#### 그래서 두번째 해결방안으로 다시 개선을 시도하였다.

#### 해결방안 2: Delta Lake 기반의 Data Lakehouse 도입

<img width="1174" alt="8" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/a76c04ee-6783-4919-bacb-344b962d4af8">

델타 레이크 솔루션(일종에 DW 솔루션)을 사용하면 파케이 파일 외에 별도로 델타 로그 파일이 생성하여 체크포인트를 관리할 수 있기 때문에 ACID Transactions 도 지원하고, upsert나 delete도 효율적으로 수행이 가능해짐.

또한 메타데이터 관리 API를 제공하기 때문에 최적화 작업에도 유용하다.

예를 들어서 파케이로 데이터를 적재할 경우 작은 file들이 아주 많이 생기는데 file들이 특정용량 이상이 되면 이 file들은 전부 합쳐버리는 작업 (compaction) 등  최적화 작업을 SQL 구문으로 손쉽게 할 수 있는 장점이 있다.

<img width="1235" alt="9" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/da828c03-e2a4-4290-9d97-0c0f7dea0c5b">

결론적으로 파이썬을 잘몰라도 편하게 SQL로 분석할 수 있는 솔루션을 제공하여 비효율성을 극복하였다.

### 3 ) 데이터 인프라의 Scalability 높이기

#### DB 데이터 수집 파이프라인

<img width="1189" alt="10" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/be289370-ad49-4f81-8522-962445fb5aa1">

RDB 같은 경우에는 데일리 스냅샷만 수집하고 있었음.(디비 데이터 분석의 니즈가 많지 않았기 때문임.)

오로라 디비도 있었는데 이거는 S3 parquet export 기능을 이용해서 데이터를 수집했었다.

Cockroach DB 는 메인 게임서버 디비여서 대용량 디비였기 때문에 스팍을 이용해서 분산처리해서 데이터를 수집하였음.

#### 로그 데이터 수집 파이프라인

<img width="1137" alt="11" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/ba364740-2167-4cd3-9f0e-dad596a0ba53">

```text
카프카를 이용해서 데이터를 수집&전처리 --> 일라스틱 스텍을 기반으로 실시간으로 로그를 검색하는 서비스를 운영하는 파트
                             --> spark을 이용해서 로그를 적재 --> Data Warehouse 로 이어지는 파트
```

오늘은 로그 수집부분에서 이야기를 해보고자 함.

#### 로그 수집의 scalability 올리기

<img width="1111" alt="12" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/735fc851-478d-4082-918e-fd23f2cfc3a1">

쿠키런을 초창기에 런칭하고 운영을 시작을 했는데 카프카 디스크가 꽉차서 프러덕션 환경 장애가 발생했었음.

#### 위와 같이 장애가 발생하지 않도록 로그 수집의 Scalability 올리기 위한 노력

- Kafka의 장애가 서비스에 직접적 영향이 없고 신뢰 가능한 구조로 관리하려고 노력하였다.
   --> 이를 위해 서비스와의 decoupling 이 필요했다.

- 각각의 로그 생산/사용 주체가 Kafka 에 대한 지식 없이 쓸 수 있도록 하였다.
   --> 이 의미는 로그 생산자에게 Kafka 의 interface 를 노출시키지 않아야 했다.

- 최소한의 로그 퀄리티가 모든 파이프라인에 걸쳐 유지될 수 있도록 자동화된 검수와 전처리를 적용하고자 했다.(로그 데이터의 퀄리티를 보장하기 위한 노력)

#### 해결방안 1.: Agent 기반의 수집 구조 도입

<img width="1094" alt="13" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/bceb4906-54b0-4957-ab6a-f952ab4b6931">

이렇게 하면 로그의 유실 가능성은 발생하나 서비스에는 영향이 없는 장점이 있다.

#### 해결방안 2: Kafka streams 기반의 전처리/검수

<img width="1104" alt="14" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/1d531129-d914-4227-9119-42b63ef9d19c">

Kafka streams을 이용해서 로그 생산자가 생산한 로그를 그대로 사용하지 않고 전처리해서 사용했다.

로그 생산자가 로그를 카프카로 쏘게 되면 Raw Log Topics 라는 토픽에 먼저 저장하고 이 데이터를 카프카 스트림즈 어플리케이션을 이용해서 전처리해서 사용했다.

또한 스키마 스토어를 운영해서 스토어에서 스키마를 가져와서 검수하도록 자동화 하였다.

##### 해결방안 3: 쓰기용/ 읽기용 Kafka 클러스터 분리

<img width="1028" alt="15" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/a3133577-adbb-45b8-a226-28d8e9748c61">

카프카가 서비스와 바로 연결이 되어 있다보니까 예를 들어서 스팍으로 대규모 배치 작업을 진행하면 카프카의 cpu가 올라가거나 네트워크에 부하가 발생하는 등의 문제가 발생했다. 이럴 경우 로그 생산자가 로그를 브로거로 못보내는 현상이 발생할 수 있기 때문에 카프카 클러스터를 두개의 티어로 쪼갰다. 그래서 로그 생산자가 로그를 쓰는 클러스터, 그리고 소비자가 소비하는 클러스터로 구분했다. 그리고 중간에는 카프카 클러스터간에 미러링이 가능하도록 역할을 수행하는 어플리케이션 서버를 두었다.

#### "숙제를 미룬다고 사라지지 않는다."

<img width="1202" alt="16" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/60260ada-5cd2-4b0e-a5e5-c49ac50e1dd0">

--> 데이터가 들어올때 지연이 있으면 안되고 바로바로 처리를 해야한다는 의미인데 초창기에는 데이터가 들어올때 바로바로 처리를 못하는 리소스가 딸리는 이런 문제가 발생했었다. 결론적으로 데이터 파이프라인이 못버틴거였다.

#### 이에 대한 해결방안 : 컴포넌트별 특성에 따른 트래픽 급증 대응 정책

Kafka (Producer-faced) 를 신뢰성 있는 Buffer 용도로 사용했다.

--> 일단 생산자가 보내주는 데이터는 어떠한 경우로 던지 다 받아주자. (신뢰성 있는 버퍼로 활용)

이후 단계 어플리케이션은 빠르게 n 배 확장해서 처리하겨나 (예: Kafka Streams) 또는 이렇게 확장이 불가능한 경우라면 사전 (혹은 이슈 발생에 따라) SLA 들 기반으로 데이터 수집을 지연시키거나 또는 버리게 정책을 수립했다.


#### 기타 데브시스터즈의 데이터 인프라 특징 : storage 와 Compute 의 분리

• 데이터 인프라에서 대부분의 컴포넌트가 Stateful

• 최대한 Storage 와 Compute 가 분리가능한 구조를 사용하도록 노력함.

분리 정도에 따라 인프라의 Scalability 가 상당부분 결정됨

(거의) 완전히 분리 가능 - 예: Apache Spark + Amazon S3

제한적으로 확장 가능 - 예: Apache Kafka (AwS EC2 / EBS storage)

인스턴스 단위로만 확장 가능 - 예: Elasticsearch (AWS EC2 / Local storage)

#### 기타 데브시스터즈의 데이터 인프라 특징 : 데이터 인프라 운영의 Scalability 올리기 위해 필요한 사항들

• Stateless 의 경우는 이미 Autoscaling 그룹 등으로 운영 자동화했지만, stateful Application 들은 제약이 많음

• Stateful Application 특성상 in-place upgrade / 작업이 빈번

• 작업자의 맥락인지 비용(과거에 어떤 작업을 왜 했었는지 히스토리를 잘 알고 있어야 하는 비용)이 높음

#### 이를 극복하기 위한 해결방안 : Terraform 과 Ansible 을 이용한 lac

코드가 진실의 근원이 되어, 엔지니어의 작업 히스토리 팔로업 / 변화사항 인지비용을 최소화할 수 있도록 대부분의 EC2 기반 Stateful Application 에서 아래와 같이 업무를 수행함.

• (Terraform) Auto-scaling group 의 기능을 제한적으로 활용/정의

• (Terraform) Instance userdata 기반 Provisioning 자동화 (service discovery, cluster join, ..)

• Rolling 하기 어려운 in-place 작업의 경우 Ansible 로 먼저 진행하고 Terraform 에 후반영하여 두개의 sync를 맞추었다.

• Custom Resource 들은 custom terraform provider / in-house 스크립트로 해결


### 4 ) scalable한 데이터플랫폼으로 게임의성공에 직접적으로 기여하기

#### 데이터 플랫폼의 실전운영 결과

<img width="1106" alt="17" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/7262f051-5646-480c-a2dd-27e354b9d746">

<img width="1175" alt="18" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/7b5d2914-200e-44bd-8145-b1e0bfd63772">

<img width="1203" alt="19" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/a5eb5263-e55c-4a61-a3cc-914e8ce2d653">

<img width="1091" alt="20" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/2d9292c1-77fd-4cff-8736-8bbfc50382a8">

<img width="1200" alt="21" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/87f6b17c-2123-4247-8c54-21afddffa141">

위에 예시는 게임 데이터를 분석가가 분석해서 게임 난이도별로 특정 스테이징 부분에서 유저 이탈률이 많은 현상을 확인해서 난이도를 조정하였고, 이를 통해 유저의 이탈률을 막은사례

<img width="1174" alt="22" src="https://github.com/minman2115/Data_engineering_studynotes_2023/assets/41605276/3825ddca-b31f-401b-8e6c-b4ba1b79d076">

쿠키런 게임하는 사람들을 킹덤 게임도 하도록 유도할 수 있지 않을까?? 게임 로그데이터를 활용하면 프로모션 하는것도 가능하지 않을까?? 에 관한 사례