# 01. 앱 진입, 라우팅, 전역 드로어 로직

## 목적

앱이 시작될 때 어떤 Provider로 감싸서 실행되는지, 라우팅이 어떻게 분기되는지, 전역 드로어가 어떤 키로 제어되는지를 정리한다.

## 핵심 파일

- `src/main.jsx`
- `src/App.jsx`
- `src/context/AuthContext.jsx`
- `src/pages/dashboard/DashboardDrawerManager.jsx`
- `src/components/ChartDetailDrawer.jsx`
- `src/components/chart-detail-drawer/index.jsx`

## 부트스트랩 흐름

1. `main.jsx`에서 `ThemeProvider`와 `AuthProvider`를 적용한 뒤 `App`을 렌더링한다.
2. `AuthProvider`는 로컬 스토리지(`authToken`, `refreshToken`, `authUser`)를 읽어 인증 상태를 복원한다.
3. `App`에서 라우터를 구성하고 인증 여부에 따라 메인 페이지를 분기한다.

## 라우팅 분기

- `/`
  - 인증됨: `Dashboard`
  - 미인증: `Login`
- `/chart-detail`
  - `DetailPageRouter`에서 `sensorType` 기반으로 `useFft` 여부를 확인
  - FFT 차트면 `FftDetailPage`, 아니면 `DetailPage`
- 그 외 경로: `/`로 리다이렉트

## 전역 드로어 제어 키

SWR 키로 드로어 상태를 중앙 제어한다. 키 목록의 정본은 `docs/core-logic/17-common-data-structures-and-api.md`를 참조한다.

- `drawer-opened`: 현재 열린 드로어 타입
- `chart-detail-params`: Bento/siteConfig 차트 상세 파라미터
- `widget-detail-widget`: 위젯 그리드 상세 대상 위젯
- `widget-detail-scope`: 형제 위젯 채널 필터(`{ sensorId, channel }`)

드로어 렌더링은 세 계층에 나뉘어 있다.

- `App.jsx`: 전역 라우터 바깥에서도 떠야 하는 일부 드로어를 직접 렌더링한다.
  - `sys-monitor`
  - `threshold-setting`
- `DashboardDrawerManager`: 대시보드 내부 작업 드로어를 렌더링한다.
  - `threshold-setting`
  - `log`
  - `site-monitoring`
  - `sys-monitor`
  - `device-manager`
  - `widget-manager`
  - `device-type-manager`
  - `sensor-type-manager`
  - `chart-detail`
- `WidgetGridLayoutDashboard`: 위젯 그리드 전용 상세 드로어를 렌더링한다.
  - `widget-detail` (`WidgetDetailDrawer`, `!isMobile`일 때만 open)

`threshold-setting`, `sys-monitor`는 `App.jsx`와 `DashboardDrawerManager.jsx` 양쪽에 렌더링 경로가 있으므로, 중복 렌더링 가능성을 함께 고려해야 한다.

## ChartDetailDrawer 연결

- `src/components/ChartDetailDrawer.jsx`는 실제 구현체(`src/components/chart-detail-drawer/index.jsx`)를 재노출하는 엔트리다.
- 상세 드로어의 실질 로직(패널 분기, 폭 계산, 인쇄 모달)은 `chart-detail-drawer/index.jsx`에 있다.
- Bento/siteConfig 상세 흐름: `docs/core-logic/08-chart-detail-drawer-normal-flow.md`, `docs/core-logic/09-chart-detail-drawer-fft-flow.md`

## WidgetDetailDrawer 연결

- `src/pages/dashboard/widget-grid-layout/detail/WidgetDetailDrawer.jsx`는 `DashboardDrawerManager`가 아니라 `WidgetGridLayoutDashboard`에서 직접 렌더링한다.
- 상세 흐름: `docs/core-logic/19-widget-detail-drawer-flow.md`

## AuthContext 책임

- 로그인/로그아웃/토큰 재발급/내 정보 조회 API 호출
- 인증 상태와 사용자 정보 전역 제공
- 권한 도우미 값 제공:
  - `isAdmin`
  - `isSuperAdmin`

## 구현 시 주의점

- 드로어 확장은 `drawer-opened` 키 체계를 유지해야 한다.
- Bento/siteConfig 차트 상세는 `chart-detail-params`, 위젯 그리드 상세는 `widget-detail-widget`/`widget-detail-scope`를 표준으로 사용해야 한다.
- 인증 상태 복원 로직과 라우팅 분기 로직이 충돌하지 않도록 초기 렌더 순서를 유지해야 한다.
