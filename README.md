# OWN — Music & Workout Social Platform

> 나만의(OWN) 운동과 음악을 기록하고 공유하는 SNS

운동할 때 들었던 음악과 그날의 운동 내용을 함께 기록하고 공유하는 피트니스 소셜 플랫폼입니다. 단순한 운동 기록이 아니라 감정과 음악을 더해 자기 동기부여와 루틴 공유, 공감형 소통을 지원합니다.

## 💡 Technical Decision & Key Challenges (핵심 기술적 고민)

> 무분별한 라이브러리 사용이나 단순 기능 구현을 넘어, **데이터의 정합성과 백엔드 기본기(RDB 설계)**를 고민하며 개발했습니다.

### 1. 다대다(N:M) 관계 모델링 및 DB 수준의 참조 무결성 보장
* **문제 상황**: 하나의 게시글(`Post`)에 여러 운동 태그(`Workout Type`)가 붙고, 반대로 하나의 운동 태그가 여러 게시글에 쓰이는 **다대다(N:M) 관계** 구조를 설계해야 했습니다.
* **해결 방안**: RDB의 구조적 한계를 해결하기 위해 중간 교차 테이블인 `post_workout_type`을 설계하여 1:N, N:1 관계로 정규화 및 해소했습니다.
* **핵심 고려 사항 (Data Integrity)**:
  * **복합 기본키(Composite PK) 설정**: `(post_id, workout_type_id)`를 묶어 PK로 지정함으로써, 동일한 게시글에 동일한 태그가 중복 매핑되는 오류를 원천 차단했습니다.
  * **`ON DELETE CASCADE` 제약조건 활용**: 사용자가 게시글을 삭제하거나 운동 종류가 변경될 때, 중간 테이블에 찌꺼기 데이터(고아 데이터)가 남지 않도록 DB 수준에서 참조 무결성을 완벽히 관리했습니다.

<details>
<summary><b>📐 실제 구현한 교차 테이블 DDL (SQL) 보기 (클릭)</b></summary>

```sql
DROP TABLE IF EXISTS post_workout_type;

CREATE TABLE post_workout_type (
    post_id INT NOT NULL,
    workout_type_id INT NOT NULL,
    PRIMARY KEY (post_id, workout_type_id),
    CONSTRAINT fk_post_workout_post FOREIGN KEY (post_id) 
        REFERENCES post(post_id) ON DELETE CASCADE,
    CONSTRAINT fk_post_workout_type FOREIGN KEY (workout_type_id) 
        REFERENCES workout_type(workout_type_id) ON DELETE CASCADE
);
</details>

<!-- 여기에 메인 화면 / 게시글 작성 화면 캡처 이미지 -->
<img width="1257" height="849" alt="own_image" src="https://github.com/user-attachments/assets/9e17f7c1-e59a-43b7-b033-e96c96fee761" />
<img width="1567" height="875" alt="own_image2" src="https://github.com/user-attachments/assets/e1be53c2-02d2-4a50-bed3-b31e6c99ef7e" />

---

### 🎯 목표
운동 기록에 "그날의 음악과 감정"이라는 맥락을 더해, 단순 기록이 아닌 공감형 피트니스 SNS 구현

### 🚀 구현 범위
백엔드는 Post / Music / Like / Bookmark 도메인 API, 프론트엔드는 사용자 관리·마이페이지 도메인(로그인/회원가입, 마이페이지 3개 탭)을 담당. Spotify API 연동으로 게시글 작성 중 실제 음악을 검색·첨부할 수 있도록 구현

### 🏆 성과
2인 팀 프로젝트, SSAFY 14기 1학기 **반 전체 2위**

---

## 기획 의도

운동 기록 서비스는 많지만, 그 순간의 감정과 함께한 음악까지 기록하는 서비스는 드물다고 생각했습니다. 운동 태그와 감정 태그를 함께 선택해 게시글을 작성하도록 설계해, 단순한 기록을 넘어 그날의 경험을 더 풍부하게 남길 수 있도록 했습니다. 또한 Spotify API를 연동해 게시글 작성 중 실제 들었던 음악을 직접 선택할 수 있게 했습니다.

---

## 핵심 기능

- **감정·운동 태그 게시글 작성**: 운동 태그와 감정 태그를 함께 선택해 그날의 운동과 감정을 기록
- **Spotify 음악 연동**: 게시글 작성 시 Spotify API로 실제 들은 음악 검색 및 선택
- **좋아요 / 북마크**: 공감 표현 및 참고하고 싶은 루틴 저장
- **무한 스크롤 피드**: 페이지네이션 기반 피드 탐색
- **AI 음악 추천 챗봇**: GPT 기반 챗봇이 사용자의 운동 상황에 맞는 음악을 대화형으로 추천

<!-- 여기에 핵심 기능별 캡처 이미지 (감정/운동 태그 선택, Spotify 연동, 좋아요/북마크) -->
<img width="897" height="800" alt="own_image4" src="https://github.com/user-attachments/assets/7fc1e58c-bd2f-4105-ad71-01d757dbfa49" />
<img width="801" height="726" alt="own_image3" src="https://github.com/user-attachments/assets/806004e1-f257-47e4-afbe-0358b108aff0" />
<img width="1337" height="835" alt="own_image5" src="https://github.com/user-attachments/assets/dc3f4d77-8ee8-4d0f-bfe7-db90e6b17764" />

---

## 기술 스택

| 구분 | 내용 |
|---|---|
| **Backend** | Spring Boot, MyBatis, 세션 기반 인증 |
| **Frontend** | Vue.js 3 (Composition API), Pinia |
| **External API** | Spotify API, GPT API (음악 추천 챗봇) |
| **DB** | MySQL |
| **Tool** | Git, Notion, Jira |

---

## 담당 역할 (2인 팀)

2인 팀이지만 막연히 "백엔드/프론트"로 나누지 않고, 프론트엔드도 **도메인 기준**으로 분담해 각자 화면부터 API까지 한 흐름을 책임지도록 구성했습니다.

### 맡은 도메인: 사용자 관리 & 마이페이지
로그인/회원가입 흐름과 사용자가 작성한 과거 기록을 보여주는 마이페이지를 전담했습니다.

**백엔드**
- Post, Music, Like, Bookmark 도메인의 DTO / DAO / Service / Controller 전체 구현

**프론트엔드**
- `user.js`, `like.js`, `bookmark.js` API 통신 모듈
- `SidebarLeft`, `MiniProfile` 등 레이아웃 컴포넌트
- `ProfileHeader`, `MyJournalList/Item` 등 마이페이지 컴포넌트
- 로그인/회원가입 페이지, 마이페이지(나의 운동일지·좋아요·북마크 탭) 페이지
- `auth.js` Pinia 스토어 (로그인 유저 정보, 토큰 관리)

> 팀원은 메인 피드, 게시글 작성 4단계, Spotify 연동 화면을 담당했습니다. `PostItem`처럼 양쪽에서 공유해 쓰는 컴포넌트는 데이터 형식(props)을 사전에 맞추는 방식으로 협업했습니다.

### 협업 프로세스
2인 프로젝트에서도 협업 기준을 명확히 하기 위해 다음 내용을 문서화해 팀원과 공유했습니다.

- API 응답 형식 규칙
- 에러 코드 규칙
- 날짜 형식 통일 (`yyyy-MM-dd`)
- DTO 네이밍 규칙
- 라우터 경로 이름, 게시글 객체 데이터 형식 등 도메인 간 인터페이스 사전 정의

---

## ERD

<!-- 여기에 ERD 캡처 이미지 -->
<img width="1458" height="1026" alt="OWN_DB_ERD" src="https://github.com/user-attachments/assets/beafa0b3-f870-4c42-9a79-043f6ccdcff5" />


## API 명세서

<!-- 여기에 API 명세서 캡처 이미지 -->
<img width="1431" height="726" alt="스크린샷 2026-06-17 오전 10 01 01" src="https://github.com/user-attachments/assets/7b4be64b-fa56-4427-955e-42d47e912cf9" />
<img width="1438" height="782" alt="스크린샷 2026-06-17 오전 10 00 37" src="https://github.com/user-attachments/assets/e5a1aefc-7e62-4f4f-b407-2eefe3c1ab88" />

---
<img width="1337" height="835" alt="own_image5" src="https://github.com/user-attachments/assets/d1299731-195a-4532-aaf8-33ec7cc77604" />

## 트러블슈팅

**좋아요·북마크한 게시물이 마이페이지에 표시되지 않는 문제**
마이페이지 조회 쿼리가 본인이 작성한 게시물만 가져오도록 되어 있어, 좋아요·북마크한 게시물이 누락되는 문제를 확인하고 별도 조회 로직을 추가해 해결했습니다.

**API 키 하드코딩 이슈**
AI 챗봇 기능 구현 시 GMS API 키를 프론트엔드 코드에 직접 작성한 채로 커밋했던 것을 발견했습니다. 이미 만료된 키였지만, 환경변수로 분리하지 않으면 Public 레포에 키가 그대로 노출된다는 것을 인지하고 코드에서 제거했습니다. 이후 외부 API 키는 처음부터 환경변수로 관리해야 한다는 점을 학습했습니다.
