# kafka
kafka playground

## Prerequisites

- Docker (OrbStack or Docker Desktop)
- docker-compose

## Quick Start

```bash
# 실행
docker-compose up -d

# 상태 확인
docker-compose ps

# 종료
docker-compose down
```

## Services

| Service | Port | Description |
|---------|------|-------------|
| Zookeeper | 2181 | Kafka 클러스터 관리 |
| Kafka | 9092 | Kafka Broker |
| Kafka Connect | 8083 | Kafka Connect REST API |

## Kafka Connect API

```bash
# Kafka Connect 상태 확인
curl http://localhost:8083/
# {"version":"7.5.0-ccs","commit":"ff3c201baa948d97889dc26c99d7cdc23d038f2e","kafka_cluster_id":"WM-OzxqXTqy8tKXEY66_jQ"}

# 커넥터 목록
curl http://localhost:8083/connectors

# 사용 가능한 플러그인 확인
curl http://localhost:8083/connector-plugins
# [{"class":"org.apache.kafka.connect.mirror.MirrorCheckpointConnector","type":"source","version":"7.5.0-ccs"},
#  {"class":"org.apache.kafka.connect.mirror.MirrorHeartbeatConnector","type":"source","version":"7.5.0-ccs"},
#  {"class":"org.apache.kafka.connect.mirror.MirrorSourceConnector","type":"source","version":"7.5.0-ccs"}]

# 커넥터 생성 예시
curl -X POST http://localhost:8083/connectors \
  -H "Content-Type: application/json" \
  -d '{
    "name": "my-connector",
    "config": {
      "connector.class": "...",
      "tasks.max": "1"
    }
  }'

# 특정 커넥터 상태 확인
curl http://localhost:8083/connectors/{connector-name}/status

# 커넥터 삭제
curl -X DELETE http://localhost:8083/connectors/{connector-name}
```

## Logs

```bash
# 전체 로그
docker-compose logs -f

# Kafka Connect 로그만
docker-compose logs -f kafka-connect
```
