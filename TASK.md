# Task: SPOEX 2026 실시간 리더보드 웹앱

## 목적
SPOEX 부스에서 TV/모니터에 풀스크린 상시 노출. 묠니르 + Power Score 실시간 랭킹.

## 핵심 요구

### 레이아웃 (1920×1080 최적화, 풀스크린)
- 좌우 50:50 분할
- 좌측: 🔨 묠니르 챌린지 Top 10
- 우측: ⚡ Power Score Top 10
- 하단: 참가 인원, 역대 최고, Power Score 한 줄 설명
- 스크롤 없음!

### 데이터 소스
Supabase (PostgREST + Realtime)
```
Base URL: https://ssidizurrvnfmqbvfsqr.supabase.co
Anon Key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InNzaWRpenVycnZuZm1xYnZmc3FyIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzE4NDg1OTgsImV4cCI6MjA4NzQyNDU5OH0.3UMwz8QPAOoN_hFu-EaheLQunHdpJk_WvMjxuiONRSI
```

### 테이블 구조
1. `spoex_results` — 결과 저장
   - id (uuid), mode ('mjolnir'|'vbt'), exercise_name, estimated_1rm, avg_velocity, score (Power Score), mjolnir_score, max_height_mm, created_at, device_id
2. `spoex_registrations` — QR 사후 등록
   - id, result_id (FK → spoex_results), nickname, gender, age_group, marketing_consent, created_at

### 리더보드 쿼리
**묠니르**: 
```
SELECT r.nickname, res.mjolnir_score, res.max_height_mm, res.created_at
FROM spoex_registrations r
JOIN spoex_results res ON r.result_id = res.id
WHERE res.mode = 'mjolnir' AND res.created_at >= today_start
ORDER BY res.mjolnir_score DESC
LIMIT 10
```

**Power Score (VBT)**:
```
SELECT r.nickname, res.score, res.exercise_name, res.estimated_1rm_kg, res.avg_velocity, res.created_at
FROM spoex_registrations r 
JOIN spoex_results res ON r.result_id = res.id
WHERE res.mode = 'vbt' AND res.created_at >= today_start
ORDER BY res.score DESC
LIMIT 10
```

⚠️ PostgREST FK 힌트: `spoex_results!spoex_registrations_result_id_fkey(...)` 필요

### 실시간 갱신
- Supabase Realtime: `spoex_registrations` INSERT 구독
- 새 등록 → 리더보드 재조회
- 갱신 연출: 새 항목 하이라이트 펄스 (2~3초 노란 배경 fade)
- Top 3 진입: 특별 강조

### 디자인
- 다크 테마 (TV 노출)
- 브랜드: Primary #5252FA, Accent #BAFC27, Bg #0A0A0F
- 폰트: Pretendard (CDN)
- 묠니르 쪽: 점수(큰 숫자) + 최대높이(cm 서브텍스트)
- Power Score 쪽: 등급 이모지 + 점수 + 운동명
- Power Score 등급: 🔥 100+ EXPLOSIVE, ⚡ 70-99 FAST, 💪 40-69 STRONG, 🐢 ~39 GRINDER
- Top 1/2/3: 🥇🥈🥉 메달
- 미등록(닉네임 없는 결과)은 표시 안 함
- 상단 로고: Roomfit (이미지 URL: https://wspn-images.wespion.workers.dev/roomfit-logo.png)
  - 다크 배경이므로 CSS filter:brightness(0) invert(1) 적용

### 기타
- 순수 HTML/CSS/JS, 프레임워크 없음
- Supabase JS 라이브러리 CDN 사용 (@supabase/supabase-js)
- 30초마다 자동 리프레시 fallback (Realtime 실패 대비)
- 풀스크린 진입 버튼 (F11 불편 방지)
- 현재 시각 표시

## 파일
```
/
├── index.html   ← 모든 것 인라인
└── TASK.md      ← 이 파일
```
