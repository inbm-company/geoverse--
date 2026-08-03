# 설정

`config/config.js`는 management, collector, sensor-api 세 서버가 공유하는 런타임 설정 모듈이다.\
각 항목은 환경변수로 덮어쓸 수 있고, 환경변수가 없으면 파일에 정의된 기본값을 사용한다.

```js
const { redis, collector, influxdb } = require('../config/config');
```

***

### base

실행 환경을 나타낸다. `NODE_ENV`가 없으면 `development`로 동작한다.\
Aligo API 등 dev/prod 분기에 참조된다.

***

### redis

Redis 연결 URL을 설정한다. 기본값은 `redis://localhost:6379`이다.\
collector의 실시간 센서 데이터 저장, 알람 시스템, management·sensor-api의 Redis 조회에 사용된다.

***

### management

Management 서버 HTTP 포트를 설정한다. 기본값은 `3001`이다.

***

### sensorApi

Sensor API 서버 설정이다.

* `port`: HTTP 포트. 기본값 `3002`
* `mongoUri`: MongoDB 연결 URI. 기본값 `mongodb://localhost:27017/geoverse`

***

### collector

수집 서버의 네트워크, 배치 집계, Redis 정리, 외부 백업 API 설정을 담는다.

#### 네트워크

* `port`: Collector HTTP/WebSocket 포트. 기본값 `3003`
* `url`: Collector 기본 URL. 기본값 `http://localhost:3003`
* `mqttUrl`: MQTT 브로커 URL. 기본값 `mqtt://localhost:1883`
* `mongoUri`: MongoDB 연결 URI

#### 배치·집계 스케줄

* `batchIntervalMinutes`: 10분 집계 주기(분). 기본값 `10`
* `hourlyAggregationOffsetMinutes`: 시간별 집계 실행 시각. 매시 X분에 실행. 기본값 `5`
* `dailyAggregationOffsetMinutes`: 일별 집계 실행 시각. 매일 00:X분에 실행. 기본값 `10`

`batch-scheduler.js`에서 시간별·일별 집계 cron 스케줄에 사용된다.

#### Redis 데이터 정리·백업

* `redisCleanupIntervalHours`: Redis 정리 주기(시간). 기본값 `24`
* `redisDataRetentionHours`: 백업 성공 후 삭제 기준 보존 시간(시간). 기본값 `48`
* `redisBackupOffsetMinutes`: Redis 백업 실행 시각. 매일 00:X분. 기본값 `30`
* `redisTestCleanupOffsetMinutes`: 10분 단위 테스트 정리 cron 분 오프셋. 기본값 `5`

`management/backup-scheduler.js`, `redis-backup.js`, `redis-storage.js`에서 사용된다.

#### 집계 범위

* `hourlyLookbackHours`: 시간별 집계 lookback 범위(시간). 기본값 `1`
* `dailyLookbackHours`: 일별 집계 lookback 범위(시간). 기본값 `24`

#### rollup

InfluxDB 롤업(집계) 간격 설정이다.

* `enabled`: 롤업 활성화 여부. `ROLLUP_ENABLED=true`일 때만 활성. 기본값 `false`
* `tenMinuteInterval`: 10분 롤업 주기. 기본값 `10m`
* `hourlyInterval`: 시간별 롤업 주기. 기본값 `1h`
* `dailyInterval`: 일별 롤업 주기. 기본값 `24h`

`batch-scheduler.js`에서 10분 배치 처리 구간 계산에 `tenMinuteInterval`을 사용한다.

#### backupApi

외부 백업 서비스 호출 설정이다. Redis 데이터를 백업 API(`POST /api/v1/backup/sensor-backup`)로 전송할 때 사용한다.

* `url`: 백업 API URL. 기본값 `http://127.0.0.1:8000`
* `timeout`: 요청 타임아웃(ms). 기본값 `60000`
* `targetSensorId`: 백업 대상 센서 ID. 기본값 `001e0651485b`
* `emailTo`: 백업 완료 알림 이메일. 기본값 `admin@company.com`

***

### influxdb

InfluxDB 연결 및 버킷 설정이다.

* `url`: InfluxDB URL. 기본값 `http://localhost:8086`
* `token`: API 토큰
* `org`: Organization 이름. 기본값 `my-org`
* `orgId`: Organization ID
* `bucket`: 공통 10분 버킷(레거시). 기본값 `sensors`
* `hourlyBucket`: 공통 시간별 버킷(레거시). 기본값 `sensors_hourly`
* `dailyBucket`: 공통 일별 버킷(레거시). 기본값 `sensors_daily`

#### getBucketName(sensorId, channelOrType, timeUnit)

센서별 InfluxDB 버킷명을 생성하는 함수이다.

**신규 방식 (3인자):** `getBucketName(sensorId, channel, timeUnit)`\
→ `{sensorId}_{channel}_{timeUnit}` 형식. 예: `001e0651485b_ch1_10m`

**레거시 방식 (2인자):** `getBucketName(sensorId, type)`\
→ `10min` → `{sensorId}_10m`, `hourly` → `{sensorId}_1h`, `daily` → `{sensorId}_1d`

***

### auth

Management 서버 JWT 인증 설정이다.

* `jwtSecret`: JWT 서명 키
* `jwtExpiresIn`: Access token 만료 시간. 기본값 `24h`

`management/middleware/auth.js`, `management/services/token-service.js`에서 사용된다.

***

### aligo

SMS 발송(알리고) API 자격증명이다.

* `apiKey`: API 키
* `userId`: 사용자 ID. 기본값 `geokorea`
* `sender`: 발신번호. 기본값 `031-454-1780`

`collector/alarm/notifier.js`, `management/aligo-api/index.js`에서 사용된다.\
수신자 번호는 이 설정이 아니라 DB·알람 config-loader에서 조회한다.

***

### alarm

알람 시스템 전역 옵션이다.

* `enabled`: 알람 활성 여부. `ALARM_ENABLED=false`일 때만 비활성. 기본값 `true`
* `receiver`: 기본 수신자 번호
* `testMode`: Aligo 테스트 모드. 기본값 `N`
* `stateStabilitySeconds`: 상태 안정화 대기 시간(초). 기본값 `10`
* `cooldownMinutes`: 동일 알람 재발송 쿨다운(분). 기본값 `10`

***

### weather

기상청 초단기실황 API 설정이다.

* `apiKey`: API 키
* `cacheTtlMs`: 응답 캐시 TTL(ms). 기본값 30분

`sensor-api/data-access/weather-client.js`에서 사용된다.

***

### postgres

Management 서버 메타데이터 DB(PostgreSQL) 연결 풀 설정이다.

* `host`: DB 호스트. 기본값 `localhost`
* `port`: DB 포트. 기본값 `5432`
* `database`: DB 이름. 기본값 `geoverse`
* `user`: 사용자. 기본값 `postgres`
* `password`: 비밀번호
* `max`: 커넥션 풀 최대 크기. 기본값 `20`
* `idleTimeoutMillis`: 유휴 연결 타임아웃(ms). 기본값 `30000`
* `connectionTimeoutMillis`: 연결 타임아웃(ms). 기본값 `2000`

`management/database/pg-client.js`에서 사용된다.
