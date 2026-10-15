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

아래에 API별 요청, 응답, 오류, 처리 규칙을 작성한다.

## 6. 미확정 사항

- 지원할 지도 공유 링크의 종류
- 네이버 저장 폴더 공유 링크에서 장소를 추출할 수 있는지
- 로그인 및 인증 방식
- 중복 장소 처리 기준
