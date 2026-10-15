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

| 번호 | 기능 | Method | URL |
|---|---|---|---|
| 5.1 | 공유 링크 분석 요청 | POST | /api/v1/place-imports |
| 5.2 | 공유 링크 분석 결과 조회 | GET | /api/v1/place-imports/{importId} |
| 5.3 | 여행 생성 | POST | /api/v1/trips |
| 5.4 | 방문 후보 저장 | POST | /api/v1/trips/{tripId}/candidates |
| 5.5 | 방문 후보 조회 | GET | /api/v1/trips/{tripId}/candidates |
| 5.6 | 일정 전체 저장 | PUT | /api/v1/trips/{tripId}/itinerary |
| 5.7 | 일정 조회 | GET | /api/v1/trips/{tripId}/itinerary |
| 5.8 | 일정 항목 메모 수정 | PATCH | /api/v1/trips/{tripId}/itinerary-items/{itemId} |

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

### 5.3 여행 생성

여행 제목과 시작일, 종료일을 입력받아 새로운 여행을 생성한다.
생성한 여행의 ID는 방문 후보 및 일정 저장에 사용한다.

- Method: POST
- URL: /api/v1/trips
- 인증 필요 여부: 미정

#### 요청 Body

```json
{
  "title": "서울 여행",
  "startDate": "2026-10-20",
  "endDate": "2026-10-22"
}
```

#### 요청 필드

| 필드 | 타입 | 필수 여부 | 설명 |
|---|---|---|---|
| title | string | 필수 | 여행 제목 |
| startDate | string | 필수 | 여행 시작일. YYYY-MM-DD 형식 |
| endDate | string | 필수 | 여행 종료일. YYYY-MM-DD 형식 |

#### 성공 응답

HTTP 상태: 201 Created

```json
{
  "tripId": "trip_001",
  "title": "서울 여행",
  "startDate": "2026-10-20",
  "endDate": "2026-10-22"
}
```

#### 응답 필드

| 필드 | 타입 | 설명 |
|---|---|---|
| tripId | string | 생성된 여행을 구분하는 ID |
| title | string | 저장된 여행 제목 |
| startDate | string | 여행 시작일 |
| endDate | string | 여행 종료일 |

#### 오류 응답 — 여행 제목 누락

HTTP 상태: 400 Bad Request

```json
{
  "error": {
    "code": "INVALID_REQUEST",
    "message": "여행 제목을 입력해 주세요.",
    "field": "title"
  }
}
```

#### 오류 응답 — 잘못된 날짜 형식

HTTP 상태: 400 Bad Request

```json
{
  "error": {
    "code": "INVALID_DATE_FORMAT",
    "message": "날짜는 YYYY-MM-DD 형식의 유효한 날짜여야 합니다.",
    "field": "startDate"
  }
}
```

#### 오류 응답 — 잘못된 여행 기간

HTTP 상태: 400 Bad Request

```json
{
  "error": {
    "code": "INVALID_TRIP_DATE_RANGE",
    "message": "여행 종료일은 시작일보다 빠를 수 없습니다.",
    "field": "endDate"
  }
}
```

#### 처리 규칙

- title, startDate, endDate는 필수 입력값이다.
- 여행 제목은 앞뒤 공백을 제거한 후 저장한다.
- 여행 제목이 빈 문자열이거나 공백만 있으면 저장하지 않는다.
- 날짜는 YYYY-MM-DD 형식이어야 하며 실제 존재하는 날짜여야 한다.
- 종료일은 시작일과 같거나 이후여야 한다.
- 시작일과 종료일이 같은 당일 여행을 허용한다.
- tripId는 서버에서 생성하며 앱에서 지정하지 않는다.
- 여행 생성 시 방문 후보와 일정 항목은 비어 있는 상태이다.
- 이후 일정에 지정하는 방문 날짜는 여행 시작일과 종료일 범위 안에 있어야 한다.
- 로그인 및 여행 소유자 연결 방식은 인증 설계 후 확정한다.

### 5.4 방문 후보 저장

공유 링크에서 추출한 장소 중 사용자가 선택한 장소를
해당 여행의 방문 후보로 저장한다.

- Method: POST
- URL: /api/v1/trips/{tripId}/candidates
- 인증 필요 여부: 미정

#### Path Parameter

| 필드 | 타입 | 필수 여부 | 설명 |
|---|---|---|---|
| tripId | string | 필수 | 장소를 담을 여행 ID |

#### 요청 Body

```json
{
  "importId": "imp_001",
  "importPlaceIds": [
    "ip_001",
    "ip_003"
  ]
}
```

#### 요청 필드

| 필드 | 타입 | 필수 여부 | 설명 |
|---|---|---|---|
| importId | string | 필수 | 장소를 추출한 분석 작업 ID |
| importPlaceIds | array of string | 필수 | 사용자가 선택한 추출 장소 ID 목록 |

#### 성공 응답

HTTP 상태: 200 OK

```json
{
  "tripId": "trip_001",
  "candidates": [
    {
      "candidateId": "cand_001",
      "placeId": "place_001",
      "name": "예시 카페",
      "address": "서울특별시 예시 주소 1",
      "latitude": 37.5,
      "longitude": 127.0,
      "added": true
    },
    {
      "candidateId": "cand_002",
      "placeId": "place_003",
      "name": "예시 공원",
      "address": "서울특별시 예시 주소 3",
      "latitude": 37.51,
      "longitude": 127.01,
      "added": true
    }
  ]
}
```

#### 응답 필드

| 필드 | 타입 | 설명 |
|---|---|---|
| tripId | string | 방문 후보가 저장된 여행 ID |
| candidates | array | 이번 요청으로 추가되었거나 기존에 저장되어 있던 후보 목록 |
| candidates[].candidateId | string | 여행 안에서 방문 후보를 구분하는 ID |
| candidates[].placeId | string | 서버에서 장소 자체를 구분하는 ID |
| candidates[].name | string | 장소명 |
| candidates[].address | string | 주소 |
| candidates[].latitude | number | 위도 |
| candidates[].longitude | number | 경도 |
| candidates[].added | boolean | 이번 요청으로 새로 저장했으면 true, 기존 후보면 false |

#### 오류 응답 — 선택한 장소 없음

HTTP 상태: 400 Bad Request

```json
{
  "error": {
    "code": "EMPTY_PLACE_SELECTION",
    "message": "방문할 장소를 한 개 이상 선택해 주세요.",
    "field": "importPlaceIds"
  }
}
```

#### 오류 응답 — 분석 미완료

HTTP 상태: 409 Conflict

```json
{
  "error": {
    "code": "IMPORT_NOT_COMPLETED",
    "message": "장소 분석이 완료된 후 다시 시도해 주세요.",
    "field": "importId"
  }
}
```

#### 오류 응답 — 선택할 수 없는 장소

HTTP 상태: 422 Unprocessable Content

```json
{
  "error": {
    "code": "PLACE_NOT_SELECTABLE",
    "message": "정보 확인이 필요한 장소가 포함되어 있습니다.",
    "field": "importPlaceIds"
  }
}
```

#### 처리 규칙

- 여행과 분석 작업이 존재해야 한다.
- 분석 작업의 상태가 COMPLETED여야 한다.
- importPlaceIds에는 최소 한 개의 ID가 있어야 한다.
- 선택한 모든 장소는 요청한 importId의 분석 결과에 속해야 한다.
- validationStatus가 VALID인 장소만 저장할 수 있다.
- 요청 목록에 동일한 importPlaceId가 여러 번 있으면 한 번만 처리한다.
- 동일한 장소가 이미 해당 여행의 후보로 저장되어 있으면 기존 candidateId를 반환한다.
- 기존 후보를 반환한 경우 added는 false이다.
- 새 후보를 저장한 경우 added는 true이다.
- 장소 중복 여부는 서버가 식별한 동일한 placeId를 기준으로 한다.
- 서로 다른 분석 결과의 장소를 같은 placeId로 연결하는 기준은 중복 판별 설계에서 확정한다.
- 선택한 항목 중 하나라도 유효하지 않으면 이번 요청의 전체 저장을 취소한다.
- 후보를 저장하는 단계에서는 방문 날짜와 순서를 지정하지 않는다.

### 5.5 방문 후보 조회

해당 여행에 저장된 방문 후보 목록을 조회한다.

- Method: GET
- URL: /api/v1/trips/{tripId}/candidates
- 인증 필요 여부: 미정

#### Path Parameter

| 필드 | 타입 | 필수 여부 | 설명 |
|---|---|---|---|
| tripId | string | 필수 | 조회할 여행 ID |

#### 요청 예시

GET /api/v1/trips/trip_001/candidates

- 요청 Body는 없다.

#### 성공 응답

HTTP 상태: 200 OK

```json
{
  "tripId": "trip_001",
  "candidates": [
    {
      "candidateId": "cand_001",
      "placeId": "place_001",
      "name": "예시 카페",
      "address": "서울특별시 예시 주소 1",
      "latitude": 37.5,
      "longitude": 127.0,
      "scheduled": false
    },
    {
      "candidateId": "cand_002",
      "placeId": "place_003",
      "name": "예시 공원",
      "address": "서울특별시 예시 주소 3",
      "latitude": 37.51,
      "longitude": 127.01,
      "scheduled": true
    }
  ]
}
```

#### 응답 필드

| 필드 | 타입 | 설명 |
|---|---|---|
| tripId | string | 여행 ID |
| candidates | array | 저장된 방문 후보 목록 |
| candidates[].candidateId | string | 방문 후보 ID |
| candidates[].placeId | string | 장소 ID |
| candidates[].name | string | 장소명 |
| candidates[].address | string | 주소 |
| candidates[].latitude | number | 위도 |
| candidates[].longitude | number | 경도 |
| candidates[].scheduled | boolean | 현재 일정에 포함되어 있으면 true |

#### 처리 규칙

- 후보 목록은 저장된 순서대로 반환한다.
- 후보가 없으면 candidates를 빈 배열로 반환한다.
- 후보가 없는 경우도 정상 조회이므로 200 OK를 반환한다.
- scheduled는 현재 저장된 일정 포함 여부를 기준으로 계산한다.
- 여행이 존재하지 않으면 TRIP_NOT_FOUND 오류를 반환한다.

### 5.6 일정 전체 저장

방문 후보에 날짜와 방문 순서를 지정하여
해당 여행의 전체 일정을 저장한다.

- Method: PUT
- URL: /api/v1/trips/{tripId}/itinerary
- 인증 필요 여부: 미정

#### Path Parameter

| 필드 | 타입 | 필수 여부 | 설명 |
|---|---|---|---|
| tripId | string | 필수 | 일정을 저장할 여행 ID |

#### 요청 Body

```json
{
  "items": [
    {
      "candidateId": "cand_001",
      "visitDate": "2026-10-20",
      "visitOrder": 1,
      "memo": "오전에 방문"
    },
    {
      "candidateId": "cand_002",
      "visitDate": "2026-10-20",
      "visitOrder": 2,
      "memo": ""
    }
  ]
}
```

#### 요청 필드

| 필드 | 타입 | 필수 여부 | 설명 |
|---|---|---|---|
| items | array | 필수 | 저장할 전체 일정 목록 |
| items[].candidateId | string | 필수 | 여행에 저장된 방문 후보 ID |
| items[].visitDate | string | 필수 | 방문 날짜. YYYY-MM-DD 형식 |
| items[].visitOrder | integer | 필수 | 같은 날짜 안에서의 방문 순서 |
| items[].memo | string | 선택 | 사용자 메모. 생략 시 빈 문자열로 저장 |

#### 성공 응답

HTTP 상태: 200 OK

```json
{
  "tripId": "trip_001",
  "items": [
    {
      "itemId": "item_001",
      "candidateId": "cand_001",
      "visitDate": "2026-10-20",
      "visitOrder": 1,
      "memo": "오전에 방문"
    },
    {
      "itemId": "item_002",
      "candidateId": "cand_002",
      "visitDate": "2026-10-20",
      "visitOrder": 2,
      "memo": ""
    }
  ]
}
```

#### 응답 필드

| 필드 | 타입 | 설명 |
|---|---|---|
| tripId | string | 여행 ID |
| items | array | 저장된 전체 일정 |
| items[].itemId | string | 일정 항목 ID |
| items[].candidateId | string | 방문 후보 ID |
| items[].visitDate | string | 방문 날짜 |
| items[].visitOrder | integer | 해당 날짜의 방문 순서 |
| items[].memo | string | 사용자 메모 |

#### 오류 응답 — 여행 기간을 벗어난 날짜

HTTP 상태: 422 Unprocessable Content

```json
{
  "error": {
    "code": "VISIT_DATE_OUT_OF_RANGE",
    "message": "방문 날짜는 여행 기간 안에 있어야 합니다.",
    "field": "items[0].visitDate"
  }
}
```

#### 오류 응답 — 중복된 방문 순서

HTTP 상태: 409 Conflict

```json
{
  "error": {
    "code": "DUPLICATE_VISIT_ORDER",
    "message": "같은 날짜에 동일한 방문 순서를 지정할 수 없습니다.",
    "field": "items"
  }
}
```

#### 오류 응답 — 잘못된 방문 순서

HTTP 상태: 422 Unprocessable Content

```json
{
  "error": {
    "code": "INVALID_VISIT_ORDER",
    "message": "방문 순서는 날짜별로 1부터 연속되어야 합니다.",
    "field": "items"
  }
}
```

#### 처리 규칙

- 요청한 items 목록으로 해당 여행의 전체 일정을 교체한다.
- items 필드를 생략하거나 null로 보내면 오류를 반환한다.
- items를 빈 배열로 보내면 전체 일정을 비운다.
- 기존 일정에서 빠진 항목은 삭제한다.
- 일정 항목이 삭제되어도 방문 후보는 유지한다.
- candidateId는 해당 여행에 속한 방문 후보여야 한다.
- 현재 초안에서는 같은 candidateId를 전체 일정에 한 번만 사용할 수 있다.
- visitDate는 여행 시작일과 종료일을 포함한 범위 안에 있어야 한다.
- visitOrder는 날짜별로 1부터 연속되어야 한다.
- 같은 날짜에 동일한 visitOrder를 사용할 수 없다.
- memo는 문자열이며 null은 허용하지 않는다.
- 기존 일정에 있던 candidateId는 itemId를 유지한다.
- 새로 일정에 포함된 candidateId에는 새로운 itemId를 생성한다.
- 항목 중 하나라도 검증에 실패하면 전체 변경을 저장하지 않는다.
- 응답은 visitDate 오름차순, visitOrder 오름차순으로 반환한다.
- AI 팁 생성 여부와 관계없이 일정 저장은 완료할 수 있어야 한다.

### 5.7 일정 조회

해당 여행의 일정과 장소 정보를 조회한다.
앱은 응답 데이터를 이용하여 일정 카드와 지도 마커를 표시한다.

- Method: GET
- URL: /api/v1/trips/{tripId}/itinerary
- 인증 필요 여부: 미정

#### Path Parameter

| 필드 | 타입 | 필수 여부 | 설명 |
|---|---|---|---|
| tripId | string | 필수 | 조회할 여행 ID |

#### 요청 예시

GET /api/v1/trips/trip_001/itinerary

- 요청 Body는 없다.

#### 성공 응답

HTTP 상태: 200 OK

아래 장소와 좌표는 응답 구조를 설명하기 위한 가상 데이터이다.

```json
{
  "tripId": "trip_001",
  "title": "서울 여행",
  "startDate": "2026-10-20",
  "endDate": "2026-10-22",
  "items": [
    {
      "itemId": "item_001",
      "candidateId": "cand_001",
      "visitDate": "2026-10-20",
      "visitOrder": 1,
      "memo": "오전에 방문",
      "place": {
        "placeId": "place_001",
        "name": "예시 카페",
        "address": "서울특별시 예시 주소 1",
        "latitude": 37.5,
        "longitude": 127.0
      }
    },
    {
      "itemId": "item_002",
      "candidateId": "cand_002",
      "visitDate": "2026-10-20",
      "visitOrder": 2,
      "memo": "",
      "place": {
        "placeId": "place_003",
        "name": "예시 공원",
        "address": "서울특별시 예시 주소 3",
        "latitude": 37.51,
        "longitude": 127.01
      }
    }
  ]
}
```

#### 응답 필드

| 필드 | 타입 | 설명 |
|---|---|---|
| tripId | string | 여행 ID |
| title | string | 여행 제목 |
| startDate | string | 여행 시작일 |
| endDate | string | 여행 종료일 |
| items | array | 저장된 일정 항목 목록 |
| items[].itemId | string | 일정 항목 ID |
| items[].candidateId | string | 방문 후보 ID |
| items[].visitDate | string | 방문 날짜 |
| items[].visitOrder | integer | 해당 날짜의 방문 순서 |
| items[].memo | string | 사용자 메모 |
| items[].place | object | 일정에 포함된 장소 정보 |
| items[].place.placeId | string | 장소 ID |
| items[].place.name | string | 장소명 |
| items[].place.address | string | 주소 |
| items[].place.latitude | number | 위도 |
| items[].place.longitude | number | 경도 |

#### 처리 규칙

- 일정은 visitDate 오름차순, visitOrder 오름차순으로 반환한다.
- 저장된 일정이 없으면 items를 빈 배열로 반환한다.
- 일정이 없는 경우도 정상 조회이므로 200 OK를 반환한다.
- 앱은 visitDate를 기준으로 일정을 날짜별로 나누어 표시한다.
- 앱은 장소의 latitude, longitude를 이용하여 지도 마커를 표시한다.
- 앱은 같은 날짜의 장소를 visitOrder 순서대로 연결할 수 있다.
- 이 응답은 실제 도로 경로, 이동 거리, 이동 소요시간을 제공하지 않는다.
- 실제 도로 경로와 이동 소요시간이 필요하면 별도의 경로 조회 기능을 설계한다.

### 5.8 일정 항목 메모 수정

저장된 일정 항목의 사용자 메모를 수정한다.

날짜 변경, 방문 순서 변경, 일정 항목 추가 및 제거는
일정 전체 저장 API를 사용한다.

- Method: PATCH
- URL: /api/v1/trips/{tripId}/itinerary-items/{itemId}
- 인증 필요 여부: 미정

#### Path Parameter

| 필드 | 타입 | 필수 여부 | 설명 |
|---|---|---|---|
| tripId | string | 필수 | 여행 ID |
| itemId | string | 필수 | 수정할 일정 항목 ID |

#### 요청 Body

```json
{
  "memo": "점심 식사 후 방문하기"
}
```

#### 요청 필드

| 필드 | 타입 | 필수 여부 | 설명 |
|---|---|---|---|
| memo | string | 필수 | 변경할 메모. 빈 문자열이면 메모를 비운다 |

#### 성공 응답

HTTP 상태: 200 OK

```json
{
  "itemId": "item_001",
  "candidateId": "cand_001",
  "visitDate": "2026-10-20",
  "visitOrder": 1,
  "memo": "점심 식사 후 방문하기"
}
```

#### 오류 응답 — 일정 항목 없음

HTTP 상태: 404 Not Found

```json
{
  "error": {
    "code": "ITINERARY_ITEM_NOT_FOUND",
    "message": "일정 항목을 찾을 수 없습니다."
  }
}
```

#### 처리 규칙

- itemId는 요청한 tripId의 일정에 속해야 한다.
- memo 필드를 생략하거나 null로 보내면 오류를 반환한다.
- memo가 빈 문자열이면 기존 메모를 비운다.
- 방문 날짜와 방문 순서는 변경하지 않는다.
- 날짜 및 순서 변경은 일정 전체 저장 API를 사용한다.

## 6. 공통 오류 응답

모든 API의 오류 응답은 아래 구조를 사용한다.

```json
{
  "error": {
    "code": "INVALID_REQUEST",
    "message": "요청 내용을 확인해 주세요.",
    "field": "title"
  }
}
```

| 필드 | 타입 | 설명 |
|---|---|---|
| error.code | string | 앱에서 오류를 구분하기 위한 코드 |
| error.message | string | 사용자에게 표시할 수 있는 오류 설명 |
| error.field | string | 오류가 발생한 요청 필드. 특정 필드와 관계없으면 생략 |

앱은 error.message 문구가 아닌 error.code를 기준으로
오류 유형을 구분한다.

### 공통 오류 목록

| HTTP 상태 | 코드 | 설명 |
|---|---|---|
| 400 | INVALID_REQUEST | 필수값 누락 또는 잘못된 요청 |
| 400 | INVALID_DATE_FORMAT | 잘못된 날짜 형식 또는 존재하지 않는 날짜 |
| 400 | INVALID_TRIP_DATE_RANGE | 종료일이 시작일보다 빠름 |
| 400 | UNSUPPORTED_SHARE_URL | 지원하지 않는 공유 링크 |
| 400 | EMPTY_PLACE_SELECTION | 선택한 장소가 없음 |
| 401 | UNAUTHORIZED | 인증 정보가 없거나 유효하지 않음 |
| 403 | ACCESS_DENIED | 요청한 자원에 접근할 권한이 없음 |
| 404 | TRIP_NOT_FOUND | 여행을 찾을 수 없음 |
| 404 | IMPORT_NOT_FOUND | 분석 작업을 찾을 수 없음 |
| 404 | CANDIDATE_NOT_FOUND | 방문 후보를 찾을 수 없음 |
| 404 | ITINERARY_ITEM_NOT_FOUND | 일정 항목을 찾을 수 없음 |
| 409 | IMPORT_NOT_COMPLETED | 링크 분석이 완료되지 않음 |
| 409 | DUPLICATE_VISIT_ORDER | 같은 날짜에 방문 순서가 중복됨 |
| 422 | PLACE_NOT_SELECTABLE | 정보가 불완전하여 선택할 수 없는 장소 |
| 422 | VISIT_DATE_OUT_OF_RANGE | 방문 날짜가 여행 기간을 벗어남 |
| 422 | INVALID_VISIT_ORDER | 방문 순서가 1부터 연속되지 않음 |
| 422 | DUPLICATE_SCHEDULED_CANDIDATE | 같은 후보가 일정에 여러 번 포함됨 |
| 500 | INTERNAL_SERVER_ERROR | 서버 내부 오류 |

인증 관련 오류는 인증 방식 확정 후 적용한다.

### 비동기 분석 실패와 조회 오류 구분

- 분석 작업을 정상적으로 조회했으면 HTTP 상태는 200이다.
- 조회된 분석 작업이 실패한 경우 응답의 status는 FAILED이다.
- 이 경우 응답 안의 error에 분석 실패 이유를 제공한다.
- 분석 작업 자체가 존재하지 않으면 HTTP 상태는 404이다.
- SHARE_LINK_UNAVAILABLE은 분석 작업 실패 시 사용하는 코드이다.

## 7. 공통 데이터 및 처리 규칙

### 데이터 형식

- 요청과 응답은 JSON을 사용한다.
- JSON 필드 이름은 camelCase를 사용한다.
- 날짜는 YYYY-MM-DD 형식을 사용한다.
- 날짜에는 시각 및 시간대 정보를 포함하지 않는다.
- 모든 ID는 문자열로 전달한다.
- 앱은 ID의 내부 형식이나 접두어에 의존하지 않는다.
- 위도와 경도는 숫자로 전달한다.
- 위도는 -90 이상 90 이하, 경도는 -180 이상 180 이하이다.
- 배열에 결과가 없으면 null 대신 빈 배열을 반환한다.

### 기본 범위

- 현재 초안은 한 번의 요청으로 전체 후보 및 일정을 조회한다.
- 페이지 나누기는 현재 범위에 포함하지 않는다.
- 현재 초안은 같은 후보의 반복 방문을 허용하지 않는다.
- 현재 초안은 한 사용자가 일정을 편집하는 흐름을 기준으로 한다.
- 공동 편집과 동시 수정 충돌 처리는 별도로 설계한다.

### 저장 규칙

- 방문 후보 저장과 일정 저장은 구분한다.
- 일정에서 장소를 제거해도 방문 후보는 유지한다.
- 일정 전체 저장은 한 번에 성공하거나 한 번에 실패해야 한다.
- 여행, 분석 작업, 방문 후보, 일정 항목에 대한 접근 권한은
  인증 설계 시 함께 정의한다.

### 예시 데이터

- 문서의 ID, 장소명, 주소, 좌표는 설명용 가상 데이터이다.
- 문서의 URL은 우리 서비스의 API 경로이다.
- 외부 지도 서비스가 제공하는 API 주소를 의미하지 않는다.

  
## 8. 미확정 사항

### 공유 링크 분석

- 지원할 지도 서비스
- 단일 장소 링크 지원 여부
- 저장 폴더 공유 링크 지원 여부
- 실제 공유 링크에서 장소명, 주소, 좌표를 추출할 수 있는지
- 비공개, 만료, 접근 제한 링크 처리
- 분석 작업의 최대 대기 시간 및 결과 보관 기간
- 장소 ID가 없는 경우의 중복 판별 기준
- 주소 및 좌표 누락 시 보완 방식

### 여행 및 일정

- 날짜를 정하지 않은 여행 생성 허용 여부
- 여행 제목 및 메모의 최대 길이
- 여행별 후보 및 일정 항목 최대 개수
- 같은 장소의 반복 방문 허용 여부
- 방문 후보 삭제 기능
- 여행 목록 조회, 여행 정보 수정 및 여행 삭제 기능
- 로그인 및 인증 방식
- 사용자별 데이터 접근 권한

## 9. AI P.S. 연동 범위

AI 팁은 일정 저장 이후 별도 작업으로 생성하는 방향을 검토한다.

### 생성할 정보

- 예상 소요시간
- 추천 방문 시간대
- 주차 관련 팁
- 입장료 관련 팁

### 설계 원칙

- AI 팁 생성 실패로 일정 저장이 실패하지 않아야 한다.
- 생성 대기, 생성 중, 완료, 실패 상태를 구분한다.
- 확인되지 않은 주차 및 입장료 정보를 확정 정보로 표시하지 않는다.
- 정보가 없으면 임의로 생성하지 않고 미확인 상태로 표시한다.
- 추정 정보와 출처로 확인한 정보를 구분한다.
- 사실 정보의 출처와 확인 시점을 저장하는 방식을 정한다.
- 팁의 재사용 기간과 재생성 조건을 정한다.

### 후속 명세 항목

- AI 팁 조회 방식
- 최초 생성 시점
- 생성 실패 후 재시도 방식
- 응답 필드 및 출처 형식

AI P.S.는 필수 MVP 기능이므로 개발 시작 전 후속 명세를 확정한다.

## 10. 지도 표시 및 길찾기 연동 범위

### 지도 표시

- 일정 조회 응답의 위도와 경도를 사용하여 마커를 표시한다.
- 같은 날짜의 장소를 방문 순서대로 연결한다.
- 단순 연결선과 실제 도로 이동 경로를 구분한다.
- 실제 이동 경로 및 소요시간 제공 여부는 별도로 결정한다.

### 외부 지도 앱 길찾기

- 일정 카드에서 네이버 지도 또는 카카오맵 길찾기를 실행한다.
- 목적지 정보는 일정 조회 응답의 장소명과 좌표를 사용한다.
- 현재 위치 접근 및 지도 앱 실행 처리는 앱에서 담당한다.
- 외부 지도 앱이 설치되지 않은 경우의 동작을 정한다.
- 딥링크 형식과 지원 환경은 구현 시 공식 문서로 확인한다.
- 기본 설계에서는 길찾기 실행을 위한 별도 서버 API를 만들지 않는다.

## 11. 별도 검토 기능

- 사진 업로드
- 백그라운드 GIF 앨범 생성
- GIF 생성 상태 조회 및 결과 다운로드

GIF 앨범은 회의 후 구현 범위와 API를 별도로 정의한다.

