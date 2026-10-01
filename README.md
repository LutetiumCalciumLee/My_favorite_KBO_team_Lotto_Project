<details>
<summary>ENG (English Version)</summary>

# KBO Favorite Team Weekly Jersey Number Lotto

A web application that generates weekly lotto numbers using player jersey numbers from your favorite KBO team.

The project combines a React interface, Python web scraping, Supabase storage, and automated deployment through GitHub Pages.

**[Open Website](https://lutetiumcalciumlee.github.io/My_favorite_KBO_team_Lotto_Project/)** 

## 1. Project Background

This project was built by applying the concepts studied and documented in the following repositories:

- [Studying_React](https://github.com/LutetiumCalciumLee/Studying_React)
- [Studying_Web_Scraping](https://github.com/LutetiumCalciumLee/Studying_Web_Scraping)

The goal was to connect frontend development and web scraping in one application, covering data collection, interactive rendering, database persistence, and automated deployment.

| Learning Repository | Concepts Applied | Implementation |
| --- | --- | --- |
| Studying_React | JSX, components, state, event handling, list rendering, useEffect, and API integration | Team selection, draw settings, weekly number cards, asynchronous loading, and redraw controls |
| Studying_Web_Scraping | Requests, BeautifulSoup, HTML parsing, and structured data storage | Collecting KBO roster dates, team names, player names, and jersey numbers, then storing the results in Supabase |

The frontend uses React with Vite. The scraper uses Requests and BeautifulSoup.

## 2. Main Features

- Select any of the 10 KBO teams.
- Display the selected team's colors and logo.
- Choose how many numbers come from the first-team roster: 0–6.
- Use the “All” mode to draw from both first-team and Futures League rosters.
- Optionally include retired jersey numbers in the first-team candidate pool.
- Generate six unique numbers between 1 and 45.
- Display player names when available and distinguish number sources by color.
- View weekly results for Sunday and Tuesday through Saturday.
- Redraw today's numbers and save the updated result.
- Restore saved personal results after refreshing the page.
- Lock previous dates and prevent Saturday redraws from 20:00 KST.
- Automate roster synchronization and website deployment with GitHub Actions.

## 3. Number Selection Rules

### First-Team Count Mode

Selecting a first-team count of `N` draws:

- `N` numbers from the first-team candidate pool.
- `6 − N` numbers from the remaining numbers between 1 and 45.

For example, selecting **3** draws three numbers from the first-team pool and three numbers outside that pool.

### All Mode

The application combines the selected team's first-team and Futures League jersey numbers, removes duplicates, and draws six numbers.

### Retired Jersey Numbers

When this option is enabled, the team's retired jersey numbers are added to the first-team candidate pool.

### Shared Rules

- Only numbers from 1 to 45 are eligible.
- Jersey numbers beginning with zero are excluded.
- Each result contains six unique numbers in ascending order.
- Monday is excluded from the weekly schedule.
- Future dates remain pending until their date arrives.
- Previous dates cannot be redrawn.
- Saturday's numbers cannot be redrawn from **20:00 KST**.

The roster reference date shown on the page may differ from today's date, depending on the latest available KBO registration data.

## 4. Technology Stack

| Area | Technologies | Purpose |
| --- | --- | --- |
| Frontend | React, JavaScript, CSS, Vite | Interactive settings and weekly result rendering |
| Web Scraping | Python, Requests, BeautifulSoup | Collecting official player registration data |
| Authentication | Supabase Auth | Identifying visitors through anonymous accounts |
| Database | Supabase PostgreSQL, Row Level Security | Storing roster snapshots and draw results |
| Hosting | GitHub Pages | Serving the built frontend |
| Automation | GitHub Actions | Building, deploying, and synchronizing data |

### Data Flow

```text
KBO Registration Pages
        ↓
Python Scraper / GitHub Actions
        ↓
Supabase PostgreSQL
        ↕
React Application / GitHub Pages
        ↕
Supabase Anonymous Authentication
```

GitHub Pages serves static frontend files. Python scraping runs through GitHub Actions, while Supabase handles authentication and database storage.

## 5. Database and Record Management

| Table | Stored Data | Access |
| --- | --- | --- |
| `roster_snapshots` | First-team and Futures League player maps, roster dates, and synchronization timestamps | Visitors can read; backend automation writes |
| `daily_results` | Shared fixed results for each date, team, mode, and retired-number setting | Visitors can read; backend automation writes |
| `user_draws` | Personal daily results and redraw updates | Authenticated users can access their own records |

### Personal Records

- An anonymous account is created when a visitor first selects a team.
- The browser retains the authentication session.
- Draw results are stored in Supabase.
- Returning with the same browser session restores the saved results.
- Clearing browser storage or using another device does not restore the previous anonymous account.
- IP addresses are not used as user identifiers.

### Access Control

Row Level Security checks record ownership and enforces the date and Saturday cutoff restrictions.

Shared results can cover up to:

**10 teams × 8 modes × 2 retired-number settings = 160 combinations per eligible date**

Previously saved shared results are not overwritten.

## 6. Project Structure

```text
.
├── src/
│   ├── App.jsx                  # Main interface
│   ├── main.jsx                 # React entry point
│   ├── cloudApi.js              # Draw generation and database operations
│   ├── supabase.js              # Supabase client
│   └── styles.css               # Application styles
├── logos/
│   └── team-logos.json          # Packaged KBO and team logo images
├── automation/
│   ├── lotto.py                 # Roster parsing and number generation
│   ├── sync_supabase.py         # Roster and shared-result synchronization
│   └── requirements.txt         # Python dependencies
├── supabase/
│   └── schema.sql               # Database tables and RLS policies
├── .github/workflows/
│   ├── deploy-pages.yml         # GitHub Pages deployment
│   └── sync-kbo.yml             # Scheduled roster synchronization
├── kbo_team_colors.json         # Team colors
├── kbo_permanant_numbers.json   # Retired jersey numbers
├── .env.example                # Environment variable example
├── index.html
├── package.json
├── package-lock.json
└── vite.config.js
```

## 7. Local Development

Use Node.js 22 or later. Python 3.12 or later is recommended for the scraping automation.

### Install and Run the Frontend

```powershell
npm ci
Copy-Item .env.example .env.local
```

Configure `.env.local`:

```dotenv
VITE_SUPABASE_URL=https://hkwhdeacrzaxbraabydd.supabase.co
VITE_SUPABASE_PUBLISHABLE_KEY=YOUR_SUPABASE_PUBLISHABLE_KEY
```

Start the development server:

```powershell
npm run dev
```

Create a production build:

```powershell
npm run build
```

The connected Supabase project must have the database schema installed and anonymous sign-ins enabled.

### Install Scraper Dependencies

```powershell
python -m pip install -r automation/requirements.txt
```

## 8. Deployment and Automation

### Supabase Setup

1. Open the Supabase project's SQL Editor.
2. Run `supabase/schema.sql`.
3. Enable **Anonymous Sign-Ins** in Authentication → Sign In / Providers.
4. Check the project's publishable key and backend secret key.

### GitHub Actions Secrets

Configure the following repository secrets:

| Secret | Purpose |
| --- | --- |
| `SUPABASE_PUBLISHABLE_KEY` | Public frontend configuration used during the Pages build |
| `SUPABASE_SECRET_KEY` | Privileged backend access used by the synchronization workflow |

The secret or service-role key must never appear in frontend code, `VITE_` variables, README files, or public repository files.

Local environment files, SQLite databases, `node_modules`, and `dist` are excluded from uploads.

### GitHub Pages

1. Commit the project files to the `main` branch.
2. Set Settings → Pages → Source to **GitHub Actions**.
3. Run **Sync KBO data** manually to populate the database.
4. Wait for **Deploy GitHub Pages** to succeed.
5. Verify team selection, redraws, and record restoration on the deployed website.

### Synchronization Schedule

- Scheduled every three hours, starting at **00:17 KST**.
- An additional run is scheduled for **Saturday at 20:00 KST**.
- The workflow synchronizes roster snapshots and generates fixed shared results for the previous eligible date.
- The Saturday run also generates that day's shared results after the cutoff.
- Existing shared results are preserved.

GitHub Actions schedules may be delayed. The database's Saturday redraw restriction applies independently of the workflow's actual execution time.

## 9. Data Sources and Notes

- [KBO First-Team Player Registration](https://www.koreabaseball.com/Player/RegisterAll.aspx)
- [KBO Futures League Player Registration](https://www.koreabaseball.com/Futures/Player/Register.aspx)

Changes to the source pages may require scraper updates.

KBO and team logos belong to their respective rights holders. This is an unofficial personal learning project. Generated numbers do not predict or guarantee lottery winnings.

</details>

<details>
<summary>KOR (한국어 버전)</summary>

# KBO 리그 최애팀 주간 등번호 로또

좋아하는 KBO 팀의 선수 등번호로 일주일의 로또 번호를 만들어 보는 웹 애플리케이션입니다.

React 화면, Python 웹 스크래핑, Supabase 데이터 저장, GitHub Pages 자동 배포를 연결한 프로젝트입니다.

**[웹사이트 열기](https://lutetiumcalciumlee.github.io/My_favorite_KBO_team_Lotto_Project/)** 

## 1. 프로젝트를 만든 배경

이 프로젝트는 아래 두 저장소에 정리한 학습 내용을 바탕으로 만들어 본 실습 프로젝트입니다.

- [Studying_React](https://github.com/LutetiumCalciumLee/Studying_React)
- [Studying_Web_Scraping](https://github.com/LutetiumCalciumLee/Studying_Web_Scraping)

프론트엔드 개발과 웹 스크래핑을 하나의 애플리케이션으로 연결해, 데이터 수집부터 화면 표시, 데이터베이스 저장, 자동 배포까지 경험하는 것을 목표로 했습니다.

| 학습 저장소 | 적용한 학습 내용 | 프로젝트 구현 |
| --- | --- | --- |
| Studying_React | JSX, 컴포넌트, 상태, 이벤트 처리, 목록 렌더링, useEffect, API 연동 | 팀 선택, 추첨 조건 설정, 요일별 번호 카드, 비동기 조회, 재추첨 기능 |
| Studying_Web_Scraping | Requests, BeautifulSoup, HTML 파싱, 구조화된 데이터 저장 | KBO 명단 기준일·팀명·선수명·등번호 수집과 Supabase 저장 |

화면은 React와 Vite로 구현했고, 수집기는 Requests와 BeautifulSoup를 사용했습니다.

## 2. 주요 기능

- KBO 10개 팀 중 원하는 팀 선택
- 선택한 팀의 색상과 로고 표시
- 1군 후보에서 뽑을 번호 수를 0~6개로 설정
- 1군·퓨처스 등록 등번호를 함께 사용하는 전체 모드
- 영구결번을 1군 후보에 포함할지 선택
- 1~45 중 중복 없는 번호 6개 생성
- 확인 가능한 선수명 표시와 번호 출처별 색상 구분
- 일요일과 화요일~토요일의 주간 결과 조회
- 오늘 번호 재추첨과 변경 결과 저장
- 페이지 새로고침 후 개인 기록 복원
- 지난 날짜 잠금과 토요일 한국시간 20:00 이후 재추첨 차단
- GitHub Actions를 통한 명단 동기화와 웹사이트 자동 배포

## 3. 번호 생성 규칙

### 1군 선수 수 모드

1군 선수 수를 `N`으로 선택하면 다음과 같이 뽑습니다.

- 1군 후보에서 `N`개
- 1~45 중 1군 후보에 없는 번호에서 `6 − N`개

예를 들어 **3명**을 선택하면 1군 후보에서 3개, 1군 후보에 없는 번호에서 3개를 뽑습니다.

### 전체 모드

선택한 팀의 1군과 퓨처스 등록 등번호를 합치고 중복을 제거한 뒤, 해당 후보에서 번호 6개를 뽑습니다.

### 영구결번 포함

이 옵션을 켜면 해당 팀의 영구결번을 1군 후보에 추가합니다.

### 공통 규칙

- 1~45 범위의 번호만 사용합니다.
- 0으로 시작하는 등번호는 제외합니다.
- 중복 없는 번호 6개를 오름차순으로 표시합니다.
- 월요일은 주간 표시와 번호 생성에서 제외합니다.
- 미래 날짜는 해당 날짜가 될 때까지 예정 상태로 표시합니다.
- 지난 날짜는 다시 뽑을 수 없습니다.
- 토요일 번호는 **한국시간 20:00부터** 다시 뽑을 수 없습니다.

화면의 KBO 기준일은 원본 등록 페이지에서 제공하는 날짜이므로, 오늘 날짜와 다를 수 있습니다.

## 4. 기술 구성

| 영역 | 기술 | 역할 |
| --- | --- | --- |
| 프론트엔드 | React, JavaScript, CSS, Vite | 조건 선택과 주간 결과 표시 |
| 웹 스크래핑 | Python, Requests, BeautifulSoup | 공식 선수 등록 데이터 수집 |
| 인증 | Supabase Auth | 익명 계정을 통한 방문자 구분 |
| 데이터베이스 | Supabase PostgreSQL, Row Level Security | 명단과 번호 기록 저장 |
| 호스팅 | GitHub Pages | 빌드된 프론트엔드 제공 |
| 자동화 | GitHub Actions | 빌드·배포·데이터 동기화 |

### 데이터 흐름

```text
KBO 선수 등록 페이지
        ↓
Python 수집기 / GitHub Actions
        ↓
Supabase PostgreSQL
        ↕
React 애플리케이션 / GitHub Pages
        ↕
Supabase 익명 인증
```

GitHub Pages는 정적 화면을 제공하고, Python 수집기는 GitHub Actions에서 실행합니다. 인증과 데이터베이스 저장은 Supabase가 담당합니다.

## 5. 데이터베이스와 기록 관리

| 테이블 | 저장 내용 | 접근 방식 |
| --- | --- | --- |
| `roster_snapshots` | 날짜·팀별 1군·퓨처스 선수 명단, 기준일, 동기화 시각 | 방문자는 조회, 서버 자동화는 저장 |
| `daily_results` | 날짜·팀·모드·영구결번 조건별 공통 고정 결과 | 방문자는 조회, 서버 자동화는 저장 |
| `user_draws` | 사용자별 오늘 번호와 재추첨 결과 | 인증된 사용자가 자신의 기록만 접근 |

### 개인 기록

- 방문자가 처음 팀을 선택하면 익명 계정을 만듭니다.
- 인증 세션은 브라우저에 보관합니다.
- 번호 기록 자체는 Supabase에 저장합니다.
- 같은 브라우저 세션으로 다시 접속하면 저장한 결과를 복원합니다.
- 브라우저 데이터를 지우거나 다른 기기를 사용하면 기존 익명 계정의 기록을 복원할 수 없습니다.
- IP 주소는 사용자 식별자로 사용하지 않습니다.

### 접근 제어

Row Level Security로 기록 소유자를 확인하고, 날짜와 토요일 마감 시간에 따른 변경 제한을 적용합니다.

공통 결과는 추첨 대상 날짜마다 최대 다음 조합을 저장합니다.

**10개 팀 × 8개 모드 × 영구결번 2개 조건 = 160개 조합**

이미 저장된 공통 결과는 덮어쓰지 않습니다.

## 6. 프로젝트 구조

```text
.
├── src/
│   ├── App.jsx                  # 메인 화면
│   ├── main.jsx                 # React 진입점
│   ├── cloudApi.js              # 번호 생성과 DB 조회·저장
│   ├── supabase.js              # Supabase 클라이언트
│   └── styles.css               # 화면 스타일
├── logos/
│   └── team-logos.json          # KBO·팀 로고 이미지 데이터
├── automation/
│   ├── lotto.py                 # 명단 파싱과 번호 생성
│   ├── sync_supabase.py         # 명단·공통 결과 동기화
│   └── requirements.txt         # Python 의존성
├── supabase/
│   └── schema.sql               # DB 테이블과 RLS 정책
├── .github/workflows/
│   ├── deploy-pages.yml         # GitHub Pages 배포
│   └── sync-kbo.yml             # 예약 명단 동기화
├── kbo_team_colors.json         # 팀 색상
├── kbo_permanant_numbers.json   # 영구결번
├── .env.example                # 환경 변수 예시
├── index.html
├── package.json
├── package-lock.json
└── vite.config.js
```

## 7. 로컬 실행

Node.js 22 이상을 사용합니다. 수집 자동화에는 Python 3.12 이상을 권장합니다.

### 프론트엔드 설치와 실행

```powershell
npm ci
Copy-Item .env.example .env.local
```

`.env.local`에 다음 값을 설정합니다.

```dotenv
VITE_SUPABASE_URL=https://hkwhdeacrzaxbraabydd.supabase.co
VITE_SUPABASE_PUBLISHABLE_KEY=본인의_SUPABASE_PUBLISHABLE_KEY
```

개발 서버를 실행합니다.

```powershell
npm run dev
```

배포용 화면을 빌드합니다.

```powershell
npm run build
```

연결할 Supabase 프로젝트에는 DB 스키마가 적용되어 있어야 하며, 익명 로그인이 활성화되어 있어야 합니다.

### 수집기 의존성 설치

```powershell
python -m pip install -r automation/requirements.txt
```

## 8. 배포와 자동화

### Supabase 설정

1. Supabase 프로젝트의 SQL Editor를 엽니다.
2. `supabase/schema.sql`을 실행합니다.
3. Authentication → Sign In / Providers에서 **Anonymous Sign-Ins**를 활성화합니다.
4. 프로젝트의 publishable key와 서버용 secret key를 확인합니다.

### GitHub Actions Secrets

저장소에 다음 Secrets를 등록합니다.

| Secret | 용도 |
| --- | --- |
| `SUPABASE_PUBLISHABLE_KEY` | Pages 빌드에 전달할 브라우저용 공개 설정 |
| `SUPABASE_SECRET_KEY` | 명단 동기화 작업에서 사용할 서버용 DB 접근 키 |

secret 또는 service_role 키는 프론트엔드 코드, `VITE_` 환경 변수, README, 공개 저장소 파일에 넣지 않습니다.

로컬 환경 변수 파일, SQLite DB, `node_modules`, `dist`는 업로드에서 제외합니다.

### GitHub Pages 설정

1. 프로젝트 파일을 `main` 브랜치에 올립니다.
2. Settings → Pages → Source를 **GitHub Actions**로 설정합니다.
3. **Sync KBO data**를 수동 실행해 데이터베이스를 채웁니다.
4. **Deploy GitHub Pages** 작업이 성공할 때까지 기다립니다.
5. 배포된 웹사이트에서 팀 선택, 재추첨, 저장 기록 복원을 확인합니다.

### 동기화 일정

- 한국시간 **00:17부터 3시간마다** 예약 실행합니다.
- **토요일 20:00**에 추가 작업을 예약합니다.
- 명단을 동기화하고, 전날이 추첨 대상 날짜인 경우 공통 고정 결과를 생성합니다.
- 토요일에는 마감 시간 이후 당일 공통 결과도 생성합니다.
- 기존 공통 결과는 유지합니다.

GitHub Actions 예약 실행은 지연될 수 있습니다. 데이터베이스의 토요일 재추첨 제한은 실제 작업 실행 시각과 관계없이 적용됩니다.

## 9. 데이터 출처와 안내

- [KBO 1군 선수 등록 현황](https://www.koreabaseball.com/Player/RegisterAll.aspx)
- [KBO 퓨처스 선수 등록 현황](https://www.koreabaseball.com/Futures/Player/Register.aspx)

원본 페이지 구조가 변경되면 수집기를 수정해야 할 수 있습니다.

KBO와 구단 로고의 권리는 각 권리자에게 있습니다. 이 프로젝트는 개인 학습을 위한 비공식 프로젝트이며, 생성한 번호는 로또 당첨을 예측하거나 보장하지 않습니다.

</details>
