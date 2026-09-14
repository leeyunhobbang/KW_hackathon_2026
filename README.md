# 🚌 노원구 공공순환버스 실시간 도착정보 시스템

> **지역사회 봉사를 위한 소프트웨어 개발 해커톤 프로젝트**

노원구에서 운영되는 공공순환버스의 실시간 위치와 운행 데이터를 수집하고,
시간대·요일·도로 상황 등에 따른 운행 패턴을 분석하여 **버스의 예상 도착시간(ETA)을 제공하는 공공교통 서비스**를 개발한다.

본 프로젝트는 단순한 버스 도착정보 앱에 그치지 않고, 향후 다양한 지역의 공공교통 서비스에서 활용할 수 있도록 **API 중심의 확장 가능한 실시간 교통 데이터 플랫폼**을 구축하는 것을 목표로 한다.

---

## 1. 프로젝트 배경

공공순환버스는 지역 주민의 이동 편의를 위해 운영되지만, 사용자가 실제 버스의 위치나 예상 도착시간을 직관적으로 확인하기 어려운 경우가 있다.

특히 다음과 같은 문제가 존재한다.

* 버스가 현재 어디에 있는지 알기 어려움
* 다음 버스가 언제 도착하는지 알기 어려움
* 시간대와 교통상황에 따라 실제 도착시간이 달라짐
* 단순 시간표만으로는 실시간 운행 상황을 반영하기 어려움
* 지역별로 공공교통 데이터를 활용할 수 있는 서비스가 제한적임

따라서 본 프로젝트에서는 **실제 운행 데이터를 수집하고 이를 분석·예측하여 사용자에게 실시간 교통정보를 제공**한다.

---

# 2. 프로젝트 목표

### 핵심 목표

> **노원구 공공순환버스의 실시간 위치와 운행 데이터를 기반으로 정확한 도착 예정시간을 제공한다.**

### 세부 목표

1. 실제 버스 운행 데이터를 수집한다.
2. 정류장 간 실제 운행시간을 분석한다.
3. 시간대·요일·교통상황에 따른 지연 패턴을 분석한다.
4. 머신러닝 기반 ETA(Estimated Time of Arrival) 예측 모델을 구축한다.
5. 예측값과 실제 도착시간을 비교하여 모델의 정확도를 평가한다.
6. 차량에 탑재 가능한 **Vehicle Terminal 하드웨어 모듈**을 구현한다.
7. 실시간 데이터를 REST API를 통해 제공한다.
8. 특정 지역에 종속되지 않는 공공교통 API 구조를 설계한다.

---

# 3. 핵심 서비스

사용자는 앱을 통해 다음과 같은 정보를 확인할 수 있다.

```text
┌─────────────────────────────┐
│       노원 공공순환버스      │
├─────────────────────────────┤
│                             │
│  📍 월계역                  │
│                             │
│  🚌 순환버스 A              │
│     약 4분 후 도착           │
│                             │
│  🚌 순환버스 A              │
│     약 17분 후 도착          │
│                             │
└─────────────────────────────┘
```

지도에서는 실시간 버스 위치를 표시한다.

```text
        🚌
         ↓
●────────●────────●────────●
A        B        C        D
```

---

# 4. 시스템 전체 구조

```text
                        ┌─────────────────────┐
                        │   Vehicle Terminal  │
                        │                     │
                        │ GPS / MCU / Network │
                        └──────────┬──────────┘
                                   │
                                   │ Telemetry
                                   ▼
                        ┌─────────────────────┐
                        │    Backend Server   │
                        │                     │
                        │ Vehicle Service     │
                        │ Route Service       │
                        │ Stop Service        │
                        │ ETA Service         │
                        └──────────┬──────────┘
                                   │
                    ┌──────────────┴──────────────┐
                    ▼                             ▼
             ┌────────────┐               ┌────────────┐
             │   Redis    │               │ PostgreSQL │
             │            │               │            │
             │ Live Data  │               │ Historical │
             └─────┬──────┘               └────────────┘
                   │
                   ▼
             ┌─────────────┐
             │  ETA Model  │
             │             │
             │ ML Prediction│
             └──────┬──────┘
                    │
                    ▼
             ┌─────────────┐
             │ Transit API │
             └──────┬──────┘
                    │
             ┌──────┴──────┐
             ▼             ▼
        ┌─────────┐   ┌──────────────┐
        │ Mobile  │   │ External     │
        │ App     │   │ Applications │
        └─────────┘   └──────────────┘
```

---

# 5. Vehicle Terminal

본 프로젝트의 전시 및 실제 서비스 확장을 위해 **차량용 전자식 모듈(Vehicle Terminal)**을 제작한다.

Vehicle Terminal은 실제 버스에 탑재되어 GPS 및 운행정보를 서버로 전달하는 장치를 가정한다.

### 역할

* 차량 위치 수집
* 차량 속도 수집
* 차량 ID 식별
* 노선 ID 식별
* 서버 연결 상태 확인
* 실시간 운행 데이터 전송

### 예상 구성

```text
┌─────────────────────────┐
│     VEHICLE TERMINAL    │
├─────────────────────────┤
│                         │
│ GPS Module              │
│      ↓                  │
│ MCU / Embedded Computer  │
│      ↓                  │
│ Network Module          │
│      ↓                  │
│ Transit Backend         │
│                         │
└─────────────────────────┘
```

### 전시용 구성

전시회에서는 실제 버스에 장착하지 않고 **GPS Simulator 또는 테스트 데이터를 이용하여 차량의 이동을 재현**할 수 있다.

```text
GPS Simulator
      │
      ▼
Vehicle Terminal
      │
      ▼
Backend
      │
      ▼
ETA Prediction
      │
      ▼
Mobile App
```

이를 통해 실제 차량 단말이 데이터를 서버로 전송하는 과정을 전시장에서 시각적으로 재현한다.

---

# 6. 데이터 수집

초기 단계에서는 실제 버스 운행을 직접 관찰하여 Ground Truth 데이터를 구축한다.

### 수집 데이터

| 데이터              | 설명             |
| ---------------- | -------------- |
| `date`           | 운행 날짜          |
| `day_of_week`    | 요일             |
| `route_id`       | 노선 ID          |
| `vehicle_id`     | 차량 ID          |
| `stop_id`        | 정류장 ID         |
| `scheduled_time` | 기존 예정시간        |
| `actual_time`    | 실제 도착시간        |
| `delay`          | 예정시간과 실제시간의 차이 |
| `weather`        | 날씨             |
| `temperature`    | 기온             |
| `traffic`        | 교통상황           |
| `latitude`       | 차량 위도          |
| `longitude`      | 차량 경도          |
| `speed`          | 차량 속도          |
| `timestamp`      | 데이터 측정시간       |

---

# 7. 정류장 간 운행시간 분석

전체 노선의 운행시간뿐만 아니라 **정류장 간 이동시간**을 분석한다.

예:

```text
A → B : 3분 42초
B → C : 5분 13초
C → D : 4분 01초
D → E : 6분 20초
```

이를 기반으로 각 구간의 평균 및 분산을 계산한다.

```text
현재 위치
   │
   ▼
C까지      3분
C → D      5분
D → E      4분
   │
   ▼
예상 도착시간 ≈ 12분
```

이 데이터는 ETA Prediction Model의 핵심 학습 데이터로 사용한다.

---

# 8. ETA Prediction

## Baseline

먼저 단순 평균 기반 ETA를 구축한다.

```text
ETA = 각 구간의 평균 소요시간 합
```

예:

```text
A → B 평균 = 4.2분
B → C 평균 = 5.1분
C → D 평균 = 3.8분

ETA = 13.1분
```

---

## Machine Learning Model

이후 운행 데이터를 기반으로 머신러닝 모델을 구축한다.

### Feature

```text
시간
요일
정류장
현재 위치
현재 속도
최근 평균 속도
과거 구간 평균 소요시간
교통상황
날씨
공휴일 여부
```

### Model

초기에는 데이터 규모에 따라 다음과 같은 전통적인 머신러닝 모델을 우선 검토한다.

* Linear Regression
* Random Forest
* Gradient Boosting
* XGBoost

데이터가 충분히 축적될 경우 시계열 모델 및 딥러닝 모델로 확장할 수 있다.

---

# 9. 실제 도착시간과 예측시간 비교

모델의 성능을 평가하기 위해 예측값과 실제 도착시간을 지속적으로 비교한다.

```text
예측 ETA : 7분
실제 ETA : 8분

오차 : +1분
```

### 주요 평가 지표

#### MAE

Mean Absolute Error

```text
MAE = 평균 |예측값 - 실제값|
```

예:

```text
예측 5분 / 실제 6분 → 1분
예측 8분 / 실제 7분 → 1분
예측 10분 / 실제 13분 → 3분

MAE = 1.67분
```

이를 통해 모델이 평균적으로 몇 분 정도의 오차를 가지는지 평가한다.

---

# 10. 교통상황에 따른 지연 분석

ETA 오차가 발생하는 원인을 분석한다.

```text
                    ETA Error
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
        시간대          요일          교통
          │             │             │
       출근시간        평일          혼잡
       퇴근시간        주말          정체
```

예를 들어:

| 조건             | 평균 지연 |
| -------------- | ----: |
| 평일 08:00~09:00 | +2.4분 |
| 평일 12:00~13:00 | +0.6분 |
| 평일 18:00~19:00 | +3.1분 |
| 주말             | +0.8분 |

이를 통해 단순한 ETA뿐만 아니라 **어떤 상황에서 예측 오차가 증가하는지** 분석한다.

---

# 11. Prediction Model 비교

머신러닝 모델을 단순히 사용하는 것이 아니라 Baseline과 성능을 비교한다.

| Model        |  MAE |
| ------------ | ---: |
| 단순 평균        | 2.8분 |
| 시간대 기반       | 2.1분 |
| 시간 + 요일      | 1.9분 |
| 시간 + 요일 + 교통 | 1.5분 |
| ML Model     | 1.3분 |

> 실제 수치는 데이터 수집 이후 측정하여 기록한다.

이를 통해 머신러닝 적용이 실제 ETA 정확도 향상에 기여했는지 검증한다.

---

# 12. Backend

Backend는 **API 중심 구조**로 설계한다.

### 주요 역할

* 차량 데이터 수집
* 실시간 차량 상태 관리
* 노선 및 정류장 정보 제공
* ETA 계산
* ML Model 호출
* 과거 운행 데이터 저장
* 외부 서비스에 교통 데이터 제공

---

# 13. API

예상 API 구조:

```http
GET /api/v1/routes
GET /api/v1/routes/{route_id}

GET /api/v1/stops
GET /api/v1/stops/{stop_id}

GET /api/v1/stops/{stop_id}/arrivals

GET /api/v1/vehicles
GET /api/v1/vehicles/{vehicle_id}

POST /api/v1/telemetry
```

### 실시간 도착정보

```http
GET /api/v1/stops/NW001/arrivals
```

Response:

```json
{
  "stop_id": "NW001",
  "arrivals": [
    {
      "route_id": "NW01",
      "vehicle_id": "BUS001",
      "eta_seconds": 420
    },
    {
      "route_id": "NW01",
      "vehicle_id": "BUS002",
      "eta_seconds": 1020
    }
  ]
}
```

---

# 14. Telemetry API

Vehicle Terminal은 실시간 운행정보를 Backend에 전송한다.

```http
POST /api/v1/telemetry
```

```json
{
  "vehicle_id": "BUS001",
  "route_id": "NW01",
  "latitude": 37.6543,
  "longitude": 127.0562,
  "speed": 21.3,
  "timestamp": "2026-09-14T13:30:00"
}
```

Backend는 이 데이터를 기반으로 현재 차량 상태를 갱신하고 ETA Prediction에 활용한다.

---

# 15. Redis

실시간 차량 데이터는 Redis를 활용하여 관리한다.

```text
vehicle:BUS001

latitude
longitude
speed
timestamp
route_id
```

Redis는 **현재 차량 상태와 같이 빠르게 변경되고 자주 조회되는 데이터**를 저장하는 용도로 사용한다.

반면 PostgreSQL 등의 데이터베이스에는 과거 운행 데이터를 저장한다.

```text
                 Backend
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
       Redis              PostgreSQL
          │                   │
     현재 차량 위치          운행 기록
     현재 속도              과거 ETA
     최신 상태              실제 도착시간
```

---

# 16. 앱

앱은 Backend API의 Client로 동작한다.

### 주요 기능

* 현재 위치 기반 가까운 정류장 검색
* 정류장별 버스 도착정보
* 지도 기반 버스 위치 표시
* ETA 표시
* 노선 및 정류장 정보
* 운행 상태 표시

앱은 ETA Model을 직접 실행하지 않는다.

```text
Mobile App
     │
     │ API Request
     ▼
Backend
     │
     ▼
ETA Model
     │
     ▼
ETA Response
     │
     ▼
Mobile App
```

이를 통해 향후 앱이 변경되더라도 Backend와 Prediction System을 그대로 사용할 수 있다.

---

# 17. API First Architecture

본 프로젝트에서 앱은 시스템의 최종 목적이 아니라 **API의 하나의 Consumer**로 정의한다.

```text
                    Transit API
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
       Mobile          Web        External App
         App
```

따라서 향후 다음과 같은 서비스에서 활용할 수 있다.

* 모바일 앱
* 웹 서비스
* 공공기관 홈페이지
* 정류장 전광판
* 키오스크
* 다른 개발자의 교통 서비스

---

# 18. 확장성

현재는 노원구 공공순환버스를 대상으로 하지만 특정 지역에 종속되지 않는 데이터 모델을 사용한다.

### 주요 Domain

```text
Agency
Route
Stop
Trip
Vehicle
VehiclePosition
StopTime
ArrivalPrediction
```

예:

```text
Agency
 ├── Nowon Public Transit
 │      ├── Route A
 │      └── Route B
 │
 └── Future Transit Agency
        ├── Route C
        └── Route D
```

따라서 향후 다른 지역의 공공버스, 마을버스, 셔틀버스 등으로 확장할 수 있다.

---

# 19. 향후 확장

### Phase 1 — Hackathon MVP

```text
실제 운행 관측
      ↓
데이터셋 구축
      ↓
ETA 분석
      ↓
ML Prediction
      ↓
API
      ↓
앱
```

### Phase 2 — Vehicle Terminal

```text
실제 GPS
    ↓
Vehicle Terminal
    ↓
Telemetry API
    ↓
Backend
```

### Phase 3 — Real-time Transit Platform

```text
여러 지역의 차량
       ↓
Transit API
       ↓
┌──────┼──────┐
앱     웹    외부 서비스
```

### Phase 4 — Open Transit Data

향후 GTFS 및 GTFS-Realtime과 같은 대중교통 데이터 표준과의 호환을 고려한다.

이를 통해 특정 앱에서만 사용하는 데이터가 아니라 **다른 서비스에서도 활용 가능한 공공교통 데이터 API**로 발전시키는 것을 목표로 한다.

---

# 20. 전시 계획

본 프로젝트는 전시회에서 실제 데이터의 흐름을 시각적으로 보여주는 것을 중요하게 생각한다.

### 전시 구성

```text
┌───────────────┐
│ Vehicle       │
│ Terminal      │
│               │
│ GPS ●         │
│ SERVER ●      │
└───────┬───────┘
        │
        ▼
   Backend Server
        │
        ▼
   ETA Prediction
        │
        ▼
┌──────────────────┐
│      MAP         │
│                  │
│ ●────●────●      │
│       🚌         │
│                  │
│ 다음 정류장       │
│ 월계역            │
│ 약 4분 후         │
└──────────────────┘
```

관람객에게 다음 데이터 흐름을 직관적으로 보여준다.

> **차량 → GPS → 서버 → 데이터 분석 → ETA Prediction → API → 사용자 앱**

---

# 21. 프로젝트의 핵심 가치

### ① 지역사회 기여

지역 주민이 공공순환버스를 더욱 편리하게 이용할 수 있도록 실시간 교통정보를 제공한다.

### ② 데이터 기반 의사결정

단순한 시간표가 아닌 실제 운행 데이터를 수집하고 분석하여 교통 상황에 따른 지연 패턴을 파악한다.

### ③ AI 활용

머신러닝을 통해 기존의 단순 평균 ETA보다 정확한 도착시간을 예측하는 것을 목표로 한다.

### ④ IoT

Vehicle Terminal을 통해 실제 차량에서 데이터를 수집할 수 있는 구조를 구현한다.

### ⑤ API 확장성

앱에 종속되지 않는 REST API를 제공하여 다양한 서비스에서 공공교통 데이터를 활용할 수 있도록 한다.

### ⑥ 장기적인 확장성

노원구 공공순환버스에서 시작하여 다른 지역 및 다양한 공공교통 서비스로 확장할 수 있는 플랫폼을 지향한다.

---

# 22. 기술 스택

### Frontend

* Mobile / Web
* Map SDK

### Backend

* Python / FastAPI
* REST API

### Database

* PostgreSQL
* Redis

### Data / AI

* Python
* Pandas
* NumPy
* Scikit-learn
* XGBoost

### Hardware

* ESP32 또는 Raspberry Pi
* GPS Module
* Network Module
* OLED / LCD Display

### Infrastructure

* Docker
* Cloud Server

> 세부 기술 스택은 개발 과정에서 변경될 수 있다.

---

# 23. 최종 목표

본 프로젝트의 최종 목표는 단순히 **"버스가 몇 분 뒤 도착하는지 알려주는 앱"**을 만드는 것이 아니다.

> **공공교통의 운행 데이터를 수집하고, 분석하고, 예측하여 누구나 활용할 수 있도록 제공하는 실시간 공공교통 데이터 플랫폼을 구축하는 것**이다.

노원구 공공순환버스를 시작점으로 하여,

```text
실제 운행
    ↓
IoT / GPS
    ↓
Data Collection
    ↓
Data Analysis
    ↓
Machine Learning
    ↓
ETA Prediction
    ↓
Transit API
    ↓
Mobile / Web / Public Service
```

로 이어지는 하나의 공공교통 데이터 생태계를 구축한다.
