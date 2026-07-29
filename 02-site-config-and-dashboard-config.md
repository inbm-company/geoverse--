# 02. 사이트 설정과 대시보드 구성 결합 로직

## 목적

정적 설정(`siteConfig`)과 DB 기반 동적 설정(그룹 노출/순서)을 결합해 실제 대시보드 렌더링에 사용할 차트/게이지 목록을 만드는 과정을 정리한다.

## 핵심 파일

- `src/config/siteConfig.js`
- `src/hooks/useDashboardConfig.js`
- `src/services/projectService.js`

## 입력 소스

### 정적 설정

- `getSite()`
- `getSiteGauges()`
- `getSiteChartConfigs()`
- `getSiteSiblings()`

### 동적 설정 (DB)

- `projectService.getGroupsBySiteCode(siteCode)`
- `projectService.getProjectBySiteCode(siteCode)` 또는 `projectService.getProject(siteCode)`

## useDashboardConfig 핵심 동작

1. 현재 `siteCode`를 조회한다.
2. SWR로 그룹 데이터와 프로젝트 데이터를 로드한다.
3. 사이트별 예외 조건을 확인한다.
   - `siteCode === '1005'`는 정적 대시보드 설정 우선
   - 게이지 8개 이하 소규모 사이트는 정적 설정 우선
4. 동적 적용 대상이면 DB 그룹 설정을 기반으로 필터링한다.
   - `showHide === true`만 활성 그룹으로 사용
   - `orderIndex` 기준 정렬
5. `Map` 기반 룩업으로 차트/게이지를 빠르게 매핑한다.
   - chart key: `groupId ?? id`
   - gauge key: `groupId`
6. 활성 그룹 목록으로 최종 결과를 만든다.
   - `filteredCharts`
   - `filteredGauges`
   - `activeGroupIds`
7. 시블링 데이터는 활성 센서 타입에 맞춰 후처리한다.
   - 비활성 센서 타입 아이템 제거
   - 차트 센서의 `calculation`을 시블링 아이템에 동기화

## 반환 데이터

`useDashboardConfig`는 아래를 반환한다.

- `filteredCharts`
- `filteredGauges`
- `siblings`
- `isLoading`
- `error`
- `siteCode`
- `dbSensorGroups`
- `dbProject`

## 성능 포인트

- SWR 옵션으로 불필요한 재검증을 줄인다.
  - `revalidateOnFocus: false`
  - `dedupingInterval: 600000`
- 필터링/매핑 로직은 `useMemo`로 메모이제이션한다.
- 룩업 테이블(`Map`) 사용으로 검색 비용을 줄인다.

## 구현 시 주의점

- 그룹 ID와 차트/게이지 매핑 키 규칙(`groupId`, `id`)을 바꾸면 대시보드 구성 전체가 깨질 수 있다.
- `1005`와 소규모 사이트의 정적 우선 조건은 운영 정책에 가까우므로 임의 제거하지 않는다.
- 시블링 아이템의 `calculation` 동기화는 표시값 일관성에 직접 영향을 준다.
