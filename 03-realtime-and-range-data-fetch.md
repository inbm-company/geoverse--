# 03. 실시간/기간 데이터 조회 로직

## 목적

차트와 게이지에서 사용하는 실시간 API/기간 API 조회 방식, 파라미터, 반환 형태를 정리한다.

## 핵심 파일

- `src/hooks/useRealtimeAPI.js`
- `src/hooks/useRangeAPI.js`

참고 파일:
- `src/services/sensorService.js`: HTTP 센서 데이터 조회가 아니라 Socket.IO 연결을 래핑하는 서비스다.

## useRealtimeAPI

### 역할

- 실시간 구간 데이터 API를 호출한다.
- 응답 원본을 `rawApiResponse`로 보관한다.
- 고빈도 차트는 `dataFormatter.js`에서 응답을 파싱한다.

### 주요 파라미터

- `sensorId`
- `hours`
- `aggregateSeconds`
- `options`
  - `includeMinMax`
  - `channel`

### 요청 규칙

- endpoint: `/api/sensors/realtime`
- query:
  - `sensorId`
  - `hours`
  - `aggregateSeconds`
  - `types` (`avg` 또는 `avg,min,max`)
  - `channel`(옵션)

### 반환

- `rawApiResponse`
- `loading`
- `error`
- `refetch`
- `currentAggregateSeconds`

## useRangeAPI

### 역할

- 기간 조회 API로 시계열 데이터를 조회한다.
- 시작/종료 시각이 없으면 호출하지 않고 빈 배열을 반환한다.

### 주요 파라미터

- `sensorId`
- `startDate`
- `endDate`
- `options`
  - `includeMinMax`
  - `channel`
  - `isFft`
  - `parseType` (LoRa/GDMS 조회 시 `isLora` 판정에 사용)

### 요청 규칙

- endpoint: `/api/sensors/range`
- query:
  - `sensorId`
  - `start`, `end` (ISO 8601)
  - `types` (`avg` 또는 `avg,min,max`)
  - `channel`(일반 MQTT 필수, LoRa/GDMS는 생략 가능)
  - `isFft=true`(FFT 조회 시)
  - `isLora=true`(GDMS 또는 `parseType` lora/gdms, 또는 `sensorId`가 `lora_` 접두일 때 — `sensorRangeParams.js` 참고)

### 응답 처리

- `data.data` 배열을 사용
- `channel`이 있으면 채널 필터 적용
- `time` 기준 오름차순 정렬

### 반환

- `rangeData`
- `loading`
- `error`
- `refetch`

## 상태 관리 공통점

- 두 훅 모두 `loading/error` 상태를 내부에서 관리한다.
- 실패 시 `error` 문자열을 설정하고 콘솔에 로그를 남긴다.
- 외부에서는 `refetch()`로 재조회한다.

## 차트 연계 포인트

- 고빈도 차트는 조회 결과를 `high-frequency/dataFormatter.js`에서 파싱/가공한다.
- 저빈도 차트는 조회 결과를 센서 config의 `dataProcessor`에서 파싱/가공한다.
- 상세 드로어(`useNormalData`, `useFftData`)는 같은 range API를 직접 조합해 추가 데이터(initialData, 테이블 데이터)를 만든다.

## 구현 시 주의점

- `startDate/endDate`가 없는 range 호출은 무조건 차단한다.
- `options` 변경 시 재호출 의존성에 포함되어야 한다.
- channel 필터와 `types` 조합은 테이블/차트 값 불일치의 주요 원인이므로 변경 시 검증이 필요하다.
