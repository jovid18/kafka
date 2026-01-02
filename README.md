# Kafka Playground

Kafka Connect 학습을 위한 로컬 환경입니다.

## 목차

- [환경 설정](#환경-설정)
- [Kafka Connect API](#kafka-connect-api)
- [실습: FileStream 커넥터](#실습-filestream-커넥터)

---

## 환경 설정

### Prerequisites

- Docker (OrbStack 또는 Docker Desktop)

### Services

| Service | Port | Description |
|---------|------|-------------|
| Zookeeper | 2181 | Kafka 클러스터 관리 |
| Kafka | 9092 | Kafka Broker |
| Kafka Connect | 8083 | Kafka Connect REST API |

### 실행

```bash
# 시작
docker-compose up -d

# 상태 확인
docker-compose ps

# 로그 확인
docker-compose logs -f kafka-connect

# 종료
docker-compose down
```

---

## Kafka Connect API

### 상태 확인

```bash
curl http://localhost:8083/
```
```json
{"version":"7.5.0-ccs","commit":"ff3c201baa948d97889dc26c99d7cdc23d038f2e","kafka_cluster_id":"..."}
```

### 플러그인 목록

```bash
curl http://localhost:8083/connector-plugins
```
```json
[
  {"class":"org.apache.kafka.connect.file.FileStreamSourceConnector","type":"source"},
  {"class":"org.apache.kafka.connect.file.FileStreamSinkConnector","type":"sink"},
  {"class":"org.apache.kafka.connect.mirror.MirrorSourceConnector","type":"source"}
]
```

### 커넥터 관리

```bash
# 커넥터 목록
curl http://localhost:8083/connectors

# 커넥터 생성
curl -X POST http://localhost:8083/connectors \
  -H "Content-Type: application/json" \
  -d '{"name":"...", "config":{...}}'

# 커넥터 상태 확인
curl http://localhost:8083/connectors/{name}/status

# 커넥터 삭제
curl -X DELETE http://localhost:8083/connectors/{name}
```

---

## 실습: FileStream 커넥터

파일과 Kafka 토픽 간 데이터를 주고받는 실습입니다.

### 파이프라인 구조

```
[/etc/kafka/server.properties]    원본 파일
              │
              ▼ FileStreamSource (파일 → 토픽)
              │
[kafka-config-topic]              Kafka 토픽
              │
              ▼ FileStreamSink (토픽 → 파일)
              │
[/tmp/copy-of-server-properties]  복사된 파일
```

### Step 1. Source 커넥터 생성

파일을 읽어서 Kafka 토픽으로 전송합니다.

```bash
curl -X POST http://localhost:8083/connectors \
  -H "Content-Type: application/json" \
  -d '{
    "name": "load-kafka-config",
    "config": {
      "connector.class": "FileStreamSource",
      "file": "/etc/kafka/server.properties",
      "topic": "kafka-config-topic"
    }
  }'
```

### Step 2. 토픽 메시지 확인

```bash
docker exec kafka kafka-console-consumer \
  --bootstrap-server localhost:9092 \
  --topic kafka-config-topic \
  --from-beginning
```
> Ctrl+C로 종료

### Step 3. Sink 커넥터 생성

Kafka 토픽의 메시지를 파일로 저장합니다.

```bash
curl -X POST http://localhost:8083/connectors \
  -H "Content-Type: application/json" \
  -d '{
    "name": "dump-kafka-config",
    "config": {
      "connector.class": "FileStreamSink",
      "file": "/tmp/copy-of-server-properties",
      "topics": "kafka-config-topic"
    }
  }'
```

### Step 4. 결과 확인

```bash
docker exec kafka-connect cat /tmp/copy-of-server-properties
```

### Step 5. 정리

```bash
curl -X DELETE http://localhost:8083/connectors/dump-kafka-config
curl -X DELETE http://localhost:8083/connectors/load-kafka-config
```

### 커넥터 비교

| 타입 | 클래스 | 방향 | 주요 설정 |
|------|--------|------|-----------|
| Source | FileStreamSource | 파일 → 토픽 | `file`, `topic` |
| Sink | FileStreamSink | 토픽 → 파일 | `file`, `topics` |

> **Note**: Source는 `topic` (단수), Sink는 `topics` (복수)를 사용합니다.
