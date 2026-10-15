# 여행 일정 앱 API 명세서

## 1. 문서 목적

여행 일정 앱의 프론트엔드와 백엔드가 주고받는
요청 및 응답 형식을 정의한다.

현재 문서는 초안이며, 팀 협의를 통해 수정한다.

## 2. 이번에 구현할 기능

1. 지도 공유 링크를 입력한다.
2. 공유 링크에서 장소 목록을 추출한다.
3. 사용자가 방문할 장소를 선택한다.
4. 선택한 장소를 여행의 방문 후보로 저장한다.
5. 장소별 방문 날짜와 순서를 지정한다.
6. 일정을 저장하고 다시 조회한다.

## 3. 공통 규칙

- API 기본 경로: /api/v1
- 요청 및 응답 형식: JSON
- 날짜 형식: YYYY-MM-DD
- 인증 방식: 미정

## 4. API 목록

| 기능 | Method | URL |
|---|---|---|
| 여행 생성 | POST | /api/v1/trips |
| 공유 링크 분석 요청 | POST | /api/v1/place-imports |
| 분석 결과 조회 | GET | /api/v1/place-imports/{importId} |
| 방문 후보 저장 | POST | /api/v1/trips/{tripId}/candidates |
| 방문 후보 조회 | GET | /api/v1/trips/{tripId}/candidates |
| 일정 저장 | PUT | /api/v1/trips/{tripId}/itinerary |
| 일정 조회 | GET | /api/v1/trips/{tripId}/itinerary |

## 5. API 상세 명세

### 5.1 공유 링크 분석 요청

사용자가 입력하거나 공유받은 지도 URL의 장소 정보를
분석하는 작업을 생성한다.

- Method: POST
- URL: /api/v1/place-imports
- 인증 필요 여부: 미정

#### 요청 Body

```json
{
  "sourceUrl": "https://naver.me/예시링크"
}
```

| 필드 | 타입 | 필수 여부 | 설명 |
|---|---|---|---|
| sourceUrl | string | 필수 | 분석할 지도 공유 링크 |

#### 성공 응답

HTTP 상태: 202 Accepted

```json
{
  "importId": "imp_001",
  "status": "PENDING"
}
```

| 필드 | 타입 | 설명 |
|---|---|---|
| importId | string | 분석 작업을 구분하는 ID |
| status | string | 현재 분석 상태 |

#### 분석 상태

| 값 | 의미 |
|---|---|
| PENDING | 분석 대기 |
| PROCESSING | 분석 중 |
| COMPLETED | 분석 완료 |
| FAILED | 분석 실패 |

#### 오류 응답

지원하지 않는 링크를 입력하면 아래와 같이 응답한다.

HTTP 상태: 400 Bad Request

```json
{
  "error": {
    "code": "UNSUPPORTED_SHARE_URL",
    "message": "지원하지 않는 지도 공유 링크입니다."
  }
}
```

#### 처리 규칙

- sourceUrl은 필수 입력값이다.
- 지원할 지도 서비스와 링크 유형은 검증 후 확정한다.
- 장소 분석 결과는 분석 결과 조회 API에서 확인한다.


### 5.2 공유 링크 분석 결과 조회

공유 링크 분석 작업의 진행 상태를 확인한다.
분석이 완료되면 추출된 장소 목록을 반환한다.

- Method: GET
- URL: /api/v1/place-imports/{importId}
- 인증 필요 여부: 미정

#### Path Parameter

| 필드 | 타입 | 필수 여부 | 설명 |
|---|---|---|---|
| importId | string | 필수 | 공유 링크 분석 요청 시 받은 작업 ID |

#### 요청 예시

GET /api/v1/place-imports/imp_001

- 요청 Body는 없다.

#### 성공 응답 — 분석 진행 중

HTTP 상태: 200 OK

```json
{
  "importId": "imp_001",
  "status": "PROCESSING",
  "places": [],
  "error": null
}
```

#### 성공 응답 — 분석 완료

HTTP 상태: 200 OK

아래 장소와 좌표는 응답 구조를 설명하기 위한 가상 데이터이다.

```json
{
  "importId": "imp_001",
  "status": "COMPLETED",
  "places": [
    {
      "importPlaceId": "ip_001",
      "provider": "NAVER",
      "providerPlaceId": "example_001",
      "name": "예시 카페",
      "address": "서울특별시 예시 주소 1",
      "latitude": 37.5,
      "longitude": 127.0,
      "validationStatus": "VALID",
      "duplicateOf": null
    },
    {
      "importPlaceId": "ip_002",
      "provider": "NAVER",
      "providerPlaceId": "example_002",
      "name": "예시 전시관",
      "address": "서울특별시 예시 주소 2",
      "latitude": null,
      "longitude": null,
      "validationStatus": "NEEDS_REVIEW",
      "duplicateOf": null
    }
  ],
  "error": null
}
```

#### 응답 필드

| 필드 | 타입 | 설명 |
|---|---|---|
| importId | string | 분석 작업 ID |
| status | string | 분석 작업의 현재 상태 |
| places | array | 추출된 장소 목록 |
| error | object 또는 null | 분석 실패 시 오류 정보 |

#### places 항목의 필드

| 필드 | 타입 | 설명 |
|---|---|---|
| importPlaceId | string | 추출 결과 안에서 장소를 구분하는 ID |
| provider | string | 장소 정보 출처. 현재 초안은 NAVER |
| providerPlaceId | string 또는 null | 지도 서비스에서 사용하는 장소 ID |
| name | string | 장소명 |
| address | string 또는 null | 장소 주소 |
| latitude | number 또는 null | 위도 |
| longitude | number 또는 null | 경도 |
| validationStatus | string | 장소 정보 검증 상태 |
| duplicateOf | string 또는 null | 중복된 원본 장소의 importPlaceId |

#### 분석 상태

| 값 | 의미 |
|---|---|
| PENDING | 분석 대기 |
| PROCESSING | 분석 중 |
| COMPLETED | 분석 완료 |
| FAILED | 분석 실패 |

#### 장소 정보 검증 상태

| 값 | 의미 |
|---|---|
| VALID | 장소명, 주소, 좌표가 있어 선택 가능한 상태 |
| NEEDS_REVIEW | 필수 정보가 누락되어 확인이 필요한 상태 |
| DUPLICATE | 같은 분석 결과 안에서 중복된 장소 |

#### 분석 작업 실패 응답

결과 조회는 성공했지만 링크 분석 작업이 실패한 경우이다.

HTTP 상태: 200 OK

```json
{
  "importId": "imp_001",
  "status": "FAILED",
  "places": [],
  "error": {
    "code": "SHARE_LINK_UNAVAILABLE",
    "message": "공유 링크에 접근할 수 없습니다."
  }
}
```

#### 존재하지 않는 작업 조회

HTTP 상태: 404 Not Found

```json
{
  "error": {
    "code": "IMPORT_NOT_FOUND",
    "message": "분석 작업을 찾을 수 없습니다."
  }
}
```

#### 처리 규칙

- PENDING 또는 PROCESSING 상태에서는 places를 빈 배열로 반환한다.
- COMPLETED 상태에서는 추출된 장소 목록을 반환한다.
- 분석이 완료되었지만 장소가 없으면 places를 빈 배열로 반환한다.
- FAILED 상태에서는 places를 빈 배열로 반환하고 error에 실패 이유를 제공한다.
- 분석 실패가 아니면 error는 null로 반환한다.
- 주소나 좌표를 확인하지 못하면 해당 필드는 null로 반환한다.
- 장소 정보가 일부 누락되어도 다른 장소의 추출 결과는 제공한다.
- 동일한 provider와 providerPlaceId가 있으면 중복으로 판별한다.
- providerPlaceId가 없는 장소의 중복 판별 기준은 추후 확정한다.
- 장소명이 같다는 이유만으로 중복 처리하지 않는다.
- DUPLICATE 상태에서는 duplicateOf에 원본 장소의 importPlaceId를 반환한다.
- 현재 초안에서는 VALID 상태인 장소만 선택할 수 있다.
- 앱은 PENDING 또는 PROCESSING 상태에서 로딩 화면을 표시한다.
- 앱은 COMPLETED 상태에서 장소 선택 목록 또는 빈 결과 안내를 표시한다.
- 앱은 FAILED 상태에서 실패 이유와 다시 시도할 수 있는 화면을 표시한다.
## 6. 미확정 사항

- 지원할 지도 공유 링크의 종류
- 네이버 저장 폴더 공유 링크에서 장소를 추출할 수 있는지
- 로그인 및 인증 방식
- 중복 장소 처리 기준
