# 위젯 관리자 그룹 CRUD — 프로그램 명세

**작성 기준**: `WidgetManagerDrawer.jsx`, `useWidgetManagerGroups.js`, `widgetManagerUtils.js`

***

## 1. 데이터 구조와 흐름

### 1.1 드로어 진입

* `상태 drawer-opened === 'widget-manager'`
* Header **위젯 관리자** (admin+)

### 1.2 데이터 로드

| API                 | 내용          |
| ------------------- | ----------- |
| device subtree      | 디바이스·센서 트리  |
| sensor widgets      | 그룹 목록       |
| sensor type catalog | 위젯화된 센서목록   |
| camera              | CCTV 생성된 목록 |

### 1.3 useWidgetManagerGroups 상태

| 필드                | 설명         |
| ----------------- | ---------- |
| `sensorGroups`    | 그룹 목록      |
| `leftOrder`       | 좌측 + 그룹 순서 |
| `dirty`           | 미저장 변경     |
| `saving`          | 저장 중       |
| `deletingGroupId` | 삭제 중 ID    |

### 1.4 CRUD 흐름

1. API 응답 → 응답데이터정규화
2. 그룹 생성·삭제·센서 배정/해제
3. 저장: create/update/reorder → API + 상태갱신

### 1.5 그룹 종류

| widgetType | 종류      |
| ---------- | ------- |
| `widget`   | 센서 그룹   |
| `cctv`     | CCTV 그룹 |

### 1.6 defaultOptions seed 규칙

* 빈 그룹에 **첫 센서** 배정 시에만 sensor type defaultOptions 1회 복사
* 기존 위젯 센서 추가/제거: options 유지, deviceChannels 등만 갱신
* 마지막 센서 제거 → 타입·options 초기화 → 다음 첫 배정 시 재 seed

***

## 2. 관련 파일과 주요 함수

| 파일                            | 역할                                    |
| ----------------------------- | ------------------------------------- |
| `WidgetManagerDrawer.jsx`     | UI·데이터 조합                             |
| `useWidgetManagerGroups.js`   | CRUD 상태·API                           |
| `widgetManagerUtils.js`       | MIME, 정규화, 라벨                         |
| `cctv/AvailableCctvPanel.jsx` | CCTV 좌측 패널                            |
| `cctv/cctvManagerUtils.js`    | CCTV 저장/로드 매핑                         |
| `WidgetSettingDrawer`         | `WIDGET_SETTING_DRAWER_TARGET_KEY` 연계 |

***

## 3. 사용 컴포넌트 설명

| 컴포넌트                         | 설명            |
| ---------------------------- | ------------- |
| `WidgetManagerDrawer`        | 그룹 CRUD 메인 UI |
| `WidgetSettingDrawer`        | 선택 위젯 상세 설정   |
| `WidgetManagerChartTypeTabs` | 차트 타입 탭       |
| `WidgetSettingSensorList`    | 센서 목록 편집      |

***

## 4. 구현 시 주의사항

1. `id`, `sensorIds`, `options` 키 규칙 변경을 자제해야 함
2. sensor type catalog 수정은 **기존** 위젯 options에 영향 없음.

***

## 5. 관련 문서

| 문서                                                                                                                                               | 내용     |
| ------------------------------------------------------------------------------------------------------------------------------------------------ | ------ |
| [12-위젯-관리자-DnD-인터랙션.md](12-%EC%9C%84%EC%A0%AF-%EA%B4%80%EB%A6%AC%EC%9E%90-DnD-%EC%9D%B8%ED%84%B0%EB%9E%99%EC%85%98.md)                           | DnD    |
| [20-관리자-위젯-설정-이름-기본값.md](20-%EA%B4%80%EB%A6%AC%EC%9E%90-%EC%9C%84%EC%A0%AF-%EC%84%A4%EC%A0%95-%EC%9D%B4%EB%A6%84-%EA%B8%B0%EB%B3%B8%EA%B0%92.md) | 이름 기본값 |
