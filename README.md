# ElasticStack 우분투 환경에서 설치   

yoonje님의 아래 레포지토리에서 'tuto'파일을 https://github.com/gjlee0802/ElasticStackStudy/blob/main/tuto 로 대체하여 수행.   
yoonje님의 설치 튜토리얼: https://github.com/yoonje/elastic-stack-tutorial   
**tuto에서 패키지 다운로드 주소 수정, 우분투 환경에 맞게 수정**

# ElasticStackStudy1

## ELK구성요소(Elasticsearch, Logstash, Kibana)

### Elasticsearch (데이터베이스)
Logstash로부터 받은 데이터를 검색 및 집계를 하여 필요한 관심 있는 정보를 획득.   
Lucene 검색엔진 기반의 데이터베이스로 고성능의 검색 기능, 대규모 분산 시스템 기능 등을 제공한다.   
표준 **RESTful API와 JSON을 이용**해 데이터를 처리합니다.  

### Logstash (Input > filter > output 의 pipeline 구조로 수집하고 필터링하여 전달)
다양한 소스( DB, csv파일 등 )의 로그 또는 트랜잭션 데이터를 수집, 집계, 파싱하여 Elasticsearch로 전달   
데이터 수집 파이프라인 도구입니다.   
각 데이터베이스의 데이터, 로우 데이터, 윈도우 이벤트 등으로부터 데이터를 수집합니다.   
다양한 플러그인을 이용하여 데이터를 집계 및 보관, 서버 데이터 처리, 파이프라인으로 데이터를 수집하여 필터를 통해 변환 후 ElasticSearch로 전송합니다.    

### Kibana (시각화 담당)
Elasticsearch의 빠른 검색을 통해 데이터를 시각화 및 모니터링   
Elastic Search에 저장된 정보들을 검색 및 분석하고 시각화하는 기능을 갖고 있습니다.   

### Beats (데이터 전송 담당)
경량 에이전트로 설치되어 데이터를 Logstash 또는 ElasticSearch로 전송합니다.   
1) FileBeat   
- 서버에서 로그파일을 제공합니다.   
2) PacketBeat   
- 응용 프로그램 서버간에 교환되는 트랜잭션에 대한 정보를 제공하는 네트워크 패킷 분석기 입니다.   
3) MetricBeat   
- 운영 체제 및 서비스에서 Metrics를 주기적으로 수집하는 서버 모니터링 에이전트입니다.   
4) WinlogBeat   
- Windows 이벤트 로그를 제공합니다.   

## RDBMS와 Elasticsearch비교
![RDBMS_Elastic](./img/RDBMS대응.jpg)
![RDBMS_Elastic_](./img/RDBMS대응_.png)


## Elasticsearch 용어 정리
### 1) 클러스터( Cluster )
클러스터란 Elasticsearch에서 가장 큰 시스템 단위를 의미하며, 최소 하나 이상의 노드로 이루어진 노드들의 집합입니다.   
서로 다른 클러스터는 데이터의 접근, 교환을 할 수 없는 독립적인 시스템으로 유지되며,   
여러 대의 서버가 하나의 클러스터를 구성할 수 있고, 한 서버에 여러 개의 클러스터가 존재할수도 있습니다.   

### 2) 노드( Node )
Elasticsearch를 구성하는 하나의 단위 프로세스를 의미합니다.   
그 역할에 따라 Master-eligible, Data, Ingest, Tribe 노드로 구분할 수 있습니다.   
- 클러스터의 일부이며 데이터를 저장하고 클러스터의 인덱싱 및 검색 기능에 참여하는 단일 서버입니다.   
- 노드에 할당되는 임의 UUID인 이름으로 식별합니다.   
- 특정 클러스터를 클러스터 이름으로 결합하도록 노드를 구성 할 수 있습니다.   

### 3) 인덱스( Index ) / 샤드( Shard ) / 복제( Replica )
 index는 RDBMS에서 database와 대응하는 개념입니다.   
또한 shard와 replica는 Elasticsearch에만 존재하는 개념이 아니라, 분산 데이터베이스 시스템에도 존재하는 개념입니다.   

인덱스   
- 색인 과정을 거친 결과물, 또는 색인된 데이터가 저장되는 저장소입니다.
- 다소 유사한 특성을 갖는 문서들의 집합입니다.   

샤드   
- Index는 잠재적으로 단일 노드의 하드웨어 제한을 초과 할 수 있는 많은 양의 데이터를 저장 할 수 있습니다. 하지만 단일 노드의 디스크가 맞지 않거나 단일 노드의 검색 요청만 처리하기에는 너무 느릴 수 있기 때문에 shards를 이용하여 Index를 여러 조각으로 나눌 수 있습니다.   
- 수평적으로 콘텐츠 볼륨을 split/scale 가능합니다.   
- 여러 노드에서 잠재적으로 분산을 통해 작업을 분산 및 병렬 처리를 할 수 있으므로 성능/처리량이 향상됩니다.   

-> 샤딩과 파티셔닝 차이
- 샤딩 : 여러시스템에 나누어 저장
- 파티셔닝 : 한 시스템에서 여러 테이블로 저장   

Shard와 Replica 개념 참고1 : https://jiseok-woo.tistory.com/8   
Shard와 Replica 개념 참고2 : http://guruble.com/elasticsearch-2-shard-replica/   

복제   
- 장애가 발생할 경우 고가용성을 제공합니다. 그렇기 때문에 복제본 샤드는 복사된 원본/기본 샤드와 동일한 노드에 할당되지 않습니다.   
- 모든 복제본에서 검색을 병렬로 실행할 수 있기 때문에 검색 볼륨/처리량을 수평 확장 할 수 있습니다.   
- 기본적으로 각 인덱스는 4개의 기본 샤드와 1개의 복제본이 할당됩니다.   
- 또 다른 형태의 shard라고 할 수 있습니다. 노드를 손실했을 경우 데이터의 신뢰성을 위해 샤드들을 복제하는 것이죠. 따라서 replica는 서로 다른 노드에 존재할 것을 권장합니다.   

->Replica가 필요한 이유 2가지   
- 1. ‘검색성능(search performance)‘   
- 2. ‘장애복구(fail-over)‘    

![ElasticsearchArch](./img/ElasticSearchArch.png)

### 4) 문서( Document )
JSON 형태의 실제 의미있는 데이터를 가진 Elasticsearch 기본 저장단위   
RDB의 row, 레코드와 비슷.   
문서는 각각 고유한 ID 값을 갖는다   
고유한 ID:   
- 사용자 지정값 or 랜덤값   
- 데이터를 찾아가는 Meta key 역할   


## Elasticsearch 특징
### Scale out
- 샤드를 통해 규모가 수평적으로 늘어날 수 있음

### 고가용성
- Replica를 통해 데이터의 안정성을 보장
![Replica](./img/replica.PNG)

### Schema Free
- Json 문서를 통해 데이터 검색을 수행하므로 스키마 개념이 없음

### Restful
- 데이터 CURD 작업은 HTTP Restful API를 통해 수행한다.
![restful](./img/restful.PNG)

### 멀티테넌시 (multitenancy)   
데이터들은 인덱스(Index) 라는 논리적인 집합 단위로 구성되며 서로 다른 저장소에 분산되어 저장됩니다.   
서로 다른 인덱스들을 별도의 커넥션 없이 하나의 질의로 묶어서 검색하고, 검색 결과들을 하나의 출력으로 도출할 수 특징.   

### 역색인(inverted index) -> ElasticSearch가 빠른 이유
   
**RDB의 경우**   
![RDBsearch](./img/RDB_likesearch.png)
like 검색: 
fox가 포함된 행들을 가져온다고 하면 다음과 같이 Text 열을 한 줄씩 찾아 내려가면서 fox가 있으면 가져오고 없으면 넘어가는 식으로 데이터를 가져옵니다   
**ElasticSearch의 경우**   
![inverted_index](./img/inverted_index.png)   
Inverted Index: 
책의 맨 뒤에 있는 주요 키워드에 대한 내용이 몇 페이지에 있는지 볼 수 있는 찾아보기 페이지에 비유할 수 있습니다.   
Elasticsearch에서는 추출된 각 키워드를 텀(term) 이라고 부릅니다.    
이렇게 역 인덱스가 있으면 fox를 포함하고 있는 도큐먼트들의 id를 바로 얻어올 수 있습니다.   

### 텍스트 분석(Text Analysis) 
Elasticsearch의 애널라이저는 0-3개의 캐릭터 필터(Character Filter)와 1개의 토크나이저(Tokenizer), 그리고 0-n개의 토큰 필터(Token Filter)로 이루어집니다.
![Analyzer](./img/text_analyzer.png)   
**캐릭터 필터: 전체 문장에서 특정 문자를 대치하거나 제거하는데 이 과정을 담당하는 기능.**   
**토크나이저: 문장에 속한 단어들을 텀 단위로 하나씩 분리 해 내는 처리 과정을 거치는데 이 과정을 담당하는 기능.(반드시 1개만 적용이 가능합니다.)**   
**lowercase 토큰필터: 대문자를 모두 소문자로 바꿔줍니다.**   
**stop 토큰필터: 불용어(stopword, 검색어로서의 가치가 없는 단어)를 제거.**   
**snowball 토큰필터: 형태소 분석 과정을 거쳐서 문법상 변형된 단어를 일반적으로 검색에 쓰이는 기본 형태로 변환.(jumps와 jumping은 모두 jump로 변경)**   




출처 :    
https://victorydntmd.tistory.com/308    
https://iassad.tistory.com/7   
https://heowc.tistory.com/49   
https://esbook.kimjmin.net/06-text-analysis/6.1-indexing-data
### 

# ElasticStackStudy2

## ElasticSearch Data 색인
- [동사] 색인 (indexing) : 데이터가 검색될 수 있는 구조로 변경하기 위해 원본 문서를 검색어 토큰들으로 변환하여 저장하는 일련의 과정입니다.   
- [명사] 인덱스 (index, indices) : 색인 과정을 거친 결과물, 또는 색인된 데이터가 저장되는 저장소입니다. 또한 Elasticsearch에서 도큐먼트들의 논리적인 집합을 표현하는 단위이기도 합니다.   
- 검색 (search) : 인덱스에 들어있는 검색어 토큰들을 포함하고 있는 문서를 찾아가는 과정입니다.   
- 질의 (query) : 사용자가 원하는 문서를 찾거나 집계 결과를 출력하기 위해 검색 시 입력하는 검색어 또는 검색 조건입니다.    
![Indexing](./img/indexing.png)   

## ElasticSearch 실행 옵션   
- -d : Elasticsearch를 백그라운 데몬으로 실행합니다.   
- -p <파일명> : Elasticsearch 프로세스 ID를 지정한 파일에 저장합니다. 실행이 종료되면 저장된 파일은 자동으로 삭제됩니다.   

## ElasticSearch 실행 환경 설정방법 2가지   
- 홈 디렉토리의 config 경로 아래 있는 파일들을 변경   
  - jvm.options - Java 힙메모리 및 환경변수 (Elasticsearch는 Java의 가상머신 위에서 실행이 되는데 7.0 기준으로 1gb의 힙메모리가 기본으로 설정되어 있습니다.)   
  - elasticsearch.yml - Elasticsearch 옵션 (elasticsearch 실행 환경에 대한 실제 설정들은 대부분 elasticsearch.yml 파일에서 설정)   
  - log4j2.properties - 로그 관련 옵션   
- 시작 명령으로 설정하는 방법   
  - Elasticsearch 를 처음 실행할 때 -E 커맨드 라인 명령을 통해서도 가능합니다.   
  ### elasticsearch.yml 설정
  - cluster.name: "<클러스터명>"   
  - node.name: "<노드명>"   
  - node.attr.<key>: "<value>"   
  - path.data: [ "<경로>" ]   
  - path.logs: "<경로>"   
  - bootstrap.memory_lock: true   
  - network.host: <ip 주소>   
  - http.port: <포트 번호>   
  - transport.port: <포트 번호>   
  - discovery.seed_hosts: [ "<호스트-1>", "<호스트-2>", ... ]   
  - cluster.initial_master_nodes: [ "<노드-1>", "<노드-2>" ]   
  ### 시작 명령으로 설정   
    클러스터명은 my-cluster 노드명은 node-1로 노드를 실행하기 위해서는 다음과 같이 실행.   
  - $ bin/elasticsearch -E cluster.name=my-cluster -E node.name="node-1"   
 
## 클러스터 구성
Elasticsearch의 노드들은 클라이언트와의 통신을 위한 http 포트(9200-9299), 노드 간의 데이터 교환을 위한 tcp 포트 (9300-9399) 총 2개의 네트워크 통신을 열어두고 있습니다.   
일반적으로 1개의 물리 서버마다 하나의 노드를 실행하는 것을 권장하고 있습니다.   
### 여러 서버에 하나의 클러스터로 실행   
- 3개의 다른 물리 서버에서 각각 1개 씩의 노드를 실행하면   
![cluster_1](./img/cluster_1.png)   
- 서버1 에서 두개의 노드를 실행하고, 또 다른 서버에서 한개의 노드를 실행   
![cluster_2](./img/cluster_2.png)   
### 하나의 서버에서 여러 클러스터 실행   
- 하나의 물리 서버에서 서로 다른 두 개의 클러스터 실행   
![cluster_3](./img/cluster_3.png)   

출처: https://esbook.kimjmin.net/03-cluster/3.1-cluster-settings   

### 디스커버리 (Discovery)
노드가 처음 실행 될 때 같은 서버, 또는 discovery.seed_hosts: [ ] 에 설정된 네트워크 상의 다른 노드들을 찾아 하나의 클러스터로 바인딩 하는 과정   
 순서   
 1. discovery.seed_hosts 설정에 있는 주소 순서대로 노드가 있는지 여부를 확인   
  - 노드가 존재하는 경우 > cluster.name 확인   
     - 일치하는 경우 > 같은 클러스터로 바인딩 > 종료   
     - 일치하지 않는 경우 > 1로 돌아가서 다음 주소 확인 반복   
  - 노드가 존재하지 않는 경우 > 1로 돌아가서 다음 주소 확인 반복   
 2. 주소가 끝날 때 까지 노드를 찾지 못한 경우   
  - 스스로 새로운 클러스터 시작   
![discovery](./img/discovery.png)

## 인덱스와 샤드 - Index & Shards
- 인덱스는 기본적으로 샤드(shard)라는 단위로 분리되고 각 노드에 분산되어 저장이 됩니다.
- 같은 샤드와 복제본은 동일한 데이터를 담고 있으며 반드시 서로 다른 노드에 저장이 됩니다.
- 샤드의 개수는 인덱스를 처음 생성할 때 지정할 수 있습니다. 
- 프라이머리 샤드 수는 인덱스를 처음 생성할 때 지정하며, 인덱스를 재색인 하지 않는 이상 바꿀 수 없습니다. 
- 복제본의 개수는 나중에 변경이 가능합니다.

## 마스터 노드와 데이터 노드 - Master & Data
### 마스터 노드 (Master Node)
하나의 노드는 인덱스의 메타 데이터, 샤드의 위치와 같은 클러스터 상태(Cluster Status) 정보를 관리하는 마스터 노드의 역할을 수행합니다.   
클러스터마다 반드시 하나의 마스터 노드가 존재합니다.   

__(elasticsearch.yml)__
- 디폴트 설정은 node.master: true로 되어 있습니다. ( 모든 노드가 마스터 노드로 선출될 수 있는 마스터 후보 노드 )
- 클러스터가 커져서 노드와 샤드들의 개수가 많아지게 되면 모든 노드들이 마스터 노드의 정보를 계속 공유하는 것은 부담이 될 수 있습니다. 
      마스터 노드로 사용하지 않는 노드들은 설정값을 node.master: false 로 하여 마스터 노드의 역할을 하지 않도록 합니다.

### 데이터 노드 (Data Node)
실제로 색인된 데이터를 저장하고 있는 노드입니다.  

__(elasticsearch.yml)__
- 마스터 후보 노드들은 node.data: false 로 설정하여 마스터 노드 역할만 하고 데이터는 저장하지 않도록 할 수 있습니다.

### Split Brain 문제
마스터 후보 노드들은 3개 이상의 **홀수 개**를 놓는 것을 권장하고 있습니다. 만약에 마스터 후보 노드를 2개 혹은 짝수로 운영하는 경우 네트워크 유실로 인해 다음과 같은 상황을 겪을 수 있습니다.
![splitbrain](./img/splitbrain.png)   
**Split Brain이란**   
네트워크 단절로 마스터 후보 노드가 분리되면 각자가 서로 다른 클러스터로 구성되어 동작하다가   
네트워크가 복구 되고 하나의 클러스터로 다시 합쳐졌을 때 데이터 정합성에 문제가 생기고 데이터 무결성 손실되는 현상.   
   
**Split Brain 문제를 피하기**   
마스터 후보 노드 개수는 항상 홀수로 하고 가동을 위한 최소 마스터 후보 노드 설정은 (전체 마스터 후보 노드)/2+1 로 설정   
   
__(elasticsearch.yml)__   
7.0버전부터는 node.master: true 인 노드가 추가되면 클러스터가 스스로 minimum_master_nodes 노드 값을 변경하도록 되었습니다.   
사용자는 최초 마스터 후보로 선출할 cluster.initial_master_nodes: [ ] 값만 설정하면 됩니다.   

## 서버 이중화 구조(Active-Standby, Active-Active) 개념
### Active-Standby 구조
하나의 서버가 활성 상태로 서비스 중일 때 다른 하나는 대기를 한다.   
활성 서버에 장애가 발생해 서비스를 못하게 되면, 대기서버가 장애 발생을 감지하고 대기 서버에 모든 서비스를 올려서 서비스를 운영하는 구조이다.   
### Active-Active 구조
다중화 장비가 두대 모두 활성화 되어 동작하는 구성이다. 두대가 모두 처리를 하기 때문에 Active-Standby에 비하여 처리률이 높지만, 보통 설정 및 구성이 복잡해지며 한대가 죽었을 경우, 두대의 처리량을 한대로 처리하기 때문에 리소스가 높아져 리소스에 대한 계획도 필요하다. 

## 샤드와 성능 문제
Elasticsearch에서는 샤드당 단일 쓰레드가 각 쿼리를 실행합니다.
캐싱을 사용하지 않을 때, 최소 쿼리 응답 시간은 데이터, 쿼리 유형 및 샤드 크기에 따라 달라집니다.   
많은 개수의 작은 샤드를 조회하면 각 샤드마다 처리 속도는 빨라지지만 더 많은 작업을 큐에 넣고 순서대로 처리해야하므로, 더 적은 개수의 큰 샤드를 검색하는 것보다 반드시 빠르다고 보장할 수 없습니다.   
동시에 여러 쿼리가 실행될 때, 여러 개의 작은 샤드가 오히려 쿼리 처리량(throughput)을 줄일 수도 있겠죠.   
샤드의 수가 너무 적으면 단일 쿼리의 응답 속도는 느려질 수 있으나 대량의 쿼리가 진입될 경우 고른 성능을 보여줄 수 있고, 샤드의 수가 너무 많으면 단일 쿼리의 응답 속도는 빠를 수 있으나 대량의 쿼리가 인입될 경우 쿼리 별 성능 차이가 심해질 수 있습니다.   
출처: https://www.elastic.co/kr/blog/how-many-shards-should-i-have-in-my-elasticsearch-cluster   
      https://brunch.co.kr/@alden/39
      
      
# ElasticStackStudy3
RESTFul API : 항상 단일 URL로 접근을 하고 PUT, GET, DELETE 같은 http 메서드로 데이터를 처리합니다   
http://<호스트>:<포트>/<인덱스>/_doc/<도큐먼트 id>   
## CRUD API
- PUT **(Update)**
~~~
PUT my_index/_doc/1   
{   
  "name":"GyeongJoo Lee",   
  "message":"안녕하세요 Elasticsearch"   
}  
~~~
- GET **(Read)**
~~~
GET my_index/_doc/1   
~~~
- DELETE **(Delete)**
~~~
DELETE my_index/_doc/1   
~~~
- POST **(Create)**
POST 메서드는 PUT 메서드와 유사하게 데이터 입력에 사용이 가능합니다.   
도큐먼트를 입력할 때 POST 메서드로 <인덱스>/_doc 까지만 입력하게 되면 자동으로 임의의 도큐먼트id 가 생성됩니다. 도큐먼트id의 자동 생성은 PUT 메서드로는 동작하지 않습니다.   
~~~
POST my_index/_doc   
{   
  "name":"GyeongJoo Lee",   
  "message":"안녕하세요 Elasticsearch"   
}  
~~~

## 벌크 API
- 여러 명령을 배치로 수행하기 위해서 _bulk API의 사용이 가능합니다.   
_bulk API로 index, create, update, delete의 동작이 가능하며 delete를 제외하고는 명령문과 데이터문을 한 줄씩 순서대로 입해야 합니다.   
- 벌크 동작은 따로따로 수행하는 것 보다 속도가 훨씬 빠릅니다. 특히 대량의 데이터를 입력 할 때는 반드시 _bulk API를 사용해야 불필요한 오버헤드가 없습니다.   

다음 명령으로 __bulk.json__ 파일에 있는 내용들을 _bulk 명령으로 실행 가능합니다. 파일 이름 앞에는 @문자를 입력합니다.   
~~~
$ curl -XPOST "http://localhost:9200/_bulk" -H 'Content-Type: application/json' --data-binary @bulk.json
~~~
## 검색 API
Elasticsearch의 진가는 쿼리를 통한 검색 기능에 있습니다.   
검색은 인덱스 단위로 이루어집니다.   

### URI 검색
GET <인덱스명>/_search 형식으로 사용하며 쿼리를 입력하지 않으면 전체 도큐먼트를 찾는 match_all 검색을 합니다.   
- URI 검색으로 검색어 "value" 검색   
~~~
GET test/_search?q=value   
~~~
- URI 검색으로 검색어 "value AND three" 검색 (AND, OR, NOT)   
~~~
GET test/_search?q=value AND three   
~~~
- URI 검색으로 "field" 필드에서 검색어 "value" 검색 (검색어 value 을 field 필드에서 찾고 싶으면)   
~~~
GET test/_search?q=field:value   
~~~
### 데이터 본문 (Data Body) 검색
데이터 본문(data body) 검색은 검색 쿼리를 데이터 본문으로 입력하는 방식입니다.   
Elasticsearch의 QueryDSL을 사용하며 쿼리 또한 Json 형식으로 되어 있습니다.   
   
**match 쿼리**
쿼리 입력은 항상 query 지정자로 시작합니다. 
그 다음 레벨에서 쿼리의 종류를 지정하는데 위에서는 match 쿼리를 지정했습니다. 
그 다음은 사용할 쿼리 별로 문법이 상이할 수 있는데 match 쿼리는 <필드명>:<검색어> 방식으로 입력합니다.
데이터 본문 검색으로 "field" 필드에서 검색어 "value" 검색
~~~
GET test/_search   
{   
  "query": {   
    "match": {   
      "field": "value"   
    }   
  }   
}   
~~~
**멀티테넌시 (Multitenancy)**
여러 개의 인덱스를 한꺼번에 묶어서 검색할 수 있는 멀티테넌시를 지원합니다.   
- 쉼표로 나열해서 여러 인덱스 검색   
~~~
GET logs-2018-01,2018-02,2018-03/_search   
~~~
- 와일드카드 * 를 이용해서 여러 인덱스 검색   
~~~
GET logs-2018-*/_search   
~~~

## Query DSL (Domain Specific Language)   
Elasticsearch 는 검색을 위한 쿼리 기능을 제공합니다.   
이런 데이터 시스템에서 제공하는 쿼리 기능을 Query DSL (Domain Specific Language) 이라고 이야기 하며 Elasticsearch 의 Query DSL 은 모두 json 형식으로 입력해야 합니다.   

### - match all(별다른 조건 없이 해당 인덱스의 모든 도큐먼트를 검색하는 쿼리)   
~~~
GET my_index/_search   
~~~
와 아래는 동일   
~~~
GET my_index/_search   
{   
  "query":{   
    "match_all":{ }   
   }   
}   
~~~

### - match(풀 텍스트 검색에 사용되는 가장 일반적인 쿼리)   
#match 쿼리로 message 필드에서 dog 검색
~~~
GET my_index/_search
{   
  "query": {   
    "match": {   
      "message": "dog"   
    }   
  }   
}   
~~~
dog가 포함된 도큐먼트들이 검색 결과로 나타납니다.   
#match 쿼리로 message 필드에서 quick dog 검색   
match 검색에 여러 개의 검색어를 집어넣게 되면 디폴트로 OR 조건으로 검색이 되어 입력된 검색어 별로 하나라도 포함된 모든 문서를 모두 검색합니다.   
~~~
GET my_index/_search
{
  "query": {
    "match": {
      "message": "quick dog"
    }
  }
}
~~~
quick과 dog중 어떤 단어라도 포함한 도큐먼트들이 검색됩니다.   

#검색어가 여럿일 때 검색 조건을 OR 가 아닌 AND 로 바꾸려면 **operator 옵션**을 사용할 수 있습니다.   
quick dog 를 AND 조건으로 검색하려면 다음과 같습니다.   
~~~
GET my_index/_search
{
  "query": {
    "match": {
      "message": {
        "query": "quick dog",
        "operator": "and"
      }
    }
  }
}
~~~

### - match_phrase   

"quick dog" 라는 구문을 공백을 포함해 정확히 일치하는 내용을 검색하려면 match_phrase 쿼리를 사용합니다.   
#match_phrase 쿼리로 "lazy dog" 구문 검색   
~~~
GET my_index/_search
{
  "query": {
    "match_phrase": {
      "message": "lazy dog"
    }
  }
}
~~~
#**slop 이라는 옵션**을 이용하여 slop에 지정된 값 만큼 단어 사이에 다른 검색어가 끼어드는 것을 허용할 수 있습니다.   
match_phrase 쿼리에 slop:1 로 "lazy dog" 구문 검색
~~~
GET my_index/_search
{
  "query": {
    "match_phrase": {
      "message": {
        "query": "lazy dog",
        "slop": 1
      }
    }
  }
}
~~~
slop의 크기를 1로 했기 때문에 lazy와dog 사이에 jumping이 있는 "Lazy jumping dog" 값도 검색이 됩니다.   

### - query_string   
URL검색에 사용하는 루씬의 검색 문법을 본문 검색에 이용하고 싶을 때 query_string 쿼리를 사용할 수 있습니다.   
   
#message 필드에서 lazy와 jumping을 모두 포함하거나 또는 "quick dog" 구문을 포함하는 도큐먼트를 검색하는 쿼리입니다.   
match_phrase 처럼 구문 검색을 할 때는 검색할 구문을 쌍따옴표 \" 안에 넣습니다.   
~~~
GET my_index/_search
{
  "query": {
    "query_string": {
      "default_field": "message",
      "query": "(jumping AND lazy) OR \"quick dog\""
    }
  }
}
~~~
"Lazy jumping dog" 도큐먼트와 "quick dog" 값을 포함하는 도큐먼트 두개가 결과로 리턴된 것을 확인할 수 있습니다.   

## Bool 복합 쿼리
- must : 쿼리가 참인 도큐먼트들을 검색합니다. 
- must_not : 쿼리가 거짓인 도큐먼트들을 검색합니다. 
- should : 검색 결과 중 이 쿼리에 해당하는 도큐먼트의 점수를 높입니다. 
- filter : 쿼리가 참인 도큐먼트를 검색하지만 스코어를 계산하지 않습니다. must 보다 검색 속도가 빠르고 캐싱이 가능합니다.
   
사용문법   
~~~
GET <인덱스명>/_search
{
  "query": {
    "bool": {
      "must": [
        { <쿼리> }, …
      ],
      "must_not": [
        { <쿼리> }, …
      ],
      "should": [
        { <쿼리> }, …
      ],
      "filter": [
        { <쿼리> }, …
      ]
    }
  }
}
~~~
## 정확도
Elasticsearch 와 같은 풀 텍스트 검색엔진은 검색 결과가 입력된 검색 조건과 얼마나 정확하게 일치하는 지를 계산하는 알고리즘을 가지고 있어   
이 정확도를 기반으로 사용자가 가장 원하는 결과를 먼저 보여줄 수 있습니다. 이 정확한 정도를 relevancy 라고 합니다.   
### Score
- _score 항목에 스코어 점수가 표시되고 이 점수가 높은 결과부터 나타납니다.  
- max_score에는 전체 결과 중에서 가장 높은 점수가 표시됩니다.  
### TF (Term Frequency)   
도큐먼트 내에 검색된 텀(term)을 많이 포함할수록 점수가 높아지는 것   

### IDF (Inverse Document Frequency)   
전체 인덱스에 검색한 텀을 포함하고 있는 **도큐먼트 개수가 많을수록** 그 텀의 자신의 **점수가 감소**하는 것   

### Field Length
도큐먼트에서 필드 길이가 큰 필드 보다는 짧은 필드에 있는 텀의 비중이 클 것입니다.   
### Bool : Should
검색어 중 특정 검색어가 포함된 결과에 가중치를 줘서 상위로 올리고 싶은 경우 사용   

## 정확값 쿼리 (Exact Value Query)
검색 조건의 참 / 거짓 여부만 판별해서 결과를 가져오는 것입니다.
- bool : filter
filter는 검색에 조건은 추가하지만 스코어에는 영향을 주지 않도록 제어할 때 사용.
 - keyword
 문자열과 공백, 대소문자까지 정확히 일치하는 데이터만을 결과로 리턴합니다.   
 스코어를 계산하지 않기 때문에 keyword 값을 검색 할 때는 filter 구문 안에 넣도록 합니다.   
    
 ## 범위 쿼리 (Range Query)
 숫자, 날짜 형식은 range 쿼리를 이용해서 검색을 합니다.   
 'range : { <필드명>: { <파라메터>:<값> } }'   
 #price 값이 700 이상 900 미만인 데이터 검색   
 ~~~
 GET phones/_search
{
  "query": {
    "range": {
      "price": {
        "gte": 700,
        "lt": 900
      }
    }
  }
}
 ~~~
#date 값이 2016-01-01 ~ 2018-01-01 사이의 데이터 검색    
~~~
GET phones/_search
{
  "query": {
    "range": {
      "date": {
        "gt": "31/12/2015",
        "lt": "2018",
        "format": "dd/MM/yyyy||yyyy"
      }
    }
  }
}
~~~
## 데이터 색인과 텍스트 분석
### _analyze API   
분석된 문장을 _analyze API를 이용해서 확인할 수 있습니다.   
토크나이저는 tokenizer, 토큰 필터는 filter 항목의 값으로 입력하면 됩니다.   
토크나이저는 하나만 적용되기 때문에 바로 입력하고, 토큰필터는 여러개를 적용할 수 있기 때문에 [ ] 안에 배열 형식으로 입력합니다.   
~~~
GET _analyze
{
  "text": "The quick brown fox jumps over the lazy dog",
  "tokenizer": "whitespace",
  "filter": [
    "lowercase",
    "stop",
    "snowball"
  ]
}
~~~

- 여러 토큰 필터를 입력 할 때는 순서가 중요하며 **만약에 stop 토큰 필터를 lowercase 보다 먼저 놓게 되면** stop 토큰필터 처리시 **대문자로 시작하는 "The"는 불용어로 간주되지 않아 그냥 남아있게 됩니다.** 그 후에 lowercase가 적용되어 소문자 "the"가 최종 검색 텀으로 역 색인에 남아있게 됩니다.   

- whitespace 토크나이저 그리고 lowercase, stop, snowball 토큰필터들을 조합한 것이 **snowball 애널라이저** 입니다.   
snowball 애널라이저를 사용한 결과는 앞의 whitespace 토크나이저 그리고 lowercase, stop, snowball 토큰필터를 사용한 결과와 동일하게 나타납니다.   

- jumps, jumping을 검색하면 실제로는 jump로 바꾸어 검색이 됩니다.


# 키바나 실습
## 키바나 매니지먼트 (Kibana management)   
basketball 인덱스 추가   
~~~
$ curl -XPUT localhost:9200/basketball   
~~~
사용될 필드들의 data type을 지정해주어 시각화에 도움을 줌. (주의: ContentType 명시)   
~~~
$ curl -XPUT 'localhost:9200/basketball/record/_mappin' -d @basketball_mapping.json -H 'ContentType:application/json'   
~~~
bulk (json자료 한꺼번에 삽입)
~~~
$ curl -XPOST 'localhost:9200/_bulk' --data-binary @bulk_basketball.json   
~~~
basketball index 삭제   
~~~
$ curl -XDELETE localhost:9200/basketball   
~~~
   
## 키바나 디스커버 (Kibana discover)   
Discover 페이지: 특정 인덱스의 데이터를 확인할 수 있음.   
   
- Discover페이지
![discover1](./img/discover1.png)
   
- 더하기 모양 버튼을 클릭하여 해당 데이터와 일치하는 것들만 시각화하도록 필터를 추가할 수 있음.
![discover2](./img/discover2.png)   
   
- 상단에서 필터목록을 볼 수 있음.   
![discover3](./img/discover3.png)   
   
- 토글 버튼을 눌러 특정 데이터에 대한 정보들만 확인할 수 있음.   
![discover4](./img/discover4.png)   
   
## 키바나 비주얼라이즈 (Kibana Visualize)   
   
- Line Chart   
![Linechart](./img/visualize1.png)
   
- Metric   
![Metric](./img/visualize2.png)
   
- Dashboard
