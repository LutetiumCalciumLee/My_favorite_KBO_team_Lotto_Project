# ⚾ KBO 리그 최애팀 주간 등번호 로또

좋아하는 KBO 팀의 선수 등번호로 일주일의 로또 번호를 만들어 보는 웹 프로젝트입니다. React로 화면을 만들고 Python으로 선수 등록 명단을 수집하며, Supabase에 명단과 번호 기록을 저장합니다.

**[웹사이트 열기](https://lutetiumcalciumlee.github.io/My_favorite_KBO_team_Lotto_Project/)** · [GitHub 저장소](https://github.com/LutetiumCalciumLee/My_favorite_KBO_team_Lotto_Project)

## 프로젝트를 만든 배경

이 프로젝트는 **[Studying_React](https://github.com/LutetiumCalciumLee/Studying_React)**와 **[Studying_Web_Scraping](https://github.com/LutetiumCalciumLee/Studying_Web_Scraping)**에 정리한 학습 내용을 바탕으로 만들어 본 실습 프로젝트입니다. React와 웹 스크래핑을 하나의 서비스로 연결해 데이터 수집, 화면 표시, 데이터베이스 저장, 자동 배포까지 경험하는 것을 목표로 했습니다.

| 학습 저장소 | 학습한 내용 | 프로젝트에 적용한 부분 |
| --- | --- | --- |
| Studying_React | JSX, 컴포넌트, 상태와 이벤트, 목록 렌더링, useEffect, 폼 입력, API 연동 | 팀·조건 선택, 요일별 번호 카드, 비동기 조회, 새로고침·재추첨 화면 |
| Studying_Web_Scraping | Requests, BeautifulSoup, HTML 데이터 추출, 구조화된 데이터 저장 | KBO 등록 페이지에서 날짜·팀·선수명·등번호 수집, JSON 변환과 DB 저장 |

학습 개념을 응용해 React + Vite와 Requests + BeautifulSoup로 구현했습니다. 학습 자료의 Selenium 예제를 그대로 사용하는 구성은 아닙니다.

## 주요 기능

- KBO 10개 팀 선택과 팀별 색상·로고 표시
- 1군 선수 번호 0~6개 선택 또는 1군·퓨처스 전체 후보에서 추첨
- 영구결번을 1군 후보에 포함할지 선택
- 1~45 중 중복 없는 번호 6개 생성, 선수명과 번호 출처 표시
- 일요일과 화요일~토요일의 주간 기록 제공; 월요일 제외
- 오늘 번호 재추첨과 Supabase 저장, 새로고침 후 개인 기록 복원
- 지난 날짜 변경 차단, 토요일 한국시간 20:00부터 당일 재추첨 차단
- GitHub Actions로 명단 동기화와 GitHub Pages 배포 자동화

1군 3명 모드를 선택하면 1군 후보에서 3개, 1군 후보에 없는 번호에서 3개를 뽑습니다. 전체 모드는 1군과 퓨처스 등록 등번호를 합쳐 6개를 뽑습니다. 영구결번 포함을 선택하면 해당 번호도 1군 후보로 취급합니다. 0으로 시작하는 등번호와 1~45 범위 밖 등번호는 제외합니다.

## 기술 구성

| 영역 | 기술 | 역할 |
| --- | --- | --- |
| 화면 | React, JavaScript, CSS, Vite | 조건 선택, 주간 표시, 재추첨 |
| 수집 | Python, Requests, BeautifulSoup | 선수 등록 명단 수집 |
| 인증·DB | Supabase Auth, PostgreSQL, RLS | 익명 사용자 구분, 사용자별 기록 저장 |
| 배포·자동화 | GitHub Pages, GitHub Actions | 정적 웹사이트 배포, 예약 동기화 |

```mermaid
flowchart LR
    KBO[KBO 등록 페이지] --> PY[GitHub Actions · Python 수집기]
    PY --> DB[(Supabase PostgreSQL)]
    WEB[GitHub Pages · React] --> AUTH[Supabase 익명 인증]
    WEB --> DB
    DB --> WEB
```

GitHub Pages는 빌드한 정적 파일을 제공하고, Supabase는 인증과 DB 저장을 담당합니다. Python 수집기는 GitHub Actions에서 실행합니다.

## 데이터 저장

| 테이블 | 저장 내용 | 접근 방식 |
| --- | --- | --- |
| `roster_snapshots` | 날짜·팀별 1군·퓨처스 명단, 갱신 시각 | 방문자는 조회, 자동화는 저장 |
| `daily_results` | 날짜·팀·조건별 공통 고정 번호와 선수명 | 방문자는 조회, 자동화는 저장 |
| `user_draws` | 사용자별 오늘 번호 및 재추첨 기록 | 인증된 사용자가 자신의 기록만 조회·저장 |

첫 방문 때 익명 계정을 만들고 브라우저에 세션을 보관합니다. 번호 자체는 Supabase에 저장합니다. 같은 브라우저의 세션으로 다시 조회할 수 있지만, 브라우저 데이터를 지우거나 다른 기기를 사용하면 기존 익명 계정 기록에 접근할 수 없습니다. IP 주소는 사용자 식별자로 사용하지 않습니다.

RLS로 개인 기록의 소유자와 날짜·토요일 20:00 제한을 검사합니다. 공통 결과는 최대 **10팀 × 8모드 × 영구결번 2조건 = 160조합**을 저장하며 기존 결과를 덮어쓰지 않습니다.

## 프로젝트 구조

```text
src/                          React 화면과 Supabase 클라이언트
logos/                        KBO·팀 로고
automation/lotto.py            명단 파싱과 번호 생성
automation/sync_supabase.py    명단·공통 결과 DB 동기화
automation/requirements.txt    Python 의존성
supabase/schema.sql           테이블과 RLS 정책
.github/workflows/            Pages 배포, KBO 동기화
kbo_team_colors.json          팀 색상
kbo_permanant_numbers.json     영구결번
.env.example                  공개 환경 변수 예시
vite.config.js                프로젝트 Pages 경로
```

## 로컬 실행

Node.js 22 이상과 Python 3.12 이상을 사용합니다.

```powershell
npm ci
Copy-Item .env.example .env.local
# .env.local에 프로젝트의 publishable key 입력
npm run dev
```

```dotenv
VITE_SUPABASE_URL=https://hkwhdeacrzaxbraabydd.supabase.co
VITE_SUPABASE_PUBLISHABLE_KEY=본인의_publishable_key
```

화면 빌드는 `npm run build`로 실행합니다. 개발 화면에서도 Supabase 테이블과 익명 인증 설정이 필요합니다.

## GitHub Pages와 Supabase 배포

1. [Supabase 프로젝트](https://supabase.com/dashboard/project/hkwhdeacrzaxbraabydd)의 SQL Editor에서 [`supabase/schema.sql`](supabase/schema.sql)을 실행합니다.
2. Authentication → Sign In / Providers에서 Anonymous Sign-Ins를 활성화합니다.
3. 저장소의 Actions Secrets에 아래 두 값을 등록합니다.
4. 배포 파일을 `main` 브랜치에 업로드합니다.
5. [Settings → Pages](https://github.com/LutetiumCalciumLee/My_favorite_KBO_team_Lotto_Project/settings/pages)의 Source를 **GitHub Actions**로 설정합니다.
6. Actions에서 **Sync KBO data**를 수동 실행해 명단을 채웁니다.
7. **Deploy GitHub Pages** 성공 후 팀 선택, 재추첨, 새로고침 후 기록 복원을 확인합니다.

| GitHub Secret | 용도 |
| --- | --- |
| `SUPABASE_PUBLISHABLE_KEY` | 브라우저용 공개 키; Pages 빌드에 전달 |
| `SUPABASE_SECRET_KEY` | 자동화 전용 secret 또는 service_role 키; DB 동기화에 사용 |

**secret/service_role 키는 React 코드, `VITE_` 변수, README 또는 공개 저장소에 넣지 않습니다.** `.env.local`, SQLite DB, `node_modules`, `dist`는 업로드에서 제외합니다.

명단 동기화는 한국시간 00:17부터 3시간마다 예약되며, 토요일에는 20:00 작업도 예약합니다. GitHub Actions 예약 작업은 지연될 수 있어 정확한 실행 시각을 보장하지 않습니다. DB의 토요일 제한은 작업 실행 시각과 관계없이 적용됩니다. KBO 페이지 구조가 바뀌면 수집기를 점검해야 합니다.

## 데이터 출처

- [KBO 1군 선수 등록 현황](https://www.koreabaseball.com/Player/RegisterAll.aspx)
- [KBO 퓨처스 선수 등록 현황](https://www.koreabaseball.com/Futures/Player/Register.aspx)

KBO와 구단 로고의 권리는 각 권리자에게 있습니다. 개인 학습과 실습을 위한 비공식 프로젝트이며, 생성 번호는 당첨을 예측하거나 보장하지 않습니다.
