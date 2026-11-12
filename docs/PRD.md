# LunarSafe PRD (Product Requirements Document)

| 항목 | 내용 |
|---|---|
| 제품명 | LunarSafe — Terrain-Uncertainty-Aware Lunar Mission Window Planner |
| 대회 | NASA Space Apps Challenge 2026 / Lunar Landing Site Navigator |
| 문서 버전 | PRD v1.0 (2026-11-12), 기준 문서: 프로젝트 계획서 v1.0 |
| 개발 가능 시간 | 2026-11-14 09:00 ~ 11-15 23:59 (현지 시간, 약 39시간) |
| 상태 | 기획 확정. **코드 작성은 11-14 09:00 이후에만 시작한다** (사전 제작 실격 위험) |

표기 규칙: `[확인]`이 붙은 항목은 이 문서 작성 시점에 원문으로 검증하지 못한 값이다. 담당자가 T+0~3h 안에 공식 문서로 확인하고 이 문서를 고친다.

---

## 1. 제품 정의

### 1.1 한 문장

사용자가 정한 임무 조건(기간, 허용 암흑 시간, 허용 통신 시야 단절 시간, 장비 높이)에 대해, 달 남극 두 후보지에서 조건을 만족하는 임무 시작 시점을 찾고, 그 판단이 지형 불확실성 아래에서도 유지되는지 보여주는 웹 애플리케이션.

### 1.2 사용자와 사용 시나리오

- 사용자: 심사위원, 달 탐사 초기 기획을 공부하는 학생·연구자.
- 기준 시나리오(데모와 동일):
  1. 후보지 A(Site01 Connecting Ridge), B(Site04 Shackleton Rim)를 고른다.
  2. 임무 기간 72 h, 허용 연속 암흑 12 h, 허용 연속 지구 시야 단절 6 h, 장비 높이 1 m를 입력한다.
  3. 시작 가능 범위(기본 30일)를 정하고 분석을 실행한다.
  4. 후보지별 가시성 타임라인과 조건 충족 시작 시점을 본다.
  5. 지형 실현 자료 N개 중 몇 개에서 판단이 유지되는지 본다.

### 1.3 하지 않는 것 (Non-goals)

착륙 궤도·추진제 최적화, 구조 안전성, 배터리·열 설계, 링크 버짓, 달 전역 분석, 비행 승인용 결과. "착륙 안전", "성공 확률"이라는 표현은 UI·문서·발표 어디에도 쓰지 않는다.

### 1.4 계획서 v1.0 대비 변경점

| # | 변경 | 이유 |
|---|---|---|
| C1 | 수평선 계산을 근거리(5 m 사이트 DEM) + 원거리(광역 LOLA 남극 DEM) 2단으로 구성 | 5 m 타일은 범위가 좁다. 높이 차 H인 지형은 약 √(2·R·H) 거리까지 수평선에 걸린다(R=1737.4 km, H=8 km이면 약 167 km) |
| C2 | 태양·지구 원반 판정(3상태)을 P2에서 P1으로 승격 | 지구 겉보기 반지름 약 0.95°, 태양 약 0.27°. 남극에서 지구는 수평선 근처에 머물러 중심점 판정만으로는 결과가 거칠다 |
| C3 | 화면 4개를 3개로 통합 | 39시간 안에 시연 안정성 확보 |
| C4 | 불확실성 분석은 근거리 지형만 교체하고 원거리 수평선은 캐시 재사용 | 실현 자료는 사이트 범위에만 있다. 연산량도 줄어든다 |
| C5 | 지점 주변 격자(3×3) 민감도 추가 (P2) | "수 m 이동으로 결과가 바뀌는가"를 보여준다 |

---

## 2. 우선순위와 완료 기준

| ID | 기능 | 등급 | 완료 기준 (Acceptance) |
|---|---|---|---|
| F1 | 사이트·지점 정보 제공 | P0 | `GET /sites`가 2개 사이트의 id, 이름, 분석 지점 위경도, DEM 유효 범위를 반환 |
| F2 | 태양·지구 방위각/고도각 시계열 | P0 | SPICE 계산값이 Horizons와 방위각·고도각 모두 0.05° 이내 (§8.1) |
| F3 | 지형 수평선 프로파일 H(A) | P0 | 720개 방위(0.5° 간격)에 대해 차폐각과 유효 플래그 반환, 테스트 §8.2 통과 |
| F4 | 가시성 시계열과 지표 | P0 | §5.4 지표 6종을 단위·기간과 함께 반환 |
| F5 | 두 후보지 비교 화면 | P0 | 동일 조건의 타임라인 2개와 지표 표가 한 화면에 표시 |
| F6 | 방법·가정·출처 공개 | P0 | 화면과 README에 좌표계, 표본 간격, 판정 규칙, 데이터 버전 표기 |
| F7 | 임무 시작 시점 탐색 | P1 | 조건 입력 → 시작 시점별 `FEASIBLE / VIOLATION / INDETERMINATE`와 사유 반환 |
| F8 | 원반 3상태 판정 | P1 | `FULL / PARTIAL / HIDDEN` 구분, 판정 규칙 선택 가능 |
| F9 | 장비 높이 비교 | P1 | 높이 2개 값의 결과를 나란히 표시, 높이 증가 시 가시성 비악화 테스트 통과 |
| F10 | 지형 실현 자료 민감도 | P2 | 실현 자료 k개 중 조건 충족 개수 표시 ("10개 중 8개 충족") |
| F11 | 주변 격자 민감도 | P2 | 3×3 지점의 판정 결과 표시 |
| F12 | CSV 내보내기 | P2 | 시계열과 분석 조건을 CSV로 다운로드 |

원칙: P0가 전부 끝나기 전에는 P1을, P1이 끝나기 전에는 P2를 시작하지 않는다. 계산하지 못한 값은 화면에 "계산 불가"로 표시하고 임의 수치를 넣지 않는다.

---

## 3. 데이터 명세

### 3.1 지형

| 용도 | 자료 | 형식 | 비고 |
|---|---|---|---|
| 근거리 DEM | PGDA Product 78, Site01·Site04 표면 보간 고도 지도 | 5 m/pix GeoTIFF, 남극 극평사 X/Y(m), 값은 고도 Z(m) | https://pgda.gsfc.nasa.gov/products/78 |
| 고도 불확실성 | 같은 상품의 `*_toterr.tif` (총 Z 불확실성, RMS, m) | GeoTIFF | 경계 구간 판정에 사용 |
| 지형 실현 자료 | 같은 상품의 `*_0001_err.tif` ~ `*_0100_err.tif` (사이트당 100개) | GeoTIFF | `[확인]` 오차장인지(기준 DEM에 더함) 완성 DEM인지 README로 확인 |
| 원거리 DEM | LOLA 남극 광역 DEM (20 m/pix급, 80°S 이남) | GeoTIFF | `[확인]` PGDA Product 81의 `LDEM_80S_20MPP_ADJ.TIF` 사용 가능 여부. 불가 시 PDS LOLA GDR 극지 DEM으로 대체 |

처리 규칙:

- 좌표 변환은 GeoTIFF에 들어 있는 CRS를 `rasterio`/`pyproj`로 읽어 수행한다. 투영식을 손으로 구현하지 않는다.
- 고도 Z의 기준 반지름은 1737.4 km로 가정한다. `[확인]` 상품 README의 기준 반지름.
- 원거리 DEM은 서버 시작 시 240 m 격자로 다운샘플한 배열을 만든다. 블록 평균과 블록 최댓값 두 배열을 보관하고, 수평선 계산에는 최댓값 배열을 쓴다(차폐를 과소평가하지 않기 위함. README에 명시).
- nodata와 타일 밖 영역은 NaN으로 두고 보간으로 채우지 않는다.

### 3.2 천체력 (SPICE 커널)

| 종류 | 파일 | 용도 |
|---|---|---|
| LSK | `naif0012.tls` | UTC ↔ ET |
| SPK | `de440s.bsp` | 태양·지구·달 위치 |
| PCK(텍스트) | `pck00011.tpc` | 천체 반지름 |
| PCK(바이너리) | `moon_pa_de440_200625.bpc` | 달 자세(주축 프레임) |
| FK | `moon_de440_*.tf` | MOON_PA → MOON_ME 변환 |

`[확인]` 파일명과 최신 버전은 https://naif.jpl.nasa.gov/pub/naif/generic_kernels/ 에서 확인한다. 달 고정 프레임 이름(`MOON_ME_DE440_ME421` 예상)은 FK 파일 본문에서 확인한다.

커널은 저장소에 커밋하지 않는다. `scripts/fetch_data.sh`가 내려받고 `data/`는 `.gitignore`에 넣는다. 사용한 파일명과 SHA-256을 `data/MANIFEST.json`에 기록한다.

### 3.3 분석 지점

- 사이트당 기준 지점 1개. T+0~3h에 DEM을 열어 능선·가장자리의 유효 격자에서 고르고 `backend/app/sites.json`에 위도, 경도, 선택 근거를 기록한다.
- 기준 지점 주변 ±50 m의 3×3 격자가 F11의 대상이다.

---

## 4. 시스템 구조

```
frontend (Vite + React + TypeScript, Plotly)
        │  HTTP/JSON
backend (Python 3.11, FastAPI)
  ├─ ephemeris.py     SPICE 래퍼: 시각 배열 → 태양·지구 az/el/각반지름
  ├─ terrain.py       DEM 로딩, 좌표 변환, 표본 추출
  ├─ horizon.py       수평선 프로파일 H(A) 계산 + 디스크 캐시
  ├─ visibility.py    3상태 판정, 지표 계산
  ├─ windows.py       임무 시작 시점 탐색
  ├─ uncertainty.py   실현 자료 반복 계산
  └─ api.py           라우터, Pydantic 스키마
```

저장소 구조:

```
backend/
  app/{api,ephemeris,terrain,horizon,visibility,windows,uncertainty}.py
  app/sites.json
  tests/
  pyproject.toml
frontend/
  src/{pages,components,api}/
scripts/fetch_data.sh
data/            (gitignore)
cache/           (gitignore, 수평선 .npz)
docs/PRD.md
Dockerfile
README.md
```

의존성: `fastapi`, `uvicorn`, `numpy`, `scipy`, `spiceypy`, `rasterio`, `pyproj`, `pytest`, `httpx`. 프론트: `react`, `plotly.js`, `react-plotly.js`.

모듈 규칙: 계산 모듈은 FastAPI를 import하지 않는다. 모든 함수는 NumPy 배열을 받고 반환해 단독 테스트가 가능해야 한다.

---

## 5. 계산 명세

### 5.1 좌표와 단위

- 달 고정 프레임: MOON_ME (LOLA와 동일). SPICE 호출은 모두 이 프레임으로 한다.
- 경도: 동경 양수, 내부 표현 [0°, 360°). 위도: 남위 음수.
- 방위각: 북 0°, 동 90°, 시계 방향. 고도각: 수평 0°, 천정 +90°.
- 시간: API 입출력은 ISO 8601 UTC, 내부는 ET(TDB 초).
- 내부 각도는 라디안, API와 UI는 도.

### 5.2 천체 위치 (`ephemeris.py`)

입력: 위도, 경도, 지형 고도 z0(m), 장비 높이 h(m), ET 배열.

1. 관측자 위치 벡터 `p = (R + z0 + h) · û(lat, lon)` (MOON_ME, R = 1737.4 km).
2. 대상 벡터: `spkpos(target, et, MOON_ME, "LT+S", "MOON")`에서 `p`를 뺀다. target은 `SUN`, `EARTH`.
3. 지역 ENU 기저: `up = û`, `east = ẑ × up` 정규화, `north = up × east`.
4. `el = asin(d·up / |d|)`, `az = atan2(d·east, d·north) mod 2π`.
5. 각반지름 `ρ = asin(R_body / |d|)`. R_sun = 695,700 km, R_earth = 6,378.137 km.

출력: 대상별 `az[], el[], rho[]`.

주의: 극점 근처에서 `ẑ × up`이 0에 가까워진다. |lat| > 89.9°이면 east를 경도 기준으로 직접 정의한다(`east = (−sin lon, cos lon, 0)`).

### 5.3 수평선 프로파일 (`horizon.py`)

출력: 방위각 720개(0.5° 간격)의 `H[]`(rad)와 `valid[]`(bool), 그리고 `max_range_km[]`.

절차:

1. 관측자 3차원 위치 `p`와 ENU 기저를 구한다(§5.2).
2. 각 방위각 A에 대해 대원 방향으로 거리 표본을 만든다.
   - 근거리: 5 m ~ 사이트 DEM 경계, 간격 5 m (최초 2 km), 이후 20 m.
   - 원거리: 사이트 DEM 경계 ~ 200 km, 간격 240 m.
3. 표본 지점의 위경도 → DEM 픽셀(쌍선형 보간) → 고도 z → 3차원 위치 `q = (R + z) · û`.
4. `d = q − p`, 고도각 `e = asin(d·up / |d|)`. 3차원 벡터로 계산하므로 달 곡률이 자동 반영된다.
5. `H(A) = max(e)`.
6. 유효성: 해당 방위의 표본 중 NaN 비율이 5%를 넘거나, 200 km 이전에 자료가 끊기면 `valid = false`.

구현 메모:

- 방위각 루프를 돌지 않고 (720 × N거리) 배열로 벡터화한다.
- 근거리 결과 `H_near`와 원거리 결과 `H_far`를 따로 저장하고 `H = max(H_near, H_far)`로 합친다. C4의 캐시 재사용을 위한 구조다.
- 캐시 키: `(site_id, lat, lon, h, dem_id, realization_id)`의 해시. `cache/*.npz`로 저장.
- 최대 거리 200 km의 근거: 관측자보다 H만큼 높은 지형이 수평선에 걸리는 한계 거리 ≈ √(2·R·H). H = 8 km일 때 약 167 km.

### 5.4 가시성 판정 (`visibility.py`)

각 시각 t, 각 천체에 대해:

- `H_c = interp(H, az)` (0°/360° 경계는 원형 보간).
- `H_lo = min H`, `H_hi = max H` — 방위각 구간 `[az − ρ, az + ρ]` 안에서.
- 판정:
  - `FULL`: `el − ρ > H_hi`
  - `HIDDEN`: `el + ρ < H_lo`
  - `PARTIAL`: 그 외
  - `INVALID`: 구간 안에 `valid = false`인 방위가 있음
- 여유각 `margin = el − H_c`도 함께 반환한다.

가시 규칙(`rule`)은 사용자가 고른다.

| rule | "보임"으로 치는 상태 | 기본 적용 |
|---|---|---|
| `any_limb` | FULL, PARTIAL | 태양 기본값 |
| `center` | margin > 0 | 지구 기본값 |
| `full_disk` | FULL | 보수적 옵션 |

지표(분석 구간 기준, 표본 간격 Δt 기본 10분):

| 지표 | 정의 | 단위 |
|---|---|---|
| `sun_visible_ratio` | 유효 표본 중 태양 보임 비율 | 0~1 |
| `max_dark_run_h` | 태양 연속 가림 최장 시간 | h |
| `earth_visible_ratio` | 유효 표본 중 지구 보임 비율 | 0~1 |
| `max_earth_gap_h` | 지구 연속 가림 최장 시간 | h |
| `both_visible_h` | 둘 다 보이는 총 시간 | h |
| `valid_ratio` | 전체 표본 중 INVALID가 아닌 비율 | 0~1 |

- 모든 시간 지표의 오차 한계는 ±Δt이며 응답에 `sample_step_min`을 넣는다.
- 연속 가림이 구간 끝까지 이어지면 `*_run_open_ended = true`를 붙인다(하한값이라는 뜻).

### 5.5 임무 시작 시점 탐색 (`windows.py`)

입력: 시작 범위 `[t0, t1]`, 시작 간격(기본 1 h), 임무 기간 D, 한계 `max_dark_h`, `max_earth_gap_h`, 규칙.

1. `[t0, t1 + D]` 전체의 가시성 시계열을 한 번만 계산한다.
2. 가림 구간을 run-length로 압축한다.
3. 각 시작 시점 s에 대해 창 `[s, s + D]`와 겹치는 가림 구간의 (창 안으로 자른) 최장 길이를 구한다.
4. 판정:
   - `INDETERMINATE`: 창의 `valid_ratio < 0.95`, 또는 한계값과 최장 가림 시간의 차이가 Δt 이하
   - `VIOLATION`: 암흑 또는 지구 단절이 한계를 초과. `reasons`에 어느 조건이 얼마나 초과했는지 기록
   - `FEASIBLE`: 그 외
5. 응답에 다음 요약을 포함한다: 일조 비율 최고 시점, 지구 가시 비율 최고 시점, FEASIBLE 연속 구간 목록. 단일 종합 점수는 만들지 않는다.
6. FEASIBLE이 하나도 없으면 "조건별 최소 위반량"(어떤 한계를 얼마로 풀면 후보가 생기는지)을 반환한다.

### 5.6 불확실성 (`uncertainty.py`)

1. 실현 자료 k에 대해 근거리 DEM을 만든다: `DEM_k = LDEM + err_k` `[확인 — §3.1]`.
2. 관측자 고도 `z0_k`도 `DEM_k`에서 다시 읽는다.
3. `H_near_k`만 재계산하고 `H_k = max(H_near_k, H_far)`.
4. §5.4, §5.5를 반복한다.
5. 반환: 실현 자료별 지표, 시작 시점별 `feasible_count / k`.

표시 문구는 "실현 자료 k개 중 n개에서 조건 충족"으로 고정한다. 확률(%)로 바꾸지 않는다. 원거리 지형은 변동시키지 않았다는 한계를 응답의 `limitations`에 넣는다.

목표 k: 최소 3, 권장 10, 확장 100.

---

## 6. API 명세

공통: `Content-Type: application/json`, 오류는 `{ "error": { "code", "message" } }`. 모든 응답에 `meta`를 붙인다.

```json
"meta": {
  "frame": "MOON_ME",
  "sample_step_min": 10,
  "dem_ids": ["site01_5m", "lola_80s_far"],
  "kernels": ["naif0012.tls", "de440s.bsp", "..."],
  "computed": true,
  "limitations": ["..."]
}
```

### GET /health
`{ "status": "ok", "kernels_loaded": true, "dems_loaded": ["site01", "site04"] }`

### GET /sites
```json
[{ "id": "site01", "name": "Connecting Ridge",
   "point": { "lat": -89.45, "lon": 222.69 },
   "dem_bounds_m": { "xmin": 0, "xmax": 0, "ymin": 0, "ymax": 0 },
   "realizations_available": 100 }]
```
(좌표 숫자는 형식 예시다. 실제 값은 §3.3에서 정한다.)

### POST /visibility
요청:
```json
{ "site_id": "site01", "start_utc": "2026-12-01T00:00:00Z",
  "end_utc": "2026-12-04T00:00:00Z", "observer_height_m": 1.0,
  "step_min": 10, "sun_rule": "any_limb", "earth_rule": "center",
  "realization": null }
```
응답:
```json
{ "times_utc": ["..."],
  "sun":   { "az_deg": [], "el_deg": [], "margin_deg": [], "state": [] },
  "earth": { "az_deg": [], "el_deg": [], "margin_deg": [], "state": [] },
  "horizon": { "az_deg": [], "h_deg": [], "valid": [] },
  "metrics": { "sun_visible_ratio": 0.0, "max_dark_run_h": 0.0,
               "max_dark_run_open_ended": false,
               "earth_visible_ratio": 0.0, "max_earth_gap_h": 0.0,
               "max_earth_gap_open_ended": false,
               "both_visible_h": 0.0, "valid_ratio": 0.0 },
  "meta": {} }
```
`state` 값: `"FULL" | "PARTIAL" | "HIDDEN" | "INVALID"`.

### POST /windows/search
요청:
```json
{ "site_ids": ["site01", "site04"],
  "search_start_utc": "2026-12-01T00:00:00Z",
  "search_end_utc": "2026-12-31T00:00:00Z",
  "start_step_h": 1, "mission_hours": 72,
  "max_dark_h": 12, "max_earth_gap_h": 6,
  "observer_height_m": 1.0,
  "sun_rule": "any_limb", "earth_rule": "center" }
```
응답:
```json
{ "results": [{ "site_id": "site01",
    "candidates": [{ "start_utc": "...", "verdict": "FEASIBLE",
                     "max_dark_run_h": 0.0, "max_earth_gap_h": 0.0,
                     "sun_visible_ratio": 0.0, "earth_visible_ratio": 0.0,
                     "valid_ratio": 0.0, "reasons": [] }],
    "feasible_ranges": [{ "from_utc": "...", "to_utc": "..." }],
    "best_by_sun": "...", "best_by_earth": "...",
    "relaxation_hint": null }],
  "meta": {} }
```

### POST /uncertainty
요청: `/windows/search` 본문 + `"realizations": [1, 2, 3]` (최대 100, `site_ids`는 1개).
응답:
```json
{ "site_id": "site01", "k": 3,
  "per_realization": [{ "id": 1, "metrics": {} }],
  "candidates": [{ "start_utc": "...", "feasible_count": 2, "baseline_verdict": "FEASIBLE" }],
  "meta": {} }
```

제한: `step_min ≥ 1`, 분석 구간 ≤ 400일, `/uncertainty`는 캐시가 없으면 202와 `job_id`를 반환해도 된다(동기 처리가 30초를 넘는 경우에만 도입).

---

## 7. 프론트엔드 명세

화면 3개, 단일 페이지 탭 구성.

**1) Setup & Windows**
- 좌측 패널: 사이트 체크박스(A, B), 시작 범위(date range), 임무 기간(h), 허용 암흑(h), 허용 지구 단절(h), 장비 높이(m), 판정 규칙 2개, 실행 버튼.
- 우측: 사이트별 시작 시점 띠(가로축 시간, 색: FEASIBLE 초록 / VIOLATION 빨강 / INDETERMINATE 회색). 띠를 클릭하면 사유와 지표가 뜨고 Comparison 탭의 구간이 해당 창으로 맞춰진다.
- FEASIBLE이 없으면 `relaxation_hint`를 문장으로 보여준다.

**2) Comparison**
- 사이트 2개의 가시성 타임라인(태양 행, 지구 행. FULL / PARTIAL / HIDDEN / INVALID 4색).
- 지표 비교표(단위와 분석 기간 표기).
- 수평선 극좌표 또는 방위각-고도 그래프: H(A) 곡선 위에 태양·지구 궤적.
- 장비 높이 두 값 비교 토글(F9).

**3) Reliability & Evidence**
- 실현 자료별 지표 표와 "k개 중 n개 충족" 막대.
- 3×3 격자 판정(F11).
- 데이터 출처, 커널 목록, 프레임, 표본 간격, 판정 규칙, 한계, 검증 상태(테스트 통과 수).
- CSV 다운로드.

UI 규칙:
- 색상 고정: 태양 `#F5A623`, 지구 `#4A90E2`, 지형 `#8B8B8B`.
- `meta.computed = false`인 데이터는 "예시 데이터" 배지를 붙인다.
- 화면의 모든 숫자는 API 응답에서만 온다. 프론트에서 과학 수치를 계산하거나 만들지 않는다.
- NASA 로고와 "NASA 승인"을 암시하는 문구는 쓰지 않는다. 하단에 "Research and educational prototype. Not a flight-certified tool." 표기.

---

## 8. 테스트 명세

테스트는 `backend/tests/`에 두고, 실패한 테스트는 지우지 않고 원인을 고친다.

### 8.1 천체 위치
| ID | 내용 | 통과 기준 |
|---|---|---|
| T-E1 | SPICE vs JPL Horizons: 두 지점, 30일, 6 h 간격의 태양·지구 az/el | 차이 ≤ 0.05° |
| T-E2 | 남극점(lat = −90°)에서 1년간 지구 중심 고도각 범위 (위도 부호·프레임 오류 검출용) | 전 구간 ±8° 안 (위도 칭동 약 ±6.7° 기준) |
| T-E3 | UTC → ET 변환 | 윤초 포함 기준값과 일치 |
| T-E4 | 극점 근처(|lat| > 89.9°) ENU 기저 | NaN 없음, 기저 직교 |

`[확인]` Horizons에서 달 표면 관측자 설정(`CENTER='coord@301'`, `COORD_TYPE`, `SITE_COORD`)과 Horizons가 쓰는 달 프레임. 프레임이 다르면 차이를 문서에 적고 기준을 조정한다.

### 8.2 수평선
| ID | 내용 | 통과 기준 |
|---|---|---|
| T-H1 | 완전 구(고도 0) 합성 DEM, h = 2 m | 전 방위 H = −acos(R/(R+h)) (오차 1e-4°) |
| T-H2 | 거리 d, 높이 Δz의 합성 벽 | 해석값(곡률 포함)과 0.01° 이내 |
| T-H3 | 0°/360° 경계 | H(0°) = H(360°), 보간 연속 |
| T-H4 | NaN 영역 포함 DEM | 해당 방위 `valid = false`, 값 미생성 |
| T-H5 | 장비 높이 증가(1 m → 10 m) | 모든 방위에서 H가 증가하지 않음 |
| T-H6 | 평면 근사 vs 3차원 계산 | 50 km 이상 지형에서 차이가 나타남을 수치로 기록 |

### 8.3 가시성·탐색
| ID | 내용 | 통과 기준 |
|---|---|---|
| T-V1 | 합성 시계열의 3상태 판정 | 기대 상태와 일치 |
| T-V2 | 구간 끝까지 이어지는 가림 | `open_ended = true` |
| T-W1 | 전 구간 가시 | 모든 시작 시점 FEASIBLE |
| T-W2 | 13 h 암흑 1회, 한계 12 h | 해당 창만 VIOLATION, 사유에 초과량 1 h |
| T-W3 | 후보 없음 | `relaxation_hint` 반환 |
| T-W4 | 유효 비율 0.9 | INDETERMINATE |
| T-W5 | Δt 10분 vs 5분 | 판정이 바뀌는 시작 시점 수를 보고 |

### 8.4 통합
- T-I1: `/visibility` → `/windows/search` 응답의 지표가 같은 창에서 일치.
- T-I2: 기준 시나리오(§1.2)를 프론트에서 끝까지 실행(수동 또는 Playwright).
- T-I3: NASA SVS 4930 일조 시각화와 같은 날짜의 정성 비교 1건, 결과를 README에 기록.

---

## 9. 비기능 요구사항

- 성능 목표(측정 후 README에 실제 값 기록): 캐시된 수평선으로 `/visibility` 30일·10분 간격 응답 2초 이내, `/windows/search` 30일·2사이트 5초 이내. 캐시 없는 수평선 계산은 지점당 60초 이내.
- 서버 시작 시 두 기준 지점의 수평선을 미리 계산해 캐시한다.
- 재현성: `scripts/fetch_data.sh` → `docker build` → `docker run`만으로 실행된다.
- 배포: 공개 URL 1개. 장애에 대비해 녹화 데모와 로컬 실행 방법을 README에 둔다.
- 라이선스와 AI 도구 사용 내역을 README에 기록한다.

---

## 10. 작업 분해와 일정

| 시간 | 작업 | 담당 | 산출물 / 게이트 |
|---|---|---|---|
| T+0~3h | 저장소 골격, `fetch_data.sh`, `[확인]` 항목 해소, 지점 선정, API 스키마 고정 | 전원 | `sites.json`, Pydantic 스키마, 프론트 mock 타입 |
| T+3~10h | `ephemeris.py` + T-E1~4 | 과학 계산 | **게이트 G1:** T-E1 통과 |
| T+3~10h | `terrain.py`: DEM 로딩·좌표 변환, 합성 DEM 생성기 | 지형 | T-H1용 픽스처 |
| T+3~10h | 프론트 골격, 입력 패널, mock 데이터로 차트 | 프론트 | 3개 탭 렌더 |
| T+10~18h | `horizon.py` 근거리+원거리, 캐시, T-H1~6 | 지형 | **게이트 G2:** 두 지점의 실제 H(A) |
| T+10~18h | `visibility.py`, `/visibility` | 과학 계산 + 백엔드 | F4 |
| T+18~25h | `windows.py`, `/windows/search`, 프론트 연동 | 백엔드 + 프론트 | **게이트 G3:** 기준 시나리오 E2E |
| T+25~31h | `uncertainty.py`, Reliability 탭, CSV | 지형 + 프론트 | F10 (k ≥ 3) |
| T+31~35h | 통합 테스트, Horizons·SVS 교차 검증 기록, 성능 측정 | QA + 전원 | README 검증 절 |
| T+35~38h | 배포, 데모 영상, 제출 페이지 | 리더 + QA | 제출 완료 |
| T+38~39h | 외부 접근 확인, 예비 | 전원 | — |

게이트 실패 시:
- G1 실패(T+10h): 시각화 작업을 멈추고 전원이 천체 위치 문제에 붙는다.
- G2 실패(T+18h): 원거리 DEM을 빼고 근거리만으로 진행하되 `valid = false` 방위를 그대로 표시한다. 지형 결과를 만들어 넣지 않는다.
- G3 실패(T+25h): P2를 전부 취소하고 F1~F6 안정화.
- T+31h에 F10이 검증되지 않았으면 "실험 기능" 배지를 붙이거나 뺀다.
- T+35h 이후 새 기능 금지.

---

## 11. 완료 정의 (Definition of Done)

- [ ] P0 기능 F1~F6이 실제 NASA 데이터로 동작한다.
- [ ] §8의 P0 관련 테스트(T-E*, T-H*, T-V*)가 통과하고 통과·실패 수가 README에 있다.
- [ ] 기준 시나리오가 공개 URL에서 끝까지 실행된다.
- [ ] 화면의 모든 수치에 단위와 기간이 있고, 예시 데이터는 배지로 구분된다.
- [ ] README에 데이터 출처, 커널·DEM 버전, 가정, 한계, AI 사용 내역이 있다.
- [ ] 제출 페이지에 공식 챌린지 Resources에 포함된 NASA 자료의 사용 사실이 적혀 있다.
- [ ] 마감 1시간 전에 제출이 끝나 있다.

---

## 12. 미해결 항목

| # | 항목 | 담당 | 기한 |
|---|---|---|---|
| O1 | `*_err.tif`의 정의(오차장 vs 완성 DEM)와 기준 반지름 | 지형 | 대회 전 (문서 조사) |
| O2 | 원거리 DEM 파일과 다운로드 경로 확정 | 지형 | 대회 전 (문서 조사) |
| O3 | SPICE 커널 최신 파일명, MOON_ME 프레임 이름 | 과학 계산 | 대회 전 (문서 조사) |
| O4 | Horizons의 달 표면 관측자 설정과 프레임 | 과학 계산 | 대회 전 (문서 조사) |
| O5 | 공식 챌린지 Resources 목록에 PGDA·Horizons 포함 여부 | 리더 | 11-13 |
| O6 | 2026 제출 가이드(영상 길이·형식) | QA | 11-13 공개 후 |
| O7 | 제품명: "LunarSafe"가 "안전 보장 안 함" 원칙과 충돌. 대안 예: LunarWindow | 리더 | T+0 |
| O8 | 배포 플랫폼 | 백엔드 | T+0 |

## 13. 참고 자료

- NASA PGDA Product 78 (달 남극 고해상도 LOLA 지형): https://pgda.gsfc.nasa.gov/products/78
- NAIF SPICE: https://naif.jpl.nasa.gov/naif/
- JPL Horizons API: https://ssd-api.jpl.nasa.gov/doc/horizons.html
- NASA SVS 달 남극 일조 시각화: https://svs.gsfc.nasa.gov/4930/
- Mazarico, E. et al. (2011), "Illumination conditions of the lunar polar regions using LOLA topography", *Icarus* 211
- 공식 챌린지: https://www.spaceappschallenge.org/2026/challenges/lunar-landing-site-navigator/
