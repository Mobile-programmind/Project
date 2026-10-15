# API 세부 명세서

- 문서 버전: v0.2
- 문서 상태: 팀 검토용 초안
- 작성일: 2026-10-15
- 대상 플랫폼: Android

## 1. 문서 목적 및 개발 범위

### 1.1 문서 목적

Android 앱에서 사용하는 데이터 구조,
Firebase 연동 작업, 입력 검증, 접근 권한 및 동기화 규칙을 정의한다.

프론트엔드 화면 개발과 데이터 처리 구현 시 본 문서를 공통 기준으로 사용한다.

현재 설계는 별도 REST 서버 없이 Firebase를 사용하는 구조이다.
따라서 본 문서에서는 HTTP API 주소 대신 데이터 경로와 처리 작업을 정의한다.

### 1.2 서비스 개요

본 서비스는 장소 링크와 직접 입력한 장소를 여행별로 모으고,
동행자와 날짜별 일정을 함께 편집하며,
여행 후 사진과 한 줄 후기를 일정에 연결하는 앱이다.

### 1.3 필수 개발 범위

| 구분 | 기능 | 설명 |
|---|---|---|
| 사용자 | 로그인 및 로그아웃 | 사용자 식별 및 여행 접근 권한 관리 |
| 여행 | 여행 생성·목록·상세·수정 | 여행 이름, 기간, 지역 관리 |
| 참여 | 초대 코드 생성 및 여행 참여 | 동행자가 같은 여행에 참여 |
| 장소 | 장소 등록·조회·수정·삭제 | 링크 또는 직접 입력으로 장소 보관 |
| 일정 | 날짜·시간·순서 지정 및 수정 | 참여자가 공동으로 일정 편집 |
| 기록 | 사진 및 한 줄 후기 등록·조회·수정·삭제 | 일정 항목에 여행 기록 연결 |
| 공유 | 날짜별 일정 텍스트 공유 | 시스템 공유 기능으로 메신저에 전달 |
| 로컬 저장 | 최근 여행 및 일정 캐시 | Room 기반 읽기용 캐시 |

### 1.4 선택 및 후속 기능

| 구분 | 기능 |
|---|---|
| 선택 | 좌표가 있는 장소의 지도 표시 |
| 선택 | GIF 앨범 생성 |
| 후속 | 여행지 추천 |
| 후속 | AI 기반 장소 참고 정보 |
| 후속 | 외부 공유 링크의 장소 정보 자동 추출 |

### 1.5 장소 링크 처리 범위

- 첫 버전은 원본 URL을 저장한다.
- 사용자가 장소명을 직접 입력하거나 확인한다.
- 공유 링크만으로 장소명, 주소, 좌표가 자동 입력된다고 가정하지 않는다.
- 주소와 좌표는 없는 상태로 저장할 수 있다.
- 좌표가 없는 장소는 원본 URL이 있으면 해당 링크를 열 수 있다.
- 원본 URL도 없으면 앱에 저장된 장소 정보만 표시한다.

---

## 2. 기술 구성 및 공통 규칙

### 2.1 기술 구성

| 영역 | 기술 | 역할 |
|---|---|---|
| Android 앱 | Kotlin | 화면 및 앱 기능 구현 |
| 사용자 인증 | Firebase Authentication | 사용자 식별 |
| 공동 데이터 | Cloud Firestore | 여행, 참여자, 장소, 일정, 기록 저장 |
| 사진 파일 | Firebase Storage | 사진 파일 저장 |
| 로컬 캐시 | Room | 최근 여행 및 일정의 읽기용 저장 |
| URL 수신 | Android 공유 Intent | 다른 앱에서 공유한 링크 수신 |
| 사진 선택 | Android Photo Picker | 사용자 사진 선택 |
| 일정 공유 | Android 시스템 공유 | 일정 텍스트를 다른 앱에 전달 |

화면 구현 방식은 XML 또는 Jetpack Compose 

### 2.2 사용자 식별

- 로그인한 사용자는 Firebase Authentication의 uid로 식별한다.
- uid를 사용자 ID로 사용한다.
- 로그인 제공 방식은 팀에서 확정한다.
- 로그인하지 않은 사용자는 여행 데이터에 접근할 수 없다.
- 사용자 ID를 요청에 넣었다는 이유만으로 권한을 인정하지 않는다.
- 실제 인증된 uid와 여행 참여 정보를 기준으로 접근을 허용한다.

### 2.3 필드 이름 및 값 형식

| 항목 | 규칙 |
|---|---|
| 필드 이름 | camelCase |
| 문서 ID | 문자열 |
| 날짜 | YYYY-MM-DD |
| 방문 시간 | HH:mm, 24시간제 |
| 생성·수정 시각 | Firestore Timestamp |
| 표시용 시각 예시 | ISO 8601 문자열 |
| 위도 | -90 이상 90 이하 |
| 경도 | -180 이상 180 이하 |
| 조회 결과 없음 | 빈 목록 |
| 선택값 없음 | 명세에 따라 null 허용 |

### 2.4 날짜 및 시간 기준

- 현재 MVP의 여행 날짜와 방문 시간은 한국 시간대를 기준으로 한다.
- 여행의 timeZone은 Asia/Seoul로 저장한다.
- 해외 여행 및 복수 시간대 지원은 후속 검토한다.
- createdAt과 updatedAt은 서버 기준 시각을 사용한다.
- 문서 예시의 시각 문자열은 설명용이며 실제 저장 타입은 Timestamp이다.

### 2.5 초안의 입력 제한

아래 제한은 구현을 위한 제안값이며 팀 검토 후 확정한다.

| 항목 | 제안 제한 |
|---|---|
| 여행 이름 | 앞뒤 공백 제거 후 1~50자 |
| 여행 지역 | 앞뒤 공백 제거 후 1~100자 |
| 장소명 | 앞뒤 공백 제거 후 1~100자 |
| 주소 | 최대 300자 |
| 원본 URL | 최대 2,048자 |
| 장소 메모 | 최대 500자 |
| 일정 메모 | 최대 500자 |
| 한 줄 후기 | 최대 200자 |

### 2.6 공통 입력 검증

- 필수 문자열은 빈 문자열이나 공백만으로 저장할 수 없다.
- 날짜는 실제 존재하는 날짜여야 한다.
- 여행 종료일은 시작일과 같거나 이후여야 한다.
- 방문 날짜는 여행 기간 안에 있어야 한다.
- URL은 http 또는 https 형식만 허용한다.
- URL 형식이 유효해도 실제 접근 가능 여부는 보장하지 않는다.
- 위도와 경도는 함께 입력하거나 함께 null로 저장한다.
- 좌표가 없다는 이유만으로 장소 등록을 막지 않는다.
- 앱 화면 검증과 별개로 보안 규칙 또는 신뢰할 수 있는 처리 계층에서 검증한다.

### 2.7 공통 관리 필드

변경 가능한 주요 문서에는 다음 필드를 사용한다.

| 필드 | 타입 | 설명 |
|---|---|---|
| createdBy | string | 최초 생성자 uid |
| createdAt | Timestamp | 최초 생성 시각 |
| updatedBy | string | 마지막 수정자 uid |
| updatedAt | Timestamp | 마지막 수정 시각 |
| version | integer | 동시 수정 충돌 확인용 버전 |

- version은 최초 저장 시 1이다.
- 수정 성공 시 version을 1 증가시킨다.
- createdBy와 createdAt은 일반 수정으로 변경하지 않는다.
- 작성자 및 수정자 값은 인증된 사용자와 일치해야 한다.

---

## 3. 데이터 구조 및 저장 경로

### 3.1 Firestore 경로

```text
users/{uid}

trips/{tripId}
  members/{uid}
  places/{placeId}
  scheduleItems/{itemId}
    memories/{memoryId}

invitations/{invitationId}
```

- 문서 ID는 각 데이터를 구분하는 식별자이다.
- 문서 예시의 tripId, placeId 등은 문서 ID를 표시한 것이다.
- Android 데이터 모델에서는 문서 ID를 별도 속성으로 보관할 수 있다.
- 같은 ID를 문서 필드에도 중복 저장할지는 구현 시 통일한다.

### 3.2 User

저장 경로:

```text
users/{uid}
```

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| displayName | string | O | 앱에 표시할 사용자 이름 |
| createdAt | Timestamp | O | 최초 등록 시각 |
| updatedAt | Timestamp | O | 프로필 수정 시각 |

이메일 등 추가 개인정보는 필요한 경우에만 저장한다.

### 3.3 Trip

저장 경로:

```text
trips/{tripId}
```

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| title | string | O | 여행 이름 |
| region | string | O | 여행 지역 |
| startDate | string | O | 여행 시작일 |
| endDate | string | O | 여행 종료일 |
| timeZone | string | O | MVP는 Asia/Seoul |
| ownerId | string | O | 여행 생성자 uid |
| memberIds | array of string | O | 현재 참여자 uid 목록 |
| itineraryVersion | integer | O | 일정 구성 전체의 변경 버전 |
| createdBy | string | O | 생성자 uid |
| createdAt | Timestamp | O | 생성 시각 |
| updatedBy | string | O | 마지막 수정자 uid |
| updatedAt | Timestamp | O | 수정 시각 |
| version | integer | O | 여행 문서 변경 버전 |

memberIds는 내 여행 목록 조회 및 접근 판단을 위한 값이다.
members 하위 문서와 항상 일치하도록 관리한다.

일반 여행 정보 수정으로 ownerId와 memberIds를 변경할 수 없다.

### 3.4 Member

저장 경로:

```text
trips/{tripId}/members/{uid}
```

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| role | string | O | OWNER 또는 MEMBER |
| displayName | string | O | 참여 시점의 표시 이름 |
| joinedAt | Timestamp | O | 참여 시각 |

- OWNER는 여행 생성자이다.
- MEMBER는 초대 코드로 참여한 사용자이다.
- 여행 생성 시 OWNER 참여 문서를 함께 생성한다.
- 현재 MVP에서 여행 이름·기간·지역 수정은 OWNER만 가능하다.
- 장소 및 일정 편집은 OWNER와 MEMBER 모두 가능하다.
- 여행 기록 수정·삭제는 작성자 또는 OWNER만 가능하다.
- 소유권 이전, 강제 퇴장, 여행 탈퇴는 후속 설계 대상으로 둔다.

### 3.5 Place

저장 경로:

```text
trips/{tripId}/places/{placeId}
```

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| name | string | O | 장소명 |
| address | string 또는 null | O | 주소 |
| sourceUrl | string 또는 null | O | 원본 장소 링크 |
| latitude | number 또는 null | O | 위도 |
| longitude | number 또는 null | O | 경도 |
| memo | string | O | 장소 메모 |
| sourceType | string | O | LINK 또는 MANUAL |
| scheduledItemId | string 또는 null | O | 현재 연결된 일정 항목 ID |
| createdBy | string | O | 등록자 uid |
| createdAt | Timestamp | O | 등록 시각 |
| updatedBy | string | O | 마지막 수정자 uid |
| updatedAt | Timestamp | O | 수정 시각 |
| version | integer | O | 장소 변경 버전 |

- LINK는 원본 URL을 함께 등록한 장소이다.
- MANUAL은 원본 URL 없이 직접 입력한 장소이다.
- 장소 등록 직후 scheduledItemId는 null이다.
- 현재 초안에서는 한 장소를 일정에 한 번만 배치한다.
- 장소명이 같아도 자동으로 동일 장소로 병합하지 않는다.

### 3.6 ScheduleItem

저장 경로:

```text
trips/{tripId}/scheduleItems/{itemId}
```

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| placeId | string | O | 같은 여행에 속한 장소 ID |
| visitDate | string | O | 방문 날짜 |
| visitTime | string 또는 null | O | 방문 시간 |
| visitOrder | integer | O | 같은 날짜 안에서의 방문 순서 |
| memo | string | O | 일정 메모 |
| memoryCount | integer | O | 연결된 여행 기록 수 |
| createdBy | string | O | 생성자 uid |
| createdAt | Timestamp | O | 생성 시각 |
| updatedBy | string | O | 마지막 수정자 uid |
| updatedAt | Timestamp | O | 수정 시각 |
| version | integer | O | 일정 항목 변경 버전 |

- 장소 정보는 placeId로 연결한다.
- 일정 표시 시 Place의 장소명, 주소, 좌표를 함께 조회한다.
- 일정 표시 순서는 visitDate, visitOrder 기준이다.
- visitTime은 선택값이며 자동으로 방문 순서를 변경하지 않는다.
- memoryCount는 기록 생성·삭제 시 함께 변경한다.

### 3.7 Memory

저장 경로:

```text
trips/{tripId}/scheduleItems/{itemId}/memories/{memoryId}
```

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| review | string | O | 한 줄 후기. 사진만 있으면 빈 문자열 허용 |
| photos | array of object | O | 사진 정보 목록 |
| createdBy | string | O | 작성자 uid |
| createdAt | Timestamp | O | 작성 시각 |
| updatedBy | string | O | 마지막 수정자 uid |
| updatedAt | Timestamp | O | 수정 시각 |
| version | integer | O | 기록 변경 버전 |

photos 항목:

| 필드 | 타입 | 설명 |
|---|---|---|
| photoId | string | 사진 식별자 |
| storagePath | string | Firebase Storage 파일 경로 |
| contentType | string | 이미지 MIME 유형 |
| sizeBytes | integer | 업로드된 파일 크기 |
| order | integer | 기록 안에서의 표시 순서 |

사진 파일 자체를 Firestore 문서에 저장하지 않는다.

### 3.8 Invitation

저장 경로:

```text
invitations/{invitationId}
```

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| tripId | string | O | 참여할 여행 ID |
| codeHash | string | O | 초대 코드 검증용 값 |
| active | boolean | O | 사용 가능 여부 |
| expiresAt | Timestamp | O | 만료 시각 |
| createdBy | string | O | 생성자 uid |
| createdAt | Timestamp | O | 생성 시각 |

- 초대 코드 원문을 일반 조회 가능한 문서로 공개하지 않는다.
- 초대 검증용 데이터는 일반 사용자가 직접 조회할 수 없도록 한다.
- 초대 생성·참여는 검증된 처리 계층을 통해 수행한다.
- 구체적인 처리 계층은 8번 미확정 사항에서 확정한다.

### 3.9 Firebase Storage 경로

```text
trips/{tripId}/items/{itemId}/memories/{memoryId}/{photoId}
```

사진은 해당 여행 참여자만 접근할 수 있어야 한다.
파일 경로만 알고 있다는 이유로 접근을 허용하지 않는다.

---

## 4. 사용자 및 여행 연동

### 4.1 로그인

입력:

- 팀에서 선택한 로그인 제공 방식의 인증 정보

처리:

1. Firebase Authentication으로 인증한다.
2. 인증된 uid를 확보한다.
3. 최초 로그인이라면 users/{uid}를 생성한다.
4. 로그인 성공 후 내 여행 목록으로 이동한다.

결과:

```json
{
  "uid": "user_001",
  "displayName": "사용자 A"
}
```

규칙:

- 비밀번호를 Firestore에 직접 저장하지 않는다.
- 로그인 취소와 로그인 실패를 구분한다.
- 사용자 표시 이름의 등록 방식은 팀에서 확정한다.

### 4.2 로그아웃

처리:

1. 현재 화면의 데이터 구독을 종료한다.
2. 인증 세션을 로그아웃한다.
3. 현재 사용자용 화면 상태를 비운다.
4. 로그아웃한 사용자의 로컬 캐시를 제거한다.
5. 로그인 화면으로 이동한다.

### 4.3 여행 생성

입력 예시:

```json
{
  "title": "서울 여행",
  "region": "서울",
  "startDate": "2026-10-20",
  "endDate": "2026-10-22"
}
```

처리:

- trips/{tripId}를 생성한다.
- ownerId는 현재 인증된 uid로 지정한다.
- memberIds에는 생성자 uid를 넣는다.
- timeZone은 Asia/Seoul로 저장한다.
- version은 1로 저장한다.
- itineraryVersion은 0으로 저장한다.
- members/{uid}에 OWNER 문서를 생성한다.
- 여행 문서와 OWNER 참여 문서는 하나의 원자적 작업으로 생성한다.

결과 예시:

```json
{
  "tripId": "trip_001",
  "title": "서울 여행",
  "region": "서울",
  "startDate": "2026-10-20",
  "endDate": "2026-10-22",
  "timeZone": "Asia/Seoul",
  "ownerId": "user_001",
  "memberIds": ["user_001"],
  "version": 1,
  "itineraryVersion": 0
}
```

### 4.4 내 여행 목록 조회

조회 조건:

- memberIds에 현재 인증된 uid가 포함된 여행

표시 정보:

- 여행 ID
- 여행 이름
- 지역
- 시작일
- 종료일
- 참여자 수

규칙:

- 최근 수정된 여행부터 표시한다.
- 참여한 여행이 없으면 빈 목록을 표시한다.
- 참여하지 않은 여행을 전체 조회하지 않는다.
- 필요한 Firestore 인덱스는 구현 시 설정한다.

### 4.5 여행 상세 조회

입력:

```json
{
  "tripId": "trip_001"
}
```

결과:

- 여행 기본 정보
- 현재 사용자 역할
- 참여자 목록
- 일정 조회에 필요한 itineraryVersion

규칙:

- 여행 참여자만 조회할 수 있다.
- 여행이 없거나 접근 권한이 없으면 적절한 오류를 표시한다.

### 4.6 여행 정보 수정

권한:

- OWNER만 가능

입력 예시:

```json
{
  "tripId": "trip_001",
  "expectedVersion": 1,
  "title": "서울 친구 여행",
  "region": "서울",
  "startDate": "2026-10-20",
  "endDate": "2026-10-23"
}
```

규칙:

- expectedVersion이 현재 version과 같아야 한다.
- 수정 성공 시 version을 1 증가시킨다.
- ownerId, memberIds는 이 작업으로 변경할 수 없다.
- 기간 변경 후 기존 일정이 여행 기간 밖으로 나가면 변경을 거절한다.
- 기간과 일정 검증 중 충돌이 발생하면 최신 데이터를 다시 확인한다.
- 기간 변경에 따른 일정 자동 삭제는 수행하지 않는다.

---

## 5. 초대 및 참여자 연동

### 5.1 초대 코드 생성

권한:

- OWNER만 가능

입력:

```json
{
  "tripId": "trip_001"
}
```

결과 예시:

```json
{
  "invitationId": "invite_001",
  "code": "예시초대코드",
  "expiresAt": "2026-10-16T12:00:00+09:00"
}
```

규칙:

- 초대 코드는 추측하기 어렵게 생성한다.
- 코드 길이와 만료 시간은 팀에서 확정한다.
- 여행별 활성 초대 코드는 하나로 제한하는 방향을 기본으로 한다.
- 새 초대 코드 생성 시 이전 활성 코드를 무효화한다.
- 코드 생성 및 반환은 검증된 처리 계층에서 수행한다.
- 일반 사용자가 invitations 전체 목록을 읽을 수 없도록 한다.

### 5.2 초대 코드로 여행 참여

입력:

```json
{
  "code": "예시초대코드"
}
```

처리:

1. 로그인 여부를 확인한다.
2. 초대 코드가 유효하고 만료되지 않았는지 확인한다.
3. 연결된 여행이 존재하는지 확인한다.
4. 이미 참여 중인지 확인한다.
5. MEMBER 참여 문서를 생성한다.
6. Trip.memberIds에 uid를 추가한다.

결과 예시:

```json
{
  "tripId": "trip_001",
  "role": "MEMBER",
  "alreadyJoined": false
}
```

규칙:

- 이미 참여 중이면 중복 참여 문서를 만들지 않는다.
- 이미 참여 중이면 alreadyJoined를 true로 반환한다.
- members 문서와 memberIds는 원자적으로 변경한다.
- 초대 코드 검증 없이 사용자가 임의의 여행에 자신을 추가할 수 없다.
- 다른 사용자의 uid를 대신 추가할 수 없다.
- 여행별 최대 참여 인원과 코드 입력 시도 제한은 팀에서 확정한다.

### 5.3 참여자 목록 조회

권한:

- 해당 여행 참여자

결과 예시:

```json
{
  "tripId": "trip_001",
  "members": [
    {
      "uid": "user_001",
      "displayName": "사용자 A",
      "role": "OWNER"
    },
    {
      "uid": "user_002",
      "displayName": "사용자 B",
      "role": "MEMBER"
    }
  ]
}
```

---

## 6. 장소 및 공동 일정 연동

### 6.1 장소 등록

권한:

- 해당 여행 참여자

입력 예시:

```json
{
  "tripId": "trip_001",
  "name": "예시 카페",
  "address": "서울특별시 예시 주소",
  "sourceUrl": "https://naver.me/예시링크", 
  "latitude": null,
  "longitude": null,
  "memo": "둘째 날 방문 후보"
}
```
 //모바일 프로그래밍 제출용 앱에서는 구글로 링크를 변환하여 url을 사용할 수 있도록 한다.
 //향후 디벨롭 과정에서 네이버나 타 여행용 앱의 URL을 파싱할 수 있도록 수정한다.

처리:

- places/{placeId}를 생성한다.
- sourceUrl이 있으면 sourceType을 LINK로 저장한다.
- sourceUrl이 없으면 sourceType을 MANUAL로 저장한다.
- scheduledItemId는 null로 저장한다.
- version은 1로 저장한다.

결과 예시:

```json
{
  "placeId": "place_001",
  "name": "예시 카페",
  "address": "서울특별시 예시 주소",
  "sourceUrl": "https://naver.me/예시링크",
  "latitude": null,
  "longitude": null,
  "memo": "둘째 날 방문 후보",
  "sourceType": "LINK",
  "scheduledItemId": null,
  "version": 1
}
```

규칙:

- 장소명은 필수이다.
- 주소, URL, 좌표는 없는 상태로 저장할 수 있다.
- 외부 링크에서 장소 정보를 자동 추출하지 않는다.
- 동일한 장소명이 있어도 자동 병합하지 않는다.
- 같은 URL의 중복 등록 경고 여부는 팀에서 확정한다.

### 6.2 장소 보관함 조회

조회 경로:

```text
trips/{tripId}/places
```

규칙:

- 저장된 순서대로 표시한다.
- scheduledItemId가 null이면 미배치 장소로 표시한다.
- scheduledItemId가 있으면 일정에 배치된 장소로 표시한다.
- 날짜가 정해지지 않은 장소도 보관함에 유지한다.
- 결과가 없으면 빈 목록과 장소 추가 안내를 표시한다.

### 6.3 장소 수정

입력:

- placeId
- expectedVersion
- 변경할 장소명, 주소, URL, 좌표, 메모

규칙:

- 해당 여행 참여자만 수정할 수 있다.
- expectedVersion을 확인한다.
- 수정 성공 시 version을 1 증가시킨다.
- scheduledItemId는 일반 장소 수정으로 변경하지 않는다.
- 장소 정보 수정은 연결된 일정 화면에도 반영한다.

### 6.4 장소 삭제

규칙:

- 해당 여행 참여자만 삭제할 수 있다.
- scheduledItemId가 null인 장소만 삭제할 수 있다.
- 일정에 배치된 장소는 먼저 일정에서 제거해야 한다.
- 장소 삭제 시 다른 장소나 일정은 변경하지 않는다.

### 6.5 일정에 장소 배치

입력 예시:

```json
{
  "tripId": "trip_001",
  "placeId": "place_001",
  "visitDate": "2026-10-20",
  "visitTime": "10:00",
  "visitOrder": 1,
  "memo": "오전에 방문",
  "expectedItineraryVersion": 0
}
```

처리:

- scheduleItems/{itemId}를 생성한다.
- Place.scheduledItemId에 itemId를 저장한다.
- memoryCount는 0으로 저장한다.
- 일정 항목 version은 1로 저장한다.
- Trip.itineraryVersion을 1 증가시킨다.
- 관련 변경은 원자적으로 수행한다.

규칙:

- placeId는 같은 여행에 속해야 한다.
- 이미 일정에 배치된 장소는 다시 배치할 수 없다.
- visitDate는 여행 기간 안에 있어야 한다.
- visitTime은 null 또는 HH:mm 형식이어야 한다.
- 해당 날짜의 지정 위치에 삽입하고 뒤 항목의 순서를 조정한다.
- 날짜별 visitOrder는 1부터 연속되도록 유지한다.
- expectedItineraryVersion이 다르면 저장을 거절하고 최신 일정을 확인한다.

### 6.6 일정 조회 및 변경 반영

조회 경로:

```text
trips/{tripId}/scheduleItems
```

화면에 필요한 결과 예시:

```json
{
  "tripId": "trip_001",
  "itineraryVersion": 1,
  "items": [
    {
      "itemId": "item_001",
      "placeId": "place_001",
      "visitDate": "2026-10-20",
      "visitTime": "10:00",
      "visitOrder": 1,
      "memo": "오전에 방문",
      "memoryCount": 0,
      "version": 1,
      "place": {
        "name": "예시 카페",
        "address": "서울특별시 예시 주소",
        "sourceUrl": "https://naver.me/예시링크",
        "latitude": null,
        "longitude": null
      }
    }
  ]
}
```

위 결과는 ScheduleItem과 Place를 조합한 앱 표시용 모델이다.
place 객체를 일정 문서에 중복 저장한다는 의미는 아니다.

규칙:

- visitDate, visitOrder 순서로 표시한다.
- 참여자의 변경을 구독하여 화면에 반영한다.
- 실시간 반영과 동시 수정 충돌 처리는 별도로 구현한다.
- 화면을 벗어나면 불필요한 데이터 구독을 종료한다.
- 일정이 없으면 빈 목록을 표시한다.

### 6.7 일정 시간 및 메모 수정

입력 예시:

```json
{
  "itemId": "item_001",
  "expectedVersion": 1,
  "visitTime": "11:00",
  "memo": "점심 전에 방문"
}
```

규칙:

- 해당 여행 참여자만 수정할 수 있다.
- 입력한 필드만 변경한다.
- visitTime에 null을 보내면 시간을 비운다.
- memo에 빈 문자열을 보내면 메모를 비운다.
- expectedVersion이 다르면 최신 내용을 확인하도록 안내한다.
- 성공 시 일정 항목 version과 itineraryVersion을 증가시킨다.

### 6.8 일정 날짜 및 순서 변경

입력 예시:

```json
{
  "tripId": "trip_001",
  "expectedItineraryVersion": 3,
  "moves": [
    {
      "itemId": "item_001",
      "visitDate": "2026-10-21",
      "visitOrder": 1
    }
  ]
}
```

규칙:

- moves는 이동할 항목과 이동할 최종 위치를 나타낸다.
- 이동하지 않는 일정 항목은 삭제하지 않는다.
- 이동 전후 날짜의 전체 순서를 재정렬한다.
- 날짜별 순서는 1부터 연속되도록 유지한다.
- 여러 항목을 이동할 때 같은 최종 위치를 중복 지정할 수 없다.
- 날짜는 여행 기간 안에 있어야 한다.
- expectedItineraryVersion이 일치해야 한다.
- 관련 일정 항목 변경과 itineraryVersion 증가는 원자적으로 처리한다.
- 시간은 자동 변경하지 않는다.
- 시간 변경이 필요한 경우 별도로 수정한다.

### 6.9 일정 항목 제거

규칙:

- 해당 여행 참여자만 제거할 수 있다.
- 연결된 여행 기록이 있으면 제거를 거절한다.
- 여행 기록을 먼저 삭제한 후 일정에서 제거할 수 있다.
- 일정 항목을 제거해도 Place는 보관함에 유지한다.
- Place.scheduledItemId를 null로 변경한다.
- 같은 날짜의 남은 방문 순서를 재정렬한다.
- itineraryVersion을 증가시킨다.
- 제거와 순서 변경은 원자적으로 처리한다.

---

## 7. 여행 기록, 공유 및 로컬 캐시

### 7.1 사진 및 후기 등록

권한:

- 해당 여행 참여자

등록 데이터 예시:

```json
{
  "itemId": "item_001",
  "review": "분위기가 좋고 쉬어가기 좋았다.",
  "photos": [
    {
      "photoId": "photo_001",
      "storagePath": "trips/trip_001/items/item_001/memories/memory_001/photo_001",
      "contentType": "image/jpeg",
      "sizeBytes": 1250000,
      "order": 1
    }
  ]
}
```

처리 순서:

1. 기록 ID와 사진 ID를 준비한다.
2. Photo Picker로 사진을 선택한다.
3. 사진 파일을 검증하고 필요한 경우 크기를 조정한다.
4. Firebase Storage에 업로드한다.
5. 업로드 성공 후 Memory 문서를 생성한다.
6. ScheduleItem.memoryCount를 증가시킨다.

규칙:

- 후기가 있거나 사진이 한 장 이상 있어야 한다.
- 사진만 있는 기록은 review를 빈 문자열로 저장한다.
- 후기만 있는 기록은 photos를 빈 배열로 저장한다.
- 사진 선택 취소는 오류로 처리하지 않는다.
- 모든 사진 업로드가 끝난 뒤 기록 저장 완료를 표시한다.
- 문서 저장 실패 시 새로 업로드한 파일을 정리하거나 재시도 대상으로 관리한다.
- 재시도 시 같은 memoryId와 photoId를 사용하여 중복 생성을 방지한다.
- 파일 크기, 장수, 지원 형식은 팀에서 확정한다.
- 실패한 업로드나 임시 파일의 정리 방식을 구현 전에 정한다.
//후에 디벨롭 과정에서 네이버 후기 등록과 연결할 수 있는지 확인한다. 여러 리뷰 이벤트를 컨택할 수 있는지 또한 향후 디벨롭 과정에 포함 시킨다.

### 7.2 기록 조회

조회 경로:

```text
trips/{tripId}/scheduleItems/{itemId}/memories
```

규칙:

- 해당 여행 참여자만 조회할 수 있다.
- 생성 시각순으로 표시한다.
- 사진은 order 순서로 표시한다.
- 사진은 Storage 접근 권한을 적용하여 불러온다.
- 영구 공개 URL을 기본 접근 방식으로 사용하지 않는다.
- 기록이 없으면 빈 목록을 표시한다.

### 7.3 기록 수정

권한:

- 기록 작성자 또는 OWNER

입력:

- memoryId
- expectedVersion
- 변경할 후기
- 최종 사진 목록

규칙:

- expectedVersion을 확인한다.
- 새 사진은 업로드 완료 후 문서에 연결한다.
- 문서 변경 성공 후 제거된 사진 파일을 정리한다.
- 후기와 사진이 모두 없어지는 수정은 거절한다.
- 기록을 완전히 비우려면 기록 삭제 기능을 사용한다.
- 파일 정리에 실패하면 재시도 상태를 관리한다.

### 7.4 기록 삭제

권한:

- 기록 작성자 또는 OWNER

규칙:

- Memory 문서 삭제와 memoryCount 감소는 원자적으로 처리한다.
- 연결된 사진 파일도 삭제 대상으로 관리한다.
- Firestore와 Storage의 변경을 하나의 원자적 작업으로 가정하지 않는다.
- 파일 삭제 실패 시 재시도할 수 있도록 한다.
- 기록 삭제가 일정 항목 삭제로 이어지지는 않는다.

### 7.5 날짜별 일정 텍스트 공유

입력:

- tripId
- 공유할 날짜

출력 예시:

```text
[서울 여행 / 2026-10-20]

1. 10:00 예시 카페
   메모: 오전에 방문

2. 시간 미정 · 예시 공원
```

규칙:

- 조회한 일정으로 앱에서 공유 텍스트를 생성한다.
- Android 시스템 공유 기능으로 전달한다.
- 별도 서버 API는 필수로 만들지 않는다.
- 시간 미지정 항목은 시간 미정으로 표시한다.
- 일정은 visitOrder 순서로 출력한다.
- 캐시 데이터로 공유할 경우 최신 상태가 아닐 수 있음을 표시한다.
- 사진, 비공개 초대 코드, 사용자 개인정보는 기본 공유 내용에 넣지 않는다.

### 7.6 Room 읽기용 캐시

캐시 대상:

- 최근 조회한 여행
- 참여 정보
- 장소 보관함
- 일정 항목

규칙:

- 클라우드 데이터를 공동 편집의 기준으로 사용한다.
- Room은 읽기용 캐시로 사용한다.
- 캐시 데이터는 사용자 uid별로 분리한다.
- 저장 시 마지막 동기화 시각을 기록한다.
- 오프라인에서는 캐시와 마지막 동기화 시각을 표시한다.
- 현재 MVP에서는 오프라인 편집을 제공하지 않는다.
- 오프라인 상태에서는 클라우드 변경 작업을 비활성화한다.
- 네트워크가 연결되어 있어도 저장 완료는 서버 반영 확인 후 표시한다.
- 접근 권한이 사라진 여행은 해당 캐시를 제거한다.
- 로그아웃 시 해당 사용자의 캐시를 제거한다.

### 7.7 선택 기능: 지도 및 원본 링크

- 좌표가 있으면 지도에 장소를 표시할 수 있다.
- 단순 장소 연결선과 실제 도로 경로를 구분한다.
- 실제 이동 거리와 소요시간은 현재 필수 범위에 포함하지 않는다.
- 좌표가 없고 sourceUrl이 있으면 원본 링크를 연다.
- 지도 앱 딥링크를 추가하는 경우 지원 형식과 설치되지 않은 경우의 동작을 별도로 정의한다.

---

## 8. 접근 권한, 오류, 검증 및 미확정 사항

### 8.1 접근 권한

| 작업 | OWNER | MEMBER | 비참여자 |
|---|---|---|---|
| 여행 조회 | 가능 | 가능 | 불가 |
| 여행 기본 정보 수정 | 가능 | 불가 | 불가 |
| 초대 코드 생성 | 가능 | 불가 | 불가 |
| 참여자 조회 | 가능 | 가능 | 불가 |
| 장소 등록·수정·삭제 | 가능 | 가능 | 불가 |
| 일정 등록·수정·제거 | 가능 | 가능 | 불가 |
| 기록 조회·등록 | 가능 | 가능 | 불가 |
| 다른 사용자의 기록 수정·삭제 | 가능 | 불가 | 불가 |
| 자신의 기록 수정·삭제 | 가능 | 가능 | 불가 |
| 사진 조회 | 가능 | 가능 | 불가 |

권한 규칙은 Firestore와 Storage에 각각 적용한다.

화면에서 버튼을 숨기는 것만으로 접근 제어를 구현하지 않는다.

### 8.2 데이터 변경 제한

- 일반 사용자는 ownerId를 변경할 수 없다.
- 일반 여행 수정으로 memberIds를 변경할 수 없다.
- 유효한 초대 절차 없이 members 문서를 생성할 수 없다.
- 일반 장소 수정으로 scheduledItemId를 변경할 수 없다.
- 일반 일정 수정으로 memoryCount를 변경할 수 없다.
- createdBy와 createdAt을 임의로 바꿀 수 없다.
- 다른 여행의 placeId나 itemId를 연결할 수 없다.
- 보안 규칙에서 허용하는 필드와 타입을 제한한다.
- 특권 처리 계층에서도 인증, 역할, 입력값을 별도로 검증한다.

### 8.3 동시 수정 충돌

- 단일 문서 수정은 version을 확인한다.
- 일정 구성 변경은 itineraryVersion을 확인한다.
- 읽은 버전과 현재 버전이 다르면 덮어쓰지 않는다.
- 충돌 시 최신 데이터를 다시 보여주고 재수정하도록 안내한다.
- 관련 여러 문서의 변경은 트랜잭션 또는 원자적 쓰기로 처리한다.
- 실시간 구독만으로 충돌 처리가 완료되었다고 판단하지 않는다.

### 8.4 앱 내부 오류 모델

Firebase 등의 오류를 화면에서 일관되게 처리하기 위해
앱 내부에서 아래 구조로 변환한다.

이 구조는 HTTP 응답 형식을 의미하지 않는다.

```json
{
  "code": "EDIT_CONFLICT",
  "message": "다른 참여자가 먼저 수정했습니다. 최신 내용을 확인해 주세요.",
  "field": null,
  "retryable": false
}
```

| 코드 | 의미 | 사용자 안내 |
|---|---|---|
| AUTH_REQUIRED | 로그인 필요 | 로그인 후 다시 시도 |
| ACCESS_DENIED | 권한 없음 | 접근 권한 확인 |
| INVALID_INPUT | 입력값 오류 | 해당 입력값 수정 |
| TRIP_NOT_FOUND | 여행 없음 | 목록 새로고침 |
| PLACE_NOT_FOUND | 장소 없음 | 장소 목록 새로고침 |
| ITEM_NOT_FOUND | 일정 없음 | 일정 새로고침 |
| MEMORY_NOT_FOUND | 기록 없음 | 기록 목록 새로고침 |
| INVALID_INVITATION | 초대 코드 오류 | 코드 확인 |
| INVITATION_EXPIRED | 초대 만료 | 새 코드 요청 |
| EDIT_CONFLICT | 동시 수정 충돌 | 최신 내용 확인 후 재수정 |
| DATE_OUT_OF_RANGE | 여행 기간 밖 일정 | 날짜 수정 |
| PLACE_ALREADY_SCHEDULED | 이미 배치된 장소 | 기존 일정 확인 |
| PLACE_IN_USE | 일정에 연결된 장소 | 일정에서 먼저 제거 |
| ITEM_HAS_MEMORIES | 기록이 연결된 일정 | 기록 먼저 삭제 |
| INVALID_PHOTO | 지원하지 않는 사진 | 다른 사진 선택 |
| UPLOAD_FAILED | 사진 업로드 실패 | 재시도 |
| NETWORK_ERROR | 네트워크 오류 | 연결 확인 |
| INTERNAL_ERROR | 예상하지 못한 오류 | 잠시 후 재시도 |

- retryable은 실제 실패 상황에 따라 결정한다.
- 오류 메시지 문구 대신 code로 오류 유형을 구분한다.
- 내부 오류 세부 정보나 개인정보를 사용자 메시지에 노출하지 않는다.

### 8.5 필수 검증 시나리오

| 번호 | 시나리오 | 완료 기준 |
|---|---|---|
| 1 | 여행 생성 | 이름·기간·지역과 OWNER가 함께 저장됨 |
| 2 | 다른 계정 초대 | 두 기기에서 같은 여행을 조회함 |
| 3 | 잘못된 초대 코드 | 참여가 거절됨 |
| 4 | 링크 장소 등록 | 원본 URL과 사용자 입력 장소명이 저장됨 |
| 5 | 직접 장소 등록 | URL과 좌표 없이 저장됨 |
| 6 | 장소 3개 일정 배치 | 2일 일정으로 나누어 표시됨 |
| 7 | 일정 날짜·순서 변경 | 다른 기기에도 일관되게 반영됨 |
| 8 | 같은 항목 동시 수정 | 오래된 버전의 변경이 조용히 덮어쓰지 않음 |
| 9 | 사진과 후기 등록 | 일정에 연결되어 다른 참여자가 조회함 |
| 10 | 사진 선택 취소 | 기존 화면과 데이터가 유지됨 |
| 11 | 사진 업로드 실패 | 완료로 표시하지 않고 재시도 가능 |
| 12 | 기록이 있는 일정 제거 | 삭제가 거절됨 |
| 13 | 일정 텍스트 공유 | 날짜·시간·순서가 반영된 텍스트가 생성됨 |
| 14 | 비참여자 접근 | Firestore 및 Storage 접근이 거절됨 |
| 15 | 오프라인 조회 | 캐시와 마지막 동기화 시각이 표시됨 |
| 16 | 로그아웃 | 해당 사용자 캐시와 화면 상태가 제거됨 |

### 8.6 구현 전 확정할 사항

| 항목 | 현재 상태 |
|---|---|
| 로그인 제공 방식 | 미정 |
| 문서의 입력 길이 제한 | 제안값 검토 필요 |
| 여행별 최대 참여 인원 | 미정 |
| 초대 코드 길이·만료 시간·시도 제한 | 미정 |
| 초대 처리 계층 | Cloud Functions 등 사용 여부 결정 필요 |
| 사진 최대 장수·크기·지원 형식 | 미정 |
| 사진 및 임시 업로드 정리 방식 | 미정 |
| 참여자의 기록 편집 권한 | 작성자 또는 OWNER라는 초안 검토 필요 |
| 같은 장소 반복 방문 | 현재 초안은 불가 |
| 여행 탈퇴·강제 퇴장·삭제 | MVP 포함 여부 결정 필요 |
| 최대 장소·일정 수와 일괄 변경 범위 | 미정 |
| 지도 또는 GIF 중 선택 기능 | 필수 기능 완료 후 결정 |

### 8.7 초안 완료 기준

본 문서는 다음 흐름을 개발하기 위한 데이터 연동 초안이다.

여행 생성
→ 초대 코드로 동행자 참여
→ 장소 링크 또는 직접 입력
→ 날짜·시간·순서 지정
→ 공동 일정 수정
→ 사진 및 후기 작성
→ 일정 텍스트 공유

실제 구현 완료 여부는 두 계정·두 기기에서 위 흐름을 수행하고,
접근 권한 및 오류 시나리오를 검증한 후 판단한다.
