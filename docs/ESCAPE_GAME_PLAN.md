# 웹 방탈출 게임 기획서

> **프로젝트명**: COMPLUS+ Escape
> **플랫폼**: wrkholic84.github.io (Jekyll/GitHub Pages)
> **장르**: 웹 기반 방탈출 + ARG (대체 현실 게임)
> **대상**: 보안/IT에 관심 있는 방문자

---

## 1. ARG(Alternate Reality Game)란?

### 정의

ARG(대체 현실 게임)는 **현실과 가상의 경계를 흐리는 인터랙티브 내러티브 게임**입니다. 플레이어는 "이것이 게임이다"라는 명시적 안내 없이, 마치 현실에서 벌어지는 일처럼 느끼며 숨겨진 단서를 추적합니다.

### ARG의 핵심 요소

| 요소 | 설명 | 본 게임 적용 |
|------|------|-------------|
| **TINAG** (This Is Not A Game) | 게임임을 드러내지 않음. 현실처럼 보이는 단서 제공 | 블로그 포스트 속에 숨겨진 암호, 소스 코드의 주석에 힌트 |
| **Rabbit Hole** | 플레이어가 게임 세계로 빠져드는 최초 진입점 | 블로그의 특정 포스트나 404 페이지에서 시작 |
| **Trailhead** | 다음 단서로 이어지는 경로 | HTML 소스 코드, HTTP 헤더, 이미지 메타데이터 등 |
| **Puppet Master** | 게임을 설계하고 운영하는 주체 (보통 숨겨짐) | 사이트 운영자 (wrkholic84) |
| **커뮤니티 협업** | 플레이어들이 단서를 공유하며 협력 | 공유 가능한 힌트 시스템 |
| **크로스 미디어** | 여러 매체를 넘나드는 단서 | 웹페이지, GitHub 저장소, 이미지 스테가노그래피 등 |

### 유명 ARG 사례

- **I Love Bees** (2004): Halo 2 프로모션. 꿀벌 양봉 사이트가 해킹당한 것처럼 연출
- **Cicada 3301** (2012~): 인터넷 역사상 가장 유명한 ARG. 암호학, 스테가노그래피 활용
- **Year Zero** (2007): Nine Inch Nails 앨범 프로모션. USB 드라이브에 숨긴 음원
- **Trial of the Serpent** (2024): 웹사이트, SNS, 실제 장소를 넘나드는 퍼즐

---

## 2. 게임 컨셉

### 배경 스토리

```
당신은 보안 연구원 "wrkholic84"의 블로그에 접속했다.
평범한 기술 블로그처럼 보이지만, 어딘가 이상하다.
최근 포스트의 마지막 줄에 이상한 문자열이 보인다...

"The system remembers what you forgot. Start where errors live."

404 페이지로 이동하면, 게임이 시작된다.
```

### 테마

**"Operation: Memory Leak"** — 시스템 어딘가에 숨겨진 기밀 데이터를 찾아 탈출하라.

전체적으로 **해킹/보안** 세계관을 기반으로 하며, 실제 보안 기술(인코딩, 암호화, 네트워크 개념)을 퍼즐에 녹여냅니다.

---

## 3. 게임 구조

### 전체 흐름

```
[Rabbit Hole: 블로그 포스트 속 숨겨진 단서]
         │
         ▼
[Stage 0: The Entrance] ─── 404 페이지에서 시작
         │
         ▼
[Stage 1: The Terminal] ─── 터미널 시뮬레이션 퍼즐
         │
         ▼
[Stage 2: The Archive] ─── 암호 해독 + 스테가노그래피
         │
         ▼
[Stage 3: The Network] ─── 네트워크/패킷 분석 퍼즐
         │
         ▼
[Stage 4: The Vault] ─── 최종 종합 퍼즐
         │
         ▼
[Ending: The Escape] ─── 탈출 성공 + 결과 공유
```

---

## 4. 스테이지 상세 설계

### Rabbit Hole (진입점) — ARG 요소

> 플레이어가 게임의 존재를 발견하는 순간

**구현 방식:**
- 블로그의 기존 포스트 하단에 눈에 잘 띄지 않는 문자열 삽입
- HTML 소스코드의 주석에 힌트 (`<!-- Find the door where pages don't exist -->`)
- `robots.txt`에 수상한 Disallow 경로 추가 (`Disallow: /escape/`)
- 사이트 파비콘이나 프로필 이미지에 스테가노그래피로 URL 숨기기

**예시:**
```html
<!-- blog post 하단, 아주 작은 글씨 또는 배경색과 같은 색상 -->
<span style="color: var(--bg-color); font-size: 1px;">
  aHR0cHM6Ly93cmtob2xpYzg0LmdpdGh1Yi5pby9lc2NhcGUv
</span>
```
_(Base64 디코딩 → 게임 URL)_

---

### Stage 0: The Entrance

> **테마**: "깨진 시스템에 접속하다"
> **URL**: `/escape/` (커스텀 404 스타일 페이지)
> **난이도**: ★☆☆☆☆

**화면 구성:**
- CRT 모니터 느낌의 레트로 터미널 UI
- 글리치 효과와 함께 텍스트가 타이핑되듯 나타남
- 화면 깜빡임, 스캔라인 효과

**퍼즐:**
```
> SYSTEM ALERT: Unauthorized access detected.
> Enter security clearance code to proceed.
>
> HINT: "I am not a page, but I tell you when pages are lost.
>        My number is the response when nothing is found."
>
> CODE: [____]
```

**정답:** `404`

**ARG 요소:**
- 페이지 소스코드에 다음 스테이지의 위치를 암시하는 주석이 숨겨져 있음
- HTTP 응답 헤더에 `X-Hint: look-deeper` 같은 커스텀 헤더 삽입 (GitHub Pages 제한으로 대체 방법 사용 가능)

---

### Stage 1: The Terminal

> **테마**: "시스템 터미널을 해킹하라"
> **URL**: `/escape/terminal/`
> **난이도**: ★★☆☆☆

**화면 구성:**
- 풀스크린 터미널 에뮬레이터 (검은 배경, 녹색 텍스트)
- 실제 명령어를 입력할 수 있는 인터랙티브 프롬프트

**퍼즐:**
```
root@escape:~$ help

Available commands: ls, cat, cd, echo, decode, history, whoami

Objective: Find the encrypted key file and decode it.
```

**풀이 과정:**
1. `ls` → 파일 목록 표시 (`readme.txt`, `.hidden/`, `logs/`)
2. `cd .hidden` → 숨겨진 디렉토리 진입
3. `ls` → `key.enc` 파일 발견
4. `cat key.enc` → `Vm0wd2QyUXlVWGw...` (Base64 인코딩된 텍스트)
5. `decode key.enc` → ROT13 + Base64 다중 인코딩 해독
6. 최종 키 입력 → 다음 스테이지 잠금 해제

**ARG 요소:**
- `history` 명령어 입력 시 이전 사용자의 명령어 기록이 나타남 (스토리 요소)
- `cat logs/access.log` → 의문의 IP 주소와 타임스탬프 (이후 스테이지 복선)

---

### Stage 2: The Archive

> **테마**: "삭제된 기밀 문서를 복원하라"
> **URL**: `/escape/archive/`
> **난이도**: ★★★☆☆

**화면 구성:**
- 파일 탐색기 스타일 UI
- 손상된 문서, 깨진 이미지, 암호화된 파일들

**퍼즐 구성 (3개의 서브 퍼즐):**

**2-A: 이미지 스테가노그래피**
- 평범해 보이는 이미지 파일 제공
- 이미지의 RGB 채널이나 LSB(최하위 비트)에 메시지 숨김
- 웹에서 직접 분석 도구 제공 (채널 분리, 밝기 조절 슬라이더)
- 숨겨진 텍스트: 다음 퍼즐의 비밀번호

**2-B: 시저 암호 + 빈도 분석**
- 암호화된 문서 텍스트 제공
- 문자 빈도 분석 도구 제공 (히스토그램 차트)
- 시저 암호 시프트 값을 찾아 복호화

**2-C: 조각난 QR 코드**
- 4조각으로 분할된 QR 코드
- 드래그 앤 드롭으로 올바른 순서로 조합
- 완성된 QR 코드 스캔 → 다음 스테이지 URL

**ARG 요소:**
- GitHub 저장소의 특정 커밋 메시지에 힌트 숨기기
- 이미지 EXIF 데이터에 GPS 좌표 → 좌표를 검색하면 의미 있는 장소 (e.g., 유명 해킹 컨퍼런스 장소)

---

### Stage 3: The Network

> **테마**: "네트워크 트래픽에서 진실을 찾아라"
> **URL**: `/escape/network/`
> **난이도**: ★★★★☆

**화면 구성:**
- 네트워크 모니터링 대시보드 UI
- 실시간으로 흘러가는 가짜 패킷 데이터
- 와이어샤크 스타일의 패킷 분석 뷰

**퍼즐:**

**3-A: 패킷 필터링**
- 수천 개의 패킷 데이터 중 수상한 패킷 필터링
- 특정 IP 주소 패턴이나 포트 번호로 필터링하면 숨겨진 메시지 발견
- Stage 1의 access.log에서 본 IP가 여기서 다시 등장 (복선 회수)

**3-B: DNS 퍼즐**
- 의문의 DNS 쿼리 목록 제공
- 서브도메인 이름을 이어붙이면 메시지 완성
  ```
  this.is.the.key.you.are.looking.for.example.com
  → "this is the key you are looking for"
  ```

**3-C: 암호화 통신 해독**
- 두 사용자 간의 암호화된 채팅 로그
- 공개된 키 교환 정보를 이용해 간단한 XOR 암호 해독
- 복호화된 대화에서 최종 Vault의 비밀번호 획득

**ARG 요소:**
- 실제 nslookup/dig 명령어로 확인 가능한 TXT 레코드 힌트 (도메인 소유 시)
- 패킷 데이터의 타임스탬프가 실제 날짜와 연동

---

### Stage 4: The Vault (최종 스테이지)

> **테마**: "금고를 열고 탈출하라"
> **URL**: `/escape/vault/`
> **난이도**: ★★★★★

**화면 구성:**
- 거대한 금고 문 3D 렌더링 (CSS 3D transforms)
- 4개의 잠금장치 (이전 스테이지에서 수집한 단서 사용)
- 타이머 (긴장감 연출, 실패해도 재시도 가능)

**퍼즐:**
- 이전 4개 스테이지에서 획득한 키/코드 조합
- 각 키를 올바른 잠금장치에 매칭
- 최종 마스터 코드: 4개 키를 특정 알고리즘으로 조합 (e.g., 각 키의 첫 글자를 SHA-256 해시)
- 해시의 앞 8자리를 입력하면 금고 오픈

**ARG 요소:**
- 금고가 열리면 "기밀 문서" 공개 — 실제로는 다음 블로그 포스트의 비공개 미리보기 또는 이스터에그
- 특정 시간대에만 열리는 보너스 잠금장치 (시간 기반 ARG)

---

### Ending: The Escape

> **탈출 성공 화면**

**구성:**
- 축하 애니메이션 (매트릭스 스타일 코드 레인)
- 클리어 시간, 힌트 사용 횟수 표시
- 고유한 "탈출 인증서" 이미지 생성 (이름 + 클리어 시간 + 날짜)
- SNS 공유 버튼 (Twitter/X, 링크 복사)
- 리더보드 (GitHub Issues API 또는 별도 백엔드 활용)

---

## 5. ARG 레이어 통합 전략

게임 바깥에서도 단서를 숨겨 플레이어의 탐구심을 자극합니다.

### 5-1. 블로그 포스트 내 숨겨진 단서
```
기존 블로그 글 → 특정 단어의 첫 글자만 모으면 메시지
HTML 주석 → <!-- stage2-hint: shift=13 -->
CSS에서 보이지 않는 텍스트 → color: transparent 또는 font-size: 0
```

### 5-2. GitHub 저장소 활용
```
특정 커밋 메시지에 단서 삽입
Issue나 PR 제목에 암호화된 힌트
README의 뱃지 이미지에 스테가노그래피
.gitignore에 의미심장한 주석
```

### 5-3. 메타데이터 활용
```
이미지 EXIF 데이터에 GPS 좌표/설명 삽입
파비콘 변경 (특정 스테이지 진입 시)
og:description 메타 태그에 숨긴 메시지
```

### 5-4. 시간 기반 이벤트
```
특정 날짜/시간에만 나타나는 단서
매 정시마다 바뀌는 힌트 텍스트
주말에만 열리는 보너스 스테이지
```

---

## 6. 기술 구현 계획

### 기술 스택

| 구분 | 기술 | 이유 |
|------|------|------|
| **프레임워크** | Vanilla JS + Jekyll | 기존 사이트와의 통합, 별도 빌드 불필요 |
| **UI/애니메이션** | CSS3 + Canvas API | CRT 효과, 글리치, 터미널 애니메이션 |
| **터미널 에뮬레이션** | 커스텀 JS 터미널 | 명령어 파싱 + 응답 시스템 |
| **암호화/해시** | Web Crypto API | 클라이언트 사이드 암호화 검증 |
| **이미지 처리** | Canvas API | 스테가노그래피 분석 도구 |
| **상태 관리** | LocalStorage | 진행 상황 저장 (서버리스) |
| **공유/리더보드** | GitHub Issues API | 서버 없이 데이터 저장 |

### 디렉토리 구조

```
/escape/
├── index.html              # Stage 0: The Entrance
├── terminal/
│   └── index.html          # Stage 1: The Terminal
├── archive/
│   └── index.html          # Stage 2: The Archive
├── network/
│   └── index.html          # Stage 3: The Network
├── vault/
│   └── index.html          # Stage 4: The Vault
├── escaped/
│   └── index.html          # Ending: 탈출 성공
├── assets/
│   ├── css/
│   │   ├── escape-common.css    # 공통 스타일
│   │   ├── terminal.css         # 터미널 효과
│   │   ├── crt.css              # CRT 모니터 효과
│   │   └── glitch.css           # 글리치 효과
│   ├── js/
│   │   ├── game-engine.js       # 게임 코어 로직
│   │   ├── terminal-emulator.js # 터미널 에뮬레이터
│   │   ├── crypto-utils.js      # 암호화 유틸리티
│   │   ├── stego-analyzer.js    # 스테가노그래피 도구
│   │   ├── progress.js          # 진행 상황 관리
│   │   └── effects.js           # 시각 효과
│   └── images/
│       ├── stego-challenge.png  # 스테가노그래피 퍼즐 이미지
│       ├── qr-pieces/           # QR 코드 조각
│       └── certificate-bg.png   # 인증서 배경
└── _data/
    └── puzzles.json             # 퍼즐 정답 (해시값으로 저장)
```

### 보안 고려사항

```
1. 정답은 절대 평문으로 저장하지 않음 → SHA-256 해시로 비교
2. 스테이지 URL은 추측하기 어렵게 설정하되, 순서대로만 접근 가능
3. LocalStorage의 진행 데이터는 무결성 검증 (HMAC)
4. 소스코드를 봐도 쉽게 풀 수 없도록 퍼즐 로직 난독화
   (단, 완전한 보안이 목적이 아니라 재미가 목적이므로 적절한 수준)
```

---

## 7. 힌트 시스템

각 스테이지에서 막힌 플레이어를 위한 단계적 힌트 시스템:

```
[힌트 1] 방향 제시 (무료)
    "이 방에서 보이지 않는 것을 찾아보세요"

[힌트 2] 구체적 가이드 (시간 패널티 +30초)
    "페이지 소스코드를 확인해보세요"

[힌트 3] 거의 정답 (시간 패널티 +60초)
    "주석에 Base64로 인코딩된 문자열이 있습니다"
```

- 힌트 사용 횟수는 최종 점수에 반영
- 최종 인증서에 "NO HINTS" 뱃지 표시 (힌트 미사용 시)

---

## 8. 개발 단계 (우선순위)

### Phase 1: 코어 시스템 (MVP)
- [ ] 게임 엔진 (상태 관리, 스테이지 전환, 진행 저장)
- [ ] CRT/터미널 UI 공통 컴포넌트
- [ ] Stage 0 (The Entrance) 구현
- [ ] Stage 1 (The Terminal) 구현

### Phase 2: 고급 퍼즐
- [ ] Stage 2 (The Archive) — 스테가노그래피, 암호 해독
- [ ] Stage 3 (The Network) — 패킷 분석 UI
- [ ] Stage 4 (The Vault) — 최종 퍼즐

### Phase 3: ARG 레이어
- [ ] 블로그 포스트에 Rabbit Hole 삽입
- [ ] GitHub 저장소 단서 배치
- [ ] 메타데이터 단서 삽입

### Phase 4: 마무리
- [ ] Ending 화면 + 인증서 생성
- [ ] 힌트 시스템
- [ ] SNS 공유 기능
- [ ] 리더보드 (선택)
- [ ] 테스트 및 밸런싱

---

## 9. 확장 가능성

- **시즌제**: 매 분기 새로운 에피소드 추가
- **협동 퍼즐**: 2인 이상이 동시에 접속해야 풀리는 퍼즐
- **실시간 이벤트**: 특정 날짜에만 열리는 한정 스테이지
- **난이도 선택**: Easy / Normal / Hard 모드
- **다국어 지원**: 한국어/영어 전환

---

*이 기획서는 wrkholic84.github.io의 웹 방탈출 게임 "Operation: Memory Leak"의 초기 기획 문서입니다.*
