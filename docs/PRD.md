# LunarSafe PRD v2.0

| 항목 | 내용 |
|---|---|
| 제품명 | LunarSafe (가칭, §13 O1) |
| 대회 / 과제 | NASA Space Apps Challenge 2026 / **Lunar Landing Site Navigator** (Advanced · Intermediate) |
| 문서 버전 | v2.0 (2026-11-12). v1.0을 대체한다 |
| 개발 가능 시간 | 2026-11-14 09:00 ~ 11-15 23:59 (현지 시간, 약 39시간) |
| 상태 | 기획 확정. **코드와 데이터 가공은 11-14 09:00 이후에만 시작한다** |

표기 규칙
- `[검증됨]` 2026-11-12에 원문(공식 페이지, NASA 데이터 페이지, NAIF 디렉터리)으로 확인한 사실.
- `[확인 필요]` 원문으로 확인하지 못한 값. 담당자가 기한 안에 확인하고 이 문서를 고친다. 확인 전에는 코드에 상수로 넣지 않는다.

---

## 0. 한 문장

> **착륙선은 찍은 점에 내리지 않고, 지형도는 실제 지형과 똑같지 않다.** LunarSafe는 누구나 30초 안에 착륙지와 날짜를 비교하게 해 주고, 그 답이 착륙 오차와 지형 오차를 넣어도 유지되는지 보여준다.

---

## 1. 과제 요구사항과 대응 `[검증됨]`

출처: 공식 과제 페이지 https://www.spaceappschallenge.org/2026/challenges/lunar-landing-site-navigator/ (2026-11-12 확인, 최종 수정 2026-10-28).

과제의 핵심 문장: 임무 기획자, 교육자, 일반인이 **착륙지와 날짜를 빠르게 비교**하고, 수평선 대비 태양·지구 위치를 시각화해 **발전 가능성과 지구 직접 통신 창**을 평가하는 **직관적인** 도구를 만든다.

배경에서 강조하는 점
- CLPS와 그 밖의 달 임무가 대상이다. 남극이 특히 어렵다.
- 일정은 고정되지 않는다. 개발 지연·발사 지연이 생기면 **같은 지역의 다른 지점, 같은 지점의 다른 날짜**를 빨리 다시 평가해야 한다.
- 기존 도구(Quickmap, Moon Trek, JPL Horizons)는 비전문가가 빠른 비교에 쓰기 어렵다.

| 과제 항목 (원문 요지) | 대응 기능 | 등급 |
|---|---|---|
| 선별된 착륙지 목록(Apollo, CLPS 목표, 타 상업·타국 임무) + 임의 좌표 입력 | F1 | P0 |
| 여러 지점의 나란히 비교 | F5 | P0 |
| 착륙 날짜·시각 선택, 달의 하루·계절에 따른 변화 탐색 | F2, F6 | P0 |
| 수평선 대비 태양·지구 위치 표시 | F3, F4 | P0 |
| **일몰·지구몰까지 남은 시간** | F4 | P0 |
| 태양 고도각을 눈에 띄게 표시, 일조 지속 시간, 지구 가시 창 | F4 | P0 |
| 2D·3D 등 다양한 시각화, 애니메이션·타임라인 | F5, F12 | P0 / P2 |
| 대기가 없으므로 수평선은 지형과 달 구면으로 결정 | §6.3 | P0 |
| 실제 지형(LOLA) 반영, 남극의 급경사·깊은 크레이터 | §6.3 | P0 |
| 지구 칭동, 달 궤도 변화 반영 | SPICE 사용으로 자동 반영 | P0 |
| **작은 위치·날짜 변화가 왜 큰 영향을 주는지 이해시키는 기능** | **F8, F9, F10 (핵심 차별점)** | P1 |
| 일정 변경 시 빠른 재평가 | F7 (일정 지연 슬라이더) | P1 |
| 전문가와 일반인 모두 사용 | 2단 화면(§8) | P0 |

Resources 탭은 **2026-11-13에 공개**된다고 페이지에 적혀 있다. 글로벌 심사 자격을 위해 그 목록의 NASA 자료를 최소 1개 쓰고 제출 페이지에 명시한다(§13 O2).

---

## 2. 전략

### 2.1 경쟁 상황 `[검증됨]`

같은 과제를 준비하는 공개 저장소를 8개 이상 확인했다(예: LightPath, CLPS-Lunar-Mission-Website, lunar-horizon-wheel, SelenSync, LunarSite Compass). 이들이 이미 다루는 것: LOLA 지형 수평선, 원반 부분 가림, DSN 지상국 접속, 날짜 탐색, Horizons 교차 검증, 3D 달.

따라서 이 기능들은 **기본 요건**이고 차별점이 아니다.

### 2.2 차별점

확인한 저장소는 모두 **한 좌표, 한 지형도**에 대해 답한다. LunarSafe는 두 가지 불확실성을 넣는다.

1. **착륙 위치 오차.** 2025년 IM-2(Athena)는 Mons Mouton 목표점에서 벗어나 크레이터 안에 내렸고, 태양 전지로 충전하지 못해 10일 계획 임무가 13시간 안에 끝났다. 벗어난 거리는 Intuitive Machines 발표로 250 m, NASA 발표로 400 m 이상이다. `[검증됨: SpaceNews, SpacePolicyOnline, Intuitive Machines 2025-03-07]`
2. **지형도 오차.** NASA GSFC는 남극 후보지 5 m DEM과 함께 오차 특성이 같은 지형 실현 자료(clone) 100개를 제공한다. `[검증됨: PGDA Product 78]`

결과물은 도구 하나와 수치 하나다.
- 도구: 과제 요건을 모두 충족하는 직관적인 비교 화면.
- 수치: **"한 점·한 지형 기준으로 가능하다고 나온 임무 창 중 몇 %가, 착륙 오차와 지형 오차를 넣으면 깨지는가."**

수치는 계산해서 나온 값을 그대로 보고한다. 작게 나오면 "두 후보지 중 어느 쪽이 더 견고한가"를 결론으로 삼는다. 값을 만들거나 유리한 조건만 고르지 않는다.

### 2.3 선행 연구와의 관계 (발표에서 먼저 밝힌다)

Barker et al. (2021, *Planetary and Space Science* 203, 105119)이 같은 DEM과 clone 100개로 **일조 조건**(평균 일조, 최장 낮, 최장 밤)의 불확실성을 이미 분석했다. 관측 높이 5 m에서는 DEM 오차의 영향이 크게 줄어든다는 결과도 있다. `[검증됨: PGDA 78 설명, NASA NTRS 발표 자료]`

LunarSafe가 새로 하는 일
- 일조뿐 아니라 **지구 가시성과 DSN 접속**에 같은 방법을 적용한다.
- 지표가 아니라 **사용자가 정한 임무 창의 성립 여부**로 바꿔 보여준다.
- **착륙 위치 오차**를 함께 넣는다.
- 논문이 아니라 누구나 쓰는 **대화형 도구**로 만든다.

"최초의 불확실성 분석"이라고 말하지 않는다. "NASA가 공개한 방법과 데이터를 임무 창 판단에 적용했다"고 말한다. PGDA 페이지도 이 방법이 일조 외의 착륙지 타당성 연구에 쓸 수 있다고 적고 있다.

### 2.4 하지 않는 것

착륙 궤도·추진제, 구조 안전성, 열·배터리 상세 설계, 링크 버짓, 비행 승인용 결과. "착륙 안전", "성공 확률"이라는 표현은 쓰지 않는다.

---

## 3. 사용자와 기준 시나리오

- 일반인·교육자: 지점과 날짜를 고르면 "지금 태양과 지구가 어디 있고, 해 지기까지 얼마 남았는가"를 바로 본다.
- 임무 기획자: 일정이 밀렸을 때 대안 날짜·지점을 비교하고, 그 판단이 얼마나 견고한지 본다.

데모 시나리오(영상과 동일)
1. 목록에서 Mons Mouton(IM-2 목표 지역)과 Connecting Ridge를 고른다.
2. 임무 기간 10일, 허용 연속 암흑 12 h, 허용 연속 지구 단절 6 h, 장비 높이 1 m를 둔다. (IM-2의 계획 기간이 10일이었다. 나머지는 예시값이며 NASA 요구조건이 아니다.)
3. 타임라인에서 태양·지구 가시 구간과 일몰·지구몰까지 남은 시간을 본다.
4. "일정 2주 지연" 슬라이더를 움직여 창이 어떻게 바뀌는지 본다.
5. "Robustness"를 켠다. 착륙 원(반경 250 m) 안의 지점들과 지형 실현 자료들에서 창이 유지되는 비율이 지도와 숫자로 나온다.

---

## 4. 기능과 완료 기준

| ID | 기능 | 등급 | 완료 기준 |
|---|---|---|---|
| F1 | 지점 목록 + 임의 좌표 | P0 | `GET /sites`가 목록(§5.3)을 반환, 임의 위경도 입력 가능, 각 지점에 사용 가능한 지형 단계(§6.3) 표기 |
| F2 | 태양·지구 방위각/고도각 시계열 | P0 | Horizons 대비 0.05° 이내 (T-E1) |
| F3 | 지형 수평선 H(A) | P0 | 720방위, 유효 플래그 포함, T-H1~H6 통과 |
| F4 | 가시성 지표 | P0 | 태양 고도각, 일조·지구 가시 비율, 최장 암흑·단절, **일몰·지구몰까지 남은 시간** 반환 (§6.5) |
| F5 | 비교 화면 | P0 | 2~3개 지점의 타임라인·지표·수평선 그래프가 한 화면에 표시 |
| F6 | 기간 탐색 | P0 | 최대 400일 범위에서 날짜 이동, 달의 하루 단위 보기 |
| F7 | 임무 창 탐색 + 일정 지연 | P1 | 조건 입력 → 시작 시점별 `FEASIBLE / VIOLATION / INDETERMINATE`와 사유, 지연 슬라이더로 즉시 갱신 |
| F8 | 착륙 위치 민감도 | P1 | 착륙 원 안 격자 지점별 판정 지도 |
| F9 | 지형 실현 자료 민감도 | P1 | clone K개에 대한 판정, "K개 중 n개" 표기 |
| F10 | 견고성 요약 | P1 | 시작 시점별 견고성 비율과 §6.7의 요약 수치 |
| F11 | DSN 지상국 접속 | P1 | 지구 가시 + 지상국 3곳 중 1곳 이상에서 달이 마스크 고도 이상 |
| F12 | 원반 3상태, 장비 높이 비교 | P1 | `FULL / PARTIAL / HIDDEN`, 높이 2값 나란히 비교 |
| F13 | 3D 보기, 애니메이션 | P2 | 수평선 파노라마 위 태양·지구 이동 재생 |
| F14 | IM-2 사례 | P2 | 목표점과 실제 착륙점의 계산 결과 비교 (§10.4의 한계 표기 포함) |
| F15 | CSV 내보내기 | P2 | 시계열과 조건 다운로드 |

순서 원칙: P0가 끝나기 전에 P1을, P1의 F7~F10이 끝나기 전에 P2를 시작하지 않는다. 계산하지 못한 값은 "계산 불가"로 표시한다.

---

## 5. 데이터

### 5.1 지형

| 단계 | 자료 | 사양 | 출처 |
|---|---|---|---|
| 근거리 | PGDA Product 78 사이트 DEM `SiteNN_final_adj_5mpp_surf.tif` | 5 m/pix GeoTIFF, 남극 평사 X/Y(m), MOON_ME(DE421), 값은 표면 높이 Z(m) `[검증됨]` | https://pgda.gsfc.nasa.gov/products/78 |
| 근거리 불확실성 | `SiteNN_final_adj_5mpp_toterr.tif` | 총 Z 불확실성(RMS, m) `[검증됨]` | 같음 |
| 지형 실현 자료 | `SiteNN_final_adj_5mpp_0001_err.tif` ~ `_0100_err.tif` | 사이트당 100개 `[검증됨]`. 사용법은 아래 | 같음 |
| 중거리 | `LDEM_80S_40MPP_ADJ.TIF` (685 MB) 또는 `LDEM_80S_20MPP_ADJ.TIF` (2.6 GB) | 80°S 이남, 평사 범위 ±304 km `[검증됨]` | https://pgda.gsfc.nasa.gov/products/90 |
| 원거리 | `LDEM_60S_240MPP_ADJ.TIF` (210 MB) | 60°S 이남, 평사 범위 ±931.2 km `[검증됨]` | 같음 |

clone 사용법 `[확인 필요, 기한 T+1h]`
- NASA 발표 자료의 도식은 "LDEM + Z 오차장 = clone"이다. 따라서 `_err.tif`는 **기준 DEM에 더하는 오차장**일 가능성이 높다.
- 확인 방법: 파일 하나를 열어 값 범위를 본다. 수 m 이내면 오차장(`DEM_k = surf + err_k`), km 단위면 완성 DEM(`DEM_k = err_k`)이다. 결과를 `data/MANIFEST.json`에 기록한다.

그 밖에 `[확인 필요, 기한 T+1h]`
- 기준 반지름: PGDA 페이지에는 적혀 있지 않다. GeoTIFF의 CRS에서 읽는다(LOLA 관례는 1737.4 km).
- Site20의 범위와 IM-2 착륙 좌표(§5.3)가 그 안에 드는지.
- 각 사이트 DEM의 크기(초기 4개 사이트는 16 × 16 km).

처리 규칙
- 좌표 변환은 파일의 CRS를 `rasterio`/`pyproj`로 읽어서 한다. 투영식을 직접 구현하지 않는다.
- nodata와 범위 밖은 NaN으로 두고 채우지 않는다.
- 중·원거리 DEM을 줄여 쓸 때는 블록 최댓값 배열을 쓴다(차폐를 과소평가하지 않기 위함. README에 명시).

### 5.2 천체력과 자세 (SPICE) `[검증됨: NAIF generic_kernels 디렉터리]`

| 종류 | 파일 | 용도 |
|---|---|---|
| LSK | `naif0012.tls` | UTC ↔ ET |
| SPK | `de440s.bsp` | 태양·지구·달 위치 |
| PCK | `pck00011.tpc` | 천체 반지름 |
| 달 자세 | `moon_pa_de440_200625.bpc` | 달 주축 프레임 |
| 달 프레임 | `moon_de440_250416.tf` | `MOON_ME_DE440_ME421` 정의 (DE421 ME 프레임과 정렬. LOLA 자료의 프레임과 맞춘다) |
| 지구 자세 | `earth_latest_high_prec.bpc`, `earth_2026_260806_2126_predict.bpc` | DSN 지상국용 `ITRF93`. predict를 먼저, high_prec를 나중에 로드한다 |

- 달 고정 프레임 이름은 코드 전체에서 `MOON_ME_DE440_ME421` 하나만 쓴다.
- `naif0012.tls`, `de440s.bsp`의 디렉터리 위치는 `fetch_data.sh` 작성 때 확인한다 `[확인 필요]`.
- 커널과 DEM은 저장소에 넣지 않는다. `scripts/fetch_data.sh`로 받고 파일명과 SHA-256을 `data/MANIFEST.json`에 기록한다.

### 5.3 지점 목록

| 구분 | 지점 | 지형 단계 | 불확실성 분석 |
|---|---|---|---|
| 남극, clone 있음 | Connecting Ridge (PGDA Site01) | 근·중·원 | 가능 |
| 남극, clone 있음 | Mons Mouton / Leibnitz beta plateau (PGDA Site20) | 근·중·원 | 가능 |
| 남극 | IM-2 Athena 착륙점 | 중·원 (Site20 범위 안이면 근 포함) | 범위 안일 때만 |
| 남극 | IM-1 Odysseus, Shackleton Rim (Site04), Nobile Rim (Site06) | 가능한 단계 | Site04·06은 가능 (P2) |
| 그 밖 | Apollo 11·15·17, Chandrayaan-3, Blue Ghost M1 등 | 구면 수평선 | 불가 |
| 임의 좌표 | 사용자 입력 | 위도에 따라 자동 결정 | 불가 |

- 모든 좌표는 `[확인 필요, 기한 T+2h]`. NASA NSSDCA 또는 LROC 공식 자료에서 가져오고 출처 URL을 `sites.json`에 함께 적는다. 기억이나 2차 자료의 숫자를 넣지 않는다.
- IM-2 착륙점은 2차 자료 기준 84.7906°S, 29.1957°E, 착륙 시각 2025-03-06 17:28:50 UTC다. LROC 발표로 재확인한다.
- Site01·Site20의 기준 분석 지점은 DEM을 연 뒤 능선·고원의 유효 격자에서 고르고 선택 근거를 기록한다.
- 80°S보다 북쪽의 지점은 구면 수평선만 쓰고 화면에 "지형 미반영"을 표시한다.

### 5.4 DSN 지상국

Goldstone, Madrid, Canberra 3곳. 좌표와 고도 마스크는 `[확인 필요, 기한 T+2h]` (DSN 810-005 문서). 마스크 기본값은 문서 값으로 하고 UI에서 바꿀 수 있게 한다.

---

## 6. 계산 명세

### 6.1 규약

- 경도: 동경 양수 [0°, 360°). 위도: 남위 음수. 방위각: 북 0°, 동 90°. 고도각: 수평 0°.
- 시간: API는 ISO 8601 UTC, 내부는 ET.
- 내부 각도는 라디안, API와 화면은 도.

### 6.2 천체 위치 (`ephemeris.py`)

입력: 위도, 경도, 지형 높이 z0, 장비 높이 h, ET 배열.

1. 관측자 `p = (R + z0 + h) · û(lat, lon)`.
2. `d = spkpos(target, et, "MOON_ME_DE440_ME421", "LT+S", "MOON") − p`. target은 `SUN`, `EARTH`.
3. ENU 기저: `up = û`, `east = ẑ × up` 정규화, `north = up × east`. |lat| > 89.9°이면 `east = (−sin lon, cos lon, 0)`.
4. `el = asin(d·up / |d|)`, `az = atan2(d·east, d·north) mod 2π`.
5. 각반지름 `ρ = asin(R_body / |d|)`. 반지름은 `pck00011.tpc`에서 `bodvrd`로 읽는다.

### 6.3 수평선 (`horizon.py`)

출력: 방위각 720개(0.5°)의 `H`, `valid`.

대기가 없으므로 수평선은 지형과 달 구면으로만 정해진다. 모든 계산은 3차원 벡터로 하므로 곡률이 자동으로 들어간다.

| 대역 | 거리 | 자료 | 표본 간격 | 재계산 단위 |
|---|---|---|---|---|
| near | 5 m ~ 사이트 DEM 경계(최대 약 8 km) | 5 m DEM (+ clone) | 2 km까지 5 m, 이후 20 m | 지점마다, clone마다 |
| mid | near 끝 ~ 50 km | 40 m DEM | 40 m | 지점마다 |
| far | 50 ~ 300 km | 240 m DEM | 240 m | 지점마다 |

- 표본 지점 → DEM 쌍선형 보간 → `q = (R + z) · û` → `e = asin((q − p)·up / |q − p|)` → 대역별 `max`.
- `H = max(H_near, H_mid, H_far)`. 지형이 없는 방위의 하한은 구면 수평선 `−acos(R / (R + z0 + h))`.
- 유효성: 한 방위에서 NaN 표본이 5%를 넘으면 `valid = false`.
- 5 m DEM이 없는 남극 지점은 near를 20 m 또는 40 m DEM으로 대신하고 화면에 해상도를 표기한다.
- mid와 far는 지점마다 계산하고 clone마다 다시 계산하지 않는다. 대신 방위별로 수평선을 만든 표본의 거리 `D`를 함께 저장하고, clone에서 관측자 높이가 `Δz0`만큼 달라지면 1차 보정 `H' = H − Δz0 / D`를 적용한다. 보정 오차는 T-H7로 실측한다.
- far를 지점마다 계산하는 이유: 250 m 이동이 50 km 밖 높이 차 5 km 지형의 고도각을 약 0.03° 바꾼다(ΔH·Δd/d²). 태양 반지름 0.27°에 비해 작지만 무시하지 않는다.
- 최대 거리 300 km의 근거: 높이 차 ΔH인 지형이 수평선에 걸리는 한계 거리 ≈ √(2·R·ΔH). ΔH = 8 km이면 약 167 km, 여유를 둔다.
- 캐시 키: `(dem_id, lat, lon, h, clone_id, 대역)`. `cache/*.npz`.

### 6.4 가시성 판정 (`visibility.py`)

- `H_c = interp(H, az)` (0°/360° 원형 보간). 방위 구간 `[az − ρ, az + ρ]`에서 `H_lo`, `H_hi`.
- `FULL`: `el − ρ > H_hi` / `HIDDEN`: `el + ρ < H_lo` / `PARTIAL`: 그 외 / `INVALID`: 구간에 무효 방위 포함.
- 규칙: `any_limb`(FULL·PARTIAL을 보임으로, 태양 기본), `center`(`el > H_c`, 지구 기본), `full_disk`(FULL만).
- DSN(F11): `ITRF93`에서 지상국별 달 고도각을 구해 마스크 이상이면 접속 가능. `dte = earth_visible AND (지상국 1곳 이상 접속 가능)`.

### 6.5 지표

표본 간격 Δt 기본 10분. 모든 시간 지표에 ±Δt 오차를 표기한다.

| 지표 | 정의 |
|---|---|
| `sun_el_deg` | 현재 태양 중심 고도각 (화면 상단에 크게) |
| `time_to_sunset_h`, `time_to_earthset_h` | 현재 시각부터 다음 가림 시작까지. 분석 구간 안에 없으면 `null`과 "구간 내 없음" |
| `sun_visible_ratio`, `earth_visible_ratio`, `dte_ratio` | 유효 표본 중 비율 |
| `max_dark_run_h`, `max_earth_gap_h` | 최장 연속 가림. 구간 끝까지 이어지면 `open_ended = true` |
| `both_visible_h` | 태양과 지구가 함께 보이는 총 시간 |
| `valid_ratio` | INVALID가 아닌 표본 비율 |

### 6.6 임무 창 탐색 (`windows.py`)

1. `[t0, t1 + D]`의 가시성 시계열을 한 번 계산하고 가림 구간을 run-length로 압축한다.
2. 시작 시점 s(기본 1 h 간격)마다 창 `[s, s + D]` 안의 최장 암흑·단절을 구한다.
3. `INDETERMINATE`: `valid_ratio < 0.95` 또는 한계와의 차이가 Δt 이하. `VIOLATION`: 한계 초과, 초과량을 `reasons`에 기록. `FEASIBLE`: 그 외.
4. 후보가 없으면 "어느 한계를 얼마로 풀면 후보가 생기는지"를 반환한다.
5. 일정 지연(F7): 같은 시계열을 재사용해 `t0`만 옮긴다. 재계산하지 않는다.

### 6.7 견고성 (`robustness.py`)

입력: 사이트, 기준 지점, 착륙 원 반경 r(기본 250 m), 격자 간격 g(기본 50 m), clone 목록(K개), 임무 조건.

1. 착륙 원 안의 격자 지점 P개를 만든다(r = 250, g = 50이면 약 80개).
2. 각 지점 i: `H_mid_i`, `H_far_i`와 방위별 거리 `D`를 계산한다(clone 무관).
3. 각 (i, k): `DEM_k`에서 `z0`를 다시 읽고 `H_near_ik`를 계산한다. mid·far에는 §6.3의 `Δz0` 보정을 적용한다. `H_ik = max(H_near_ik, H_mid_ik', H_far_ik')`.
4. 각 (i, k)에 §6.4~6.6 적용.
5. 시작 시점 s의 견고성 `R(s) = (FEASIBLE인 (i, k) 수) / (P × K)`.

요약 수치
- `nominal_feasible`: 기준 지점·기준 DEM에서 FEASIBLE인 시작 시점 수.
- `fragile_share`: 그중 `R(s) < 0.9`인 비율. **발표의 핵심 수치.**
- `robust_windows`: `R(s) ≥ 0.9`인 연속 구간.
- 위치만 바꾼 경우(K = 기준 DEM만)와 지형만 바꾼 경우(P = 기준 지점만)의 `fragile_share`를 따로 낸다. 어느 쪽 영향이 큰지 보여준다.

표기 규칙
- "지점 P개 × 실현 K개 = N개 조합 중 n개에서 조건 충족"으로 쓴다. 확률(%)이나 성공률로 부르지 않는다.
- 착륙 원은 실제 착륙 분산 모델이 아니라 균일 격자다. 반경 250 m는 IM-2의 보고된 이탈 거리에서 가져온 예시값이라고 화면에 적는다.
- 중·원거리 지형 자체는 변동시키지 않았다고(관측자 높이 보정만 적용) `limitations`에 적는다.
- 임계값 0.9는 UI에서 바꿀 수 있고, 결과를 본 뒤 유리하게 조정하지 않는다.

연산량 추정 (측정 전 추정치. T+14h에 실측으로 교체)
- near 수평선 1회: 720방위 × 약 700표본 ≈ 50만 보간 → NumPy로 0.1 s 안팎.
- P = 80, K = 100 → near 8,000회 ≈ 15분(1코어). 8코어면 수 분. mid·far는 지점당 1회씩 80회라 1분 안팎.
- 목표 K: 최소 10, 권장 30, 확장 100. 시간이 모자라면 K를 줄이고 사용한 K를 표시한다.

---

## 7. API

공통: JSON. 모든 응답에 `meta`(프레임, 표본 간격, 사용 DEM·커널, 지형 단계, `computed`, `limitations`)를 붙인다.

| API | 기능 |
|---|---|
| `GET /health` | 커널·DEM 로드 상태 |
| `GET /sites` | 지점 목록, 좌표, 출처, 지형 단계, clone 사용 가능 여부 |
| `POST /visibility` | 지점(목록 id 또는 위경도)·기간·높이·규칙 → 시계열, 수평선, 지표 |
| `POST /windows/search` | 지점 1~3개·탐색 범위·임무 조건 → 시작 시점별 판정과 요약 |
| `POST /robustness` | 사이트·착륙 원·clone 목록·임무 조건 → 격자 지점 판정, `R(s)`, 요약 수치 |
| `GET /robustness/{job_id}` | 긴 작업의 진행률과 결과 |

`POST /visibility` 요청
```json
{ "site": {"id": "site20"}, "start_utc": "2026-12-01T00:00:00Z",
  "end_utc": "2026-12-15T00:00:00Z", "now_utc": "2026-12-03T06:00:00Z",
  "observer_height_m": 1.0, "step_min": 10,
  "sun_rule": "any_limb", "earth_rule": "center", "dsn": true }
```
`site`는 `{"id": ...}` 또는 `{"lat": ..., "lon": ...}`.

응답
```json
{ "times_utc": [], 
  "sun":   {"az_deg": [], "el_deg": [], "margin_deg": [], "state": []},
  "earth": {"az_deg": [], "el_deg": [], "margin_deg": [], "state": [], "dte": []},
  "horizon": {"az_deg": [], "h_deg": [], "valid": []},
  "metrics": {"sun_el_deg": 0, "time_to_sunset_h": null, "time_to_earthset_h": null,
              "sun_visible_ratio": 0, "earth_visible_ratio": 0, "dte_ratio": 0,
              "max_dark_run_h": 0, "max_dark_run_open_ended": false,
              "max_earth_gap_h": 0, "max_earth_gap_open_ended": false,
              "both_visible_h": 0, "valid_ratio": 0},
  "meta": {} }
```

`POST /windows/search` 요청에 추가되는 필드: `sites[]`, `search_start_utc`, `search_end_utc`, `start_step_h`, `mission_hours`, `max_dark_h`, `max_earth_gap_h`, `delay_days`.
응답: 사이트별 `candidates[{start_utc, verdict, max_dark_run_h, max_earth_gap_h, reasons[]}]`, `feasible_ranges[]`, `relaxation_hint`.

`POST /robustness` 요청에 추가되는 필드: `site_id`, `radius_m`, `grid_m`, `clones[]`, `threshold`.
응답
```json
{ "points": [{"dx_m": 0, "dy_m": 0, "lat": 0, "lon": 0, "feasible_share": 0}],
  "candidates": [{"start_utc": "", "nominal_verdict": "FEASIBLE", "robustness": 0, "n": 0, "N": 0}],
  "summary": {"P": 0, "K": 0, "nominal_feasible": 0, "fragile_share": 0,
              "fragile_share_position_only": 0, "fragile_share_terrain_only": 0,
              "robust_windows": []},
  "meta": {} }
```

제한: `step_min ≥ 1`, 분석 구간 ≤ 400일, 지점 ≤ 3, clone ≤ 100.

---

## 8. 화면

원칙: 첫 화면은 일반인이 30초 안에 답을 얻는 화면이다. 깊은 분석은 한 단계 뒤에 둔다.

**화면 1 — Navigator (기본)**
- 왼쪽: 지점 목록 + 좌표 입력 + 날짜·시각 + "일정 지연" 슬라이더.
- 가운데: 수평선 파노라마(방위각 가로축). 지형 실루엣 위에 태양과 지구가 표시되고 시간 슬라이더로 움직인다.
- 위쪽 카드: 태양 고도각, 일몰까지 남은 시간, 지구몰까지 남은 시간, 지구 직접 통신 가능 여부.
- 아래: 지점별 타임라인(태양 행, 지구 행). 지점 2~3개를 위아래로 겹쳐 비교한다.

**화면 2 — Mission Window**
- 임무 기간, 허용 암흑·단절, 장비 높이 입력.
- 시작 시점 띠(FEASIBLE 초록, VIOLATION 빨강, INDETERMINATE 회색), 클릭하면 사유.
- 후보가 없으면 완화 힌트를 문장으로 보여준다.

**화면 3 — Robustness**
- 착륙 원 지도: 격자 지점마다 조건 충족 비율을 색으로 표시.
- 시작 시점 띠 위에 견고성 `R(s)`를 겹쳐 그린다.
- 큰 숫자 하나: `fragile_share`와 그 분모("N개 조합").
- 위치 영향과 지형 영향을 나눈 막대 2개.
- 방법·가정·데이터 출처·검증 상태(테스트 통과 수)·한계.

화면 규칙
- 색: 태양 `#F5A623`, 지구 `#4A90E2`, 지형 `#8B8B8B`. 색만으로 구분하지 않고 무늬나 글자를 함께 쓴다.
- 화면의 숫자는 모두 API 응답에서 온다. `meta.computed = false`이면 "예시 데이터" 배지를 붙인다.
- 지형을 반영하지 않은 지점은 "구면 수평선(지형 미반영)"을 표시한다.
- NASA 로고와 승인을 암시하는 문구를 쓰지 않는다. 하단에 "Research and educational prototype. Not a flight-certified tool."

---

## 9. 구조

```
frontend/  Vite + React + TypeScript, Plotly
backend/   Python 3.11, FastAPI
  app/ephemeris.py  terrain.py  horizon.py  visibility.py
      windows.py  robustness.py  dsn.py  api.py  sites.json
  tests/
scripts/fetch_data.sh
data/   (gitignore)   cache/   (gitignore)
docs/PRD.md   Dockerfile   README.md
```

- 의존성: `fastapi`, `uvicorn`, `numpy`, `scipy`, `spiceypy`, `rasterio`, `pyproj`, `pytest`, `httpx`.
- 계산 모듈은 FastAPI를 import하지 않고 NumPy 배열만 주고받는다.
- 견고성 계산은 `multiprocessing`으로 지점 단위 병렬화한다.
- 서버 시작 시 Site01·Site20 기준 지점의 수평선을 미리 계산한다. 데모용 견고성 결과도 미리 계산해 캐시하고, 화면에는 실제 계산값임을 표시한다.

---

## 10. 검증

실패한 테스트는 지우지 않고 원인을 고친다. 통과하지 못한 항목은 README에 그대로 적는다.

### 10.1 천체 위치
| ID | 내용 | 기준 |
|---|---|---|
| T-E1 | SPICE vs JPL Horizons, 2개 지점, 30일, 6 h 간격 | 태양·지구 az/el 차이 ≤ 0.05° |
| T-E2 | 남극점에서 1년간 지구 중심 고도각 | 전 구간 ±8° 안 (위도 칭동 약 ±6.7°) |
| T-E3 | UTC → ET | 기준값 일치 |
| T-E4 | 극 근처 ENU 기저 | NaN 없음, 직교 |

Horizons 설정 `[확인 필요, 기한 T+3h]`: 달 표면 관측자는 `CENTER='coord@301'`, `COORD_TYPE='GEODETIC'`, `SITE_COORD='동경,위도,고도(km)'`, 방위각·고도각은 `QUANTITIES='4'`로 예상한다. Horizons가 쓰는 달 프레임이 ME인지 확인하고, 다르면 차이를 문서에 적고 기준을 조정한다.

### 10.2 수평선
| ID | 내용 | 기준 |
|---|---|---|
| T-H1 | 완전 구 합성 DEM, h = 2 m | 전 방위 `−acos(R/(R+h))`, 오차 1e-4° |
| T-H2 | 거리 d, 높이 Δz의 합성 벽 | 곡률 포함 해석값과 0.01° 이내 |
| T-H3 | 0°/360° 경계 | 연속 |
| T-H4 | NaN 포함 DEM | 해당 방위 `valid = false` |
| T-H5 | 장비 높이 1 m → 10 m | 모든 방위에서 H가 커지지 않음 |
| T-H6 | 대역 경계 | near/mid/far 경계에서 표본 누락 없음 |
| T-H7 | `Δz0` 1차 보정 vs mid·far 전체 재계산 (Δz0 = ±1 m) | 차이 ≤ 0.001° |

### 10.3 가시성·탐색·견고성
| ID | 내용 | 기준 |
|---|---|---|
| T-V1 | 합성 시계열의 3상태 판정 | 기대값 일치 |
| T-V2 | 일몰까지 남은 시간 | 합성 시계열에서 Δt 이내 |
| T-V3 | 구간 끝까지 가림 | `open_ended = true` |
| T-W1~W4 | 전부 가능 / 13 h 암흑(한계 12 h) / 후보 없음 / 유효 비율 0.9 | 각각 FEASIBLE / 해당 창만 VIOLATION(초과 1 h) / 완화 힌트 / INDETERMINATE |
| T-W5 | Δt 10분 vs 5분 | 판정이 바뀌는 시작 시점 수 보고 |
| T-R1 | P = 1, K = 기준 DEM | 견고성 결과가 `/windows/search`와 동일 |
| T-R2 | 모든 clone이 0인 합성 오차장 | `fragile_share_terrain_only = 0` |
| T-R3 | 격자 대칭성 | 평탄 합성 지형에서 모든 지점 판정 동일 |

### 10.4 외부 자료와 대조
- T-X1: NASA SVS 4930(남극 일조 시각화)와 같은 날짜의 정성 비교 1건.
- T-X2: Barker et al. (2021)의 결과 재현. 장비 높이 1 m와 5 m에서 clone 간 편차를 비교해, 5 m에서 편차가 줄어드는지 본다. 줄지 않으면 원인을 찾고 결과를 그대로 적는다.
- T-X3 (F14, P2): IM-2 목표점과 실제 착륙점의 일조 계산. 한계: 5 m 또는 20 m DEM은 착륙선이 들어간 작은 크레이터를 해상하지 못할 수 있다. 따라서 이 사례는 **검증이 아니라 위치 오차의 영향을 설명하는 사례**로만 쓴다. 목표점 좌표가 공개 자료에 없으면 실제 착륙점 주변 반경 250 m·400 m 원으로 대신한다.

---

## 11. 일정

| 시간 | 작업 | 게이트 |
|---|---|---|
| T+0~3h | 저장소 골격, `fetch_data.sh`, `[확인 필요]` 전부 해소, 지점 좌표·출처 확정, API 스키마 고정 | 미해결 `[확인 필요]` 0건 |
| T+3~9h | `ephemeris.py` + T-E. `terrain.py`. 프론트 골격(mock 타입) | **G1:** T-E1 통과 |
| T+9~16h | `horizon.py` 3대역 + 캐시 + T-H. `visibility.py` | **G2:** Site01·Site20 실제 H(A), near 1회 소요 시간 실측 |
| T+16~21h | `windows.py`, 화면 1·2 연동, 일정 지연 | **G3:** 데모 시나리오 1~4단계 동작 |
| T+21~28h | `robustness.py`, 화면 3, DSN | **G4:** K ≥ 10에서 `fragile_share` 산출 |
| T+28~32h | K 확대 실행(백그라운드), T-X1·X2, 원반 3상태, 높이 비교 | — |
| T+32~35h | 통합 테스트, 성능 실측, README | 기능 동결 |
| T+35~38h | 배포, 영상, 제출 페이지 | 제출 완료 |
| T+38~39h | 외부 접근 확인, 예비 | — |

게이트 실패 시
- G1 실패: 전원이 천체 위치에 붙는다. 시각화 중단.
- G2 실패: mid·far를 빼고 near만으로 진행하며 무효 방위를 그대로 표시한다. 지형 결과를 만들어 넣지 않는다.
- G3 실패: F8~F15를 취소하고 P0 안정화. 이 경우 차별점이 사라지므로 목표를 지역 통과로 낮춘다.
- G4 실패: 견고성을 "실험 기능"으로 분리하고 위치 민감도(F8, clone 없음)만 남긴다.
- T+35h 이후 새 기능 금지. 마감 1시간 전에 제출을 끝낸다.

---

## 12. 발표

- 첫 문장: "A lander never lands on the pin, and the map is never the Moon."
- 30초 구성: ① IM-2 사례 한 줄 ② 지점·날짜 선택과 파노라마 ③ 일정 지연 슬라이더 ④ Robustness 켜기 ⑤ `fragile_share` 숫자와 견고한 창.
- 심사 기준별 근거: 영향력(일정 변경 시 재평가라는 과제 배경), 창의성(위치·지형 불확실성), 타당성(SPICE·Horizons 대조, Barker 2021 재현, 테스트 수), 관련성(§1 표), 발표력(위 구성).
- 금지 표현: "NASA 승인", "성공 확률", "안전 보장", "최초". 권장: "NASA가 공개한 지형 실현 자료로 판단의 견고성을 비교했다."

---

## 13. 미해결 항목

| # | 항목 | 담당 | 기한 |
|---|---|---|---|
| O1 | 제품명. "Safe"가 "안전을 보장하지 않는다"는 원칙과 충돌한다 | 리더 | T+0 |
| O2 | 11-13 공개되는 과제 Resources 목록 확인, 사용할 NASA 자료 지정 | 리더 | 11-13 |
| O3 | 2026 제출 가이드(영상 길이·형식)와 사전 작업 규정 원문 | QA | 11-13 |
| O4 | `_err.tif` 정의, 기준 반지름, Site20 범위 | 지형 | T+1h |
| O5 | 지점 좌표와 출처, DSN 좌표·마스크 | 과학 계산 | T+2h |
| O6 | Horizons 달 표면 관측자 설정 | 과학 계산 | T+3h |
| O7 | 배포 플랫폼(8코어 이상, 메모리 16 GB 이상 권장. clone 100개 = 사이트당 약 4 GB) | 백엔드 | T+0 |
| O8 | 팀원별 역할 배정 | 리더 | 11-13 |

사전 작업에 대해: 과거 공식 FAQ(2024)는 행사 시작 전에 솔루션 작업을 시작할 수 없다고 적었고, 2026년 참가 팀들의 저장소도 같은 규정을 인용한다. 2026 FAQ 원문은 이 문서 작성 시점에 직접 읽지 못했다. 원문 확인 전까지는 문서 조사와 계획만 한다.

---

## 14. 완료 정의

- [ ] §1 표의 P0 항목이 실제 NASA 데이터로 동작한다.
- [ ] T-E, T-H, T-V, T-W가 통과하고 통과·실패 수가 README에 있다.
- [ ] 데모 시나리오가 공개 URL에서 끝까지 실행된다.
- [ ] `fragile_share`가 실제 계산값이고, 사용한 P·K·임계값이 함께 표시된다.
- [ ] 모든 숫자에 단위와 기간이 있고, 지형 미반영·예시 데이터가 구분된다.
- [ ] README에 데이터 출처, 커널·DEM 버전, 가정, 한계, 선행 연구(Barker 2021), AI 도구 사용 내역이 있다.
- [ ] 제출 페이지에 과제 Resources의 NASA 자료 사용 사실이 적혀 있다.

---

## 15. 참고 자료

- 공식 과제: https://www.spaceappschallenge.org/2026/challenges/lunar-landing-site-navigator/
- PGDA Product 78 (사이트 DEM, clone): https://pgda.gsfc.nasa.gov/products/78
- PGDA Product 90 (남극 광역 DEM): https://pgda.gsfc.nasa.gov/products/90
- PGDA Product 81 (87°S 이남 5 m 모자이크): https://pgda.gsfc.nasa.gov/products/81
- Barker, M. K. et al. (2021), *Planetary and Space Science* 203, 105119, https://doi.org/10.1016/j.pss.2020.105119
- Barker, M. K. et al. (2023), *Planetary Science Journal* 4, 183, https://doi.org/10.3847/PSJ/acf3e1
- Mazarico, E. et al. (2011), *Icarus* 211, 1066–1081
- NAIF generic kernels: https://naif.jpl.nasa.gov/pub/naif/generic_kernels/
- JPL Horizons API: https://ssd-api.jpl.nasa.gov/doc/horizons.html
- NASA SVS 4930: https://svs.gsfc.nasa.gov/4930/
- IM-2 임무 종료 발표: https://www.intuitivemachines.com/im-2 , https://spacenews.com/im-2-lunar-lander-mission-ends/
