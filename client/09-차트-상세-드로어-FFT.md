# 차트 상세 드로어(FFT) — 프로그램 명세

**작성 기준**: `chart-detail-drawer/index.jsx`, `FftDrawerPanel.jsx`, `useFftData.js`

***

## 1. 데이터 구조와 흐름

### 1.1 FFT 분기

`chart-detail-drawer/index.jsx`에서 센서 타입이 `tension`, `seismometer`, `accelerometer`, `newAccelerometer` 등 FFT 대상이면 `useFftData` → `FftDrawerPanel`.

(siteConfig `useFft`와 상세 드로어 FFT\_SENSOR\_TYPES 목록을 함께 확인)

### 1.2 useFftData

| 단계  | 내용                                                          |
| --- | ----------------------------------------------------------- |
| 센서  | `chartId` 우선 → `sensorType`                                 |
| 날짜  | range: 최근 24h / selectedDateTime: 10분 단위                    |
| API | range + `isFft=true`, initial + range 동시                    |
| 출력  | `rangeData`, `initialData`, `chartData`, `excelData`        |
| 이미지 | `selectedDateTime`, `selectedAxis`, chartType → Raw/FFT URL |

### 1.3 FftDrawerPanel UI

1. **10분 섹션**: `SingleDateSearchHeader`, 축·센서 선택, Raw/FFT 이미지
2. **기간 섹션**: `PeriodSearchHeader(isFFT)`, `DetailChart`, `SummaryTable`, `BaseTable`

### 1.4 표시 규칙

* `chart.showFftChart !== false`일 때 FFT 이미지
* seismometer 등: `naturalFreq` 보조 표시

***

## 2. 관련 파일과 주요 함수

| 파일                                        | 역할               |
| ----------------------------------------- | ---------------- |
| `chart-detail-drawer/index.jsx`           | FFT 분기           |
| `FftDrawerPanel.jsx`                      | FFT UI           |
| `hooks/useFftData.js`                     | FFT 데이터·이미지 URL  |
| `pages/detail/SingleDateSearchHeader.jsx` | 10분 시점 선택        |
| `utils/chartDetailUtils.js`               | FFT 차트 타입·URL 헬퍼 |

***

## 3. 사용 컴포넌트 설명

| 컴포넌트                 | 설명                                                               |
| -------------------- | ---------------------------------------------------------------- |
| `ChartDetailDrawer`  | FFT 상세                                                           |
| `WidgetDetailDrawer` | `parseType === 'fft'` 시 FftDrawerPanel + `useWidgetFftImageData` |
| `FftDetailPage`      | `/chart-detail` 전용 FFT 페이지                                       |

***

## 4. 구현 시 주의사항

1. 단일 시점 이미지와 기간 차트 시간 파라미터 동기화 중요.
2. `selectedAxis` 기본값은 `dataKey`와 연결.
3. 이미지 URL 실패와 차트 데이터 실패는 별도 에러 처리.

***

## 5. 관련 문서

| 문서                                                                                                                         | 내용     |
| -------------------------------------------------------------------------------------------------------------------------- | ------ |
| [08-차트-상세-드로어-일반.md](08-%EC%B0%A8%ED%8A%B8-%EC%83%81%EC%84%B8-%EB%93%9C%EB%A1%9C%EC%96%B4-%EC%9D%BC%EB%B0%98.md)           | 일반 상세  |
| [19-위젯-그리드-상세-드로어.md](19-%EC%9C%84%EC%A0%AF-%EA%B7%B8%EB%A6%AC%EB%93%9C-%EC%83%81%EC%84%B8-%EB%93%9C%EB%A1%9C%EC%96%B4.md) | 위젯 FFT |
