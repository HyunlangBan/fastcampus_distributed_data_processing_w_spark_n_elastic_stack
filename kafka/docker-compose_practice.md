# docker-compose로 kafka, kafka-ui 띄워보기

- 참고
[https://devocean.sk.com/blog/techBoardDetail.do?ID=164016](https://devocean.sk.com/blog/techBoardDetail.do?ID=164016)

>→ Q. kafka를 여러개 띄울때 zookeeper도 여러개 띄워야하나?
>
> Tutorials generally keep things nice and simple, so one ZooKeeper (often one Kafka broker too). Useful for getting started; useless for any kind of resilience :)
> 
> 
> In practice, you are going to need three ZooKeeper nodes minimum.
> 
> [https://stackoverflow.com/questions/67623334/do-you-need-multiple-zookeeper-instances-to-run-a-multiple-broker-kafka](https://stackoverflow.com/questions/67623334/do-you-need-multiple-zookeeper-instances-to-run-a-multiple-broker-kafka)
> 
- 실제로 서비스를 운영할때는 zookeeper를 3개정도를 띄운다고 함
- 일단 난 zookeeper 1개, kafka 3개를 띄웠고 fakfa-ui까지 띄웠다.

### docker-compose.yml

```yml
version: '2'
services:
  zookeeper-1:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_SERVER_ID: 1 # zookeeper 클러스터에서 유일하게 주키퍼를 식별할 아이디
      ZOOKEEPER_CLIENT_PORT: 2181 # 즉 컨테이너 내부에서 주키퍼는 2181로 실행
      ZOOKEEPER_TICK_TIME: 2000 # 클러스터를 구성할때 동기화를 위한 기본 틱 타임. 단위는 ms
      ZOOKEEPER_INIT_LIMIT: 5 # 주키퍼 초기화를 위한 제한 시간, 멀티 브로커에서 유효
      ZOOKEEPER_SYNC_LIMIT: 2 # 주키퍼 리더와 나머지 서버들의 싱크 타임, 멀티 브로커에서 유효 
    ports:
      - "22181:2181"

  kafka-1:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper-1
    ports:
      - "29092:29092" # 외부포트:컨테이너내부포트
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: 'zookeeper-1:2181' # kafka가 zookeeper에 커넥션하기 위한 대상
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka-1:9092,PLAINTEXT_HOST://localhost:29092 # 외부에서 접속하기 위한 리스너 설정
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT # 보안을 위한 프로토콜 매핑
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT # 도커 내부에서 사용할 리스너 이름을 지정
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1 # 트랜잭션 상태에서 복제 계수를 지정
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1 # 트랜잭션 최소 ISR(InSyncReplicas 설정) 을 지정

  kafka-2:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper-1
    ports:
      - "39092:39092"
    environment:
      KAFKA_BROKER_ID: 2
      KAFKA_ZOOKEEPER_CONNECT: 'zookeeper-1:2181'
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka-2:9092,PLAINTEXT_HOST://localhost:39092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1

  kafka-3:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper-1
    ports:
      - "49092:49092"
    environment:
      KAFKA_BROKER_ID: 3
      KAFKA_ZOOKEEPER_CONNECT: 'zookeeper-1:2181'
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka-3:9092,PLAINTEXT_HOST://localhost:49092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1

  kafka-ui:
    image: provectuslabs/kafka-ui
    depends_on:
      - zookeeper-1
      - kafka-1
      - kafka-2
      - kafka-3
    container_name: kafka-ui
    ports:
      - "8080:8080"
    restart: always
    environment:
      - KAFKA_CLUSTERS_0_NAME=local
      - KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=kafka-1:9092,kafka-2:9092,kafka-3:9092
      - KAFKA_CLUSTERS_0_ZOOKEEPER=zookeeper-1:2181
```

> PLAINTEXT?
>   - 리스너에서 사용되는 보안 프로토콜 타입의 한 종류
>   - `PLAINTEXT` means the listener will be without authentication and non-encrypted.
>   - [https://stackoverflow.com/questions/50737950/what-does-plaintext-keyword-means-in-kafka-configuration](https://stackoverflow.com/questions/50737950/what-does-plaintext-keyword-means-in-kafka-configuration)


하고나니 devocean에서 kafka와 kafka와 kafka-ui를 docker-compose로 띄우고 자세한 설명까지 있는 게시물을 발견!
[https://devocean.sk.com/blog/techBoardDetail.do?ID=163980](https://devocean.sk.com/blog/techBoardDetail.do?ID=163980)
보면서 참고하면 좋을 것 같다.
