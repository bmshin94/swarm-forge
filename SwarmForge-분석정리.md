# SwarmForge 분석 정리 📚

> 이 문서는 SwarmForge 저장소를 직접 뜯어보며 분석한 내용을 정리한 자료입니다.
> **다루는 내용:** 프로젝트 정체 → 폴더 구조 → 동작 원리 → 설치/사용법 → 수익화 아이디어 → PHP 재구현 검토

---

## 🔗 관련 GitHub 주소 모음

| 구분 | 주소 |
|---|---|
| **원본 저장소 (Uncle Bob)** | https://github.com/unclebob/swarm-forge |
| **내 포크** | https://github.com/bmshin94/swarm-forge |
| `two-pack` 브랜치 | https://github.com/unclebob/swarm-forge/tree/two-pack |
| `four-pack` 브랜치 | https://github.com/unclebob/swarm-forge/tree/four-pack |
| `six-pack` 브랜치 | https://github.com/unclebob/swarm-forge/tree/six-pack |
| `simple-windows` 태그 (구버전 스냅샷) | https://github.com/unclebob/swarm-forge/releases/tag/simple-windows |

### 헌법(constitution)에서 요구하는 외부 도구

| 언어 | 뮤테이션 | CRAP | DRY |
|---|---|---|---|
| Go | https://github.com/unclebob/mutate4go | https://github.com/unclebob/crap4go | https://github.com/unclebob/dry4go |
| Clojure | https://github.com/unclebob/clj-mutate | https://github.com/unclebob/crap4clj | https://github.com/unclebob/dry4clj |
| Java | https://github.com/unclebob/mutate4java | https://github.com/unclebob/crap4java | https://github.com/unclebob/dry4java |

- 인수 테스트 파이프라인: https://github.com/unclebob/Acceptance-Pipeline-Specification

---

## 1️⃣ 이게 뭐하는 물건이야?

### 한 줄 요약
> **tmux + git worktree 위에서 AI 에이전트(claude/codex/copilot/grok) 여러 개를 각각 다른 "역할"로 띄워놓고,
> 서로 작업을 넘겨주며(handoff) 협업시키는 로컬 오케스트레이션 시스템**

### 쉽게 비유하면 (식당 버전)

보통 AI 사용:
```
요리사 한 명 → "파스타 만들어줘" → 혼자 다 함 → 끝
```

SwarmForge:
```
🧑‍💼 주문받는 직원(specifier)  → 손님이 원하는 걸 정확히 명세로 정리
        ↓
👨‍🍳 요리사(coder)            → TDD로 실제 구현
        ↓
🧹 정리 담당(refactorer)      → 리팩터링, 커버리지 개선
        ↓
👔 매니저(architect)          → 구조 검토 후 "합격!" 도장 🎉
```

사용자는 **사장님** 역할: 작업 지시 + 승인/거절만 하면 됨.

### 누가 만들었나
**Robert C. Martin (Uncle Bob)** — 『Clean Code』, 『Clean Architecture』 저자.
그래서 에이전트에게 강제하는 "헌법"이 매우 엄격함 (TDD 필수, 커버리지·CRAP·DRY·뮤테이션 테스트 강제).

### ⚠️ 코인 사기 주의
README 최상단에 빨간 글씨 경고가 있음:
> **"Do not spend any money on a bankrbot SWARM token."**
> (이 프로젝트 이름을 도용한 가짜 코인이 있으니 절대 속지 말 것 🚨)

---

## 2️⃣ 폴더 구조 분석

### 최상위

| 경로 | 정체 |
|---|---|
| `README.md` (400줄) | 사실상 이 브랜치의 본체. 전체 사용설명서 |
| `get-swarm-forge` | **설치 스크립트**. `main`(공용 엔진) + pack 브랜치(역할 프롬프트)를 합성해서 내 프로젝트에 설치 |
| `close-swarm` | 스웜 강제 종료. tmux 세션·창 전부 정리 |
| `swarmforge/scripts/` | **엔진룸** (총 약 7,060줄) |
| `swarmforge/constitution/articles/` | **에이전트 헌법** 3종 |
| `swarmforge/handoff-protocol.md` | 에이전트 간 작업 전달 프로토콜 설계 문서 |
| `test/` + `bb.edn` | Babashka 유닛 테스트 + Playwright 대시보드 테스트 |
| `issues.md` | 개발자용 이슈 목록 (281줄) |
| `platoon-brainstorm.md` | **미구현** 미래 구상 — 여러 팀(squad)을 지휘관(Lieutenant)이 총괄하는 "중대" 개념 |
| `AGENTS.md` | "프롬프트 문구는 자동 테스트로 검증하지 마라" 규칙 |
| ❌ `LICENSE` | **없음!** (아래 라이선스 섹션 참고) |

### 엔진룸 핵심 파일 (줄 수 순)

| 파일 | 줄 수 | 역할 |
|---|---|---|
| `pack_web.bb` | 1,676 | 웹 대시보드 서버 |
| `swarm_handoff.bb` | 888 | 작업 넘기기 (검증 + 감사 게이트) |
| `swarmforge.bb` | 886 | **런처** — conf 읽고 worktree 만들고 tmux 띄움 |
| `pack/dashboard.html` | 915 | 대시보드 UI (칸반보드) |
| `handoffd.bb` | 428 | **우체부 데몬** — outbox 감시 → inbox 배달 → tmux 알림 |
| `pack_board.bb` | 360 | 보드 상태 관리 |
| `handoff_lib.bb` | 241 | 핸드오프 공용 로직 |
| `pack_dashboard_request.bb` | 234 | 승인/질문 요청 처리 |
| `swarm_tool.bb` | 223 | 외부 도구 설치/확인 헬퍼 |

**중요한 발견:** 전체 7,060줄 중 **2,591줄(약 37%)이 대시보드**(`pack_web.bb` + `dashboard.html`).

### 헌법 3종 (`swarmforge/constitution/articles/`)

| 파일 | 내용 |
|---|---|
| `engineering.prompt` | TDD 필수, CRAP·DRY·뮤테이션 도구 강제 설치/실행, Clojure는 Speclj 사용, 도구 동시 실행 금지 등 |
| `workflow.prompt` | 자기 worktree 밖으로 나가지 말 것, 커밋 메시지에 `By <role>.` 필수, 임시파일은 `./tmp/` 사용 |
| `handoffs.prompt` | 작업 전달은 반드시 `swarm_handoff.sh` 사용, tmux 직접 조작 금지, 막히면 `pack_dashboard_request.sh clarify`로 사람에게 질문 |

> 💡 pack 브랜치는 이 3개 파일명을 덮어쓸 수 없음.
> 추가 규칙이 필요하면 `local-engineering.prompt`, `local-workflow.prompt` 같은 `local-*` 이름을 사용.

---

## 3️⃣ 동작 원리

### 전체 흐름

```
[대시보드에서 New Task 입력]
        ↓
  specifier (Gherkin 명세 작성) → 사람이 Approve/Reject 👀
        ↓
    coder (TDD로 구현)
        ↓
  refactorer (정리·커버리지)
        ↓
   architect (구조 검토) → 완료 브로드캐스트 → Done 🎉
```

### 핵심 설계 3가지

**① git worktree 격리**
역할마다 `.worktrees/<역할>` 독립 작업 폴더를 가짐 → 서로 파일 안 밟음.
(단, `master`로 지정된 역할은 메인 체크아웃에서 작업)

**② 파일 기반 핸드오프 큐**
tmux로 직접 메시지를 쏘지 않고, `handoffd` 데몬이 파일을 옮겨 배달함.

```
.swarmforge/handoffs/
  outbox/     ← 보낼 것 (데몬이 감시)
  sent/       ← 배달 완료
  failed/     ← 실패
  inbox/
    new/          ← 받은 일감 (= 작업 큐)
    in_process/   ← 처리 중
    completed/    ← 완료
```
파일 위치가 곧 상태라서 **재시작해도 상태가 살아있음**.

파일명 규칙: `<우선순위>_<타임스탬프>_<순번>_from_<보낸역할>_to_<받는역할들>.handoff`

**③ ⭐ 감사 게이트 (AUDIT_REQUIRED)**
> **첫 번째 git handoff 호출은 무조건 반려된다.**

에이전트가 "다 했어요~" 하고 넘기려 하면:
1. 첫 호출 → `AUDIT_REQUIRED` 반환, 큐에 안 넣음, 감사 카운터 +1
2. 에이전트는 작업 전체를 다시 읽고 → 요구사항을 하나하나 추적 → 경계/실패 케이스 점검 → 발견한 문제 전부 수정 → 검증 재실행
3. 두 번째 호출이 **완전히 동일**해야 통과

드래프트/작업/보낸이/받는이/커밋 중 **하나라도 바뀌면 감사 무효**, 다시 카운트.
→ 대충 넘기는 걸 구조적으로 차단하는 장치. **이 아이디어가 가장 상품성 있음.**

### 에이전트가 쓰는 3개 명령어

| 명령어 | 하는 일 |
|---|---|
| `swarm_handoff.sh <draft>` | 작업 넘기기 (검증 + 감사 게이트) |
| `ready_for_next.sh` | 일 받기 (`NO_TASK` / `TASK:` / `BATCH:` 출력) |
| `done_with_current.sh` | 일 끝내기 (`MAIL_WAITING` / `NO_TASK` 출력) |

메시지 타입은 딱 2개: `git_handoff`(커밋 기반 작업 전달), `note`(80자 이내 한 줄 메모)

---

## 4️⃣ 설치 및 사용법

### 0) 준비물

```sh
for c in zsh git tmux bb claude; do
  printf "%-8s " "$c"; command -v $c >/dev/null && echo "✅" || echo "❌ 없음"
done
```

| 필요한 것 | 설치 |
|---|---|
| zsh | 맥 기본 탑재 |
| git | `brew install git` |
| tmux | `brew install tmux` |
| bb (Babashka) | `brew install borkdude/brew/babashka` |
| AI CLI 하나 이상 | `claude` / `codex` / `copilot` / `grok` (로그인 완료 상태여야 함) |

> ⚠️ macOS / Linux 전용. 윈도우는 WSL 안에서 실행.

### 1) `get-swarm-forge` 설치 (최초 1회)

```sh
mkdir -p ~/cmds
cp ~/swarm-forge/get-swarm-forge ~/cmds/
cp ~/swarm-forge/close-swarm     ~/cmds/
chmod +x ~/cmds/get-swarm-forge ~/cmds/close-swarm

echo 'export PATH="$HOME/cmds:$PATH"' >> ~/.zshrc
source ~/.zshrc

get-swarm-forge          # 사용법이 뜨면 성공
```

### 2) 프로젝트에 설치

```sh
mkdir ~/test-swarm && cd ~/test-swarm
get-swarm-forge four-pack claude
```

**문법:** `get-swarm-forge <브랜치> [에이전트] [추가 CLI 옵션...]`

| 브랜치 | 팀 구성 | 추천 상황 |
|---|---|---|
| `two-pack` | coder → cleaner | 작은 수정, 빠르게 |
| `four-pack` | specifier → coder → refactorer → architect | **입문 추천** ⭐ |
| `six-pack` | specifier → coder → cleaner → architect → hardender → QA | 대형 프로젝트 |

> 🚫 `main`은 부품 창고라서 실행 불가.

**설치 시 일어나는 일**
1. 원본 저장소(https://github.com/unclebob/swarm-forge)의 `main` 다운로드 → `scripts/` + 공용 헌법 3개 복사
2. 지정한 pack 브랜치 다운로드 → `swarm`, `swarmforge.conf`, `roles/`, `constitution.prompt` 복사
3. 필수 파일 하나라도 없으면 즉시 중단(fail fast)

**🔴 경고:** 설치 스크립트가 아래 3개를 **묻지 않고 삭제**함.
```sh
rm -rf swarmforge .worktrees .swarmforge
```
반드시 **빈 폴더 / 실험용 폴더**에서 실행할 것.

> 💡 다른 저장소에서 받고 싶으면:
> `SWARMFORGE_REPO_URL="https://github.com/bmshin94/swarm-forge" get-swarm-forge four-pack claude`
> 단, 내 포크에는 `main` 브랜치만 있어서 pack 브랜치 다운로드에 실패함 → 기본값(원본) 사용 권장.

### 3) 실행

```sh
./swarm
```

출력 예:
```
Dashboard: http://127.0.0.1:53412
pack_web start
window-invisible specifier claude master
...
```
브라우저가 자동으로 열림. 안 열리면 `cat .swarmforge/dashboard-url`

<details>
<summary>내부 동작 10단계</summary>

1. `swarmforge.conf` 읽어서 팀 구성 파악
2. 역할 프롬프트·헬퍼 스크립트·터미널 어댑터 검증
3. git 저장소가 아니면 `git init` + 첫 커밋 자동 생성
4. 역할별 `.worktrees/<역할>` worktree 생성 (`master`/`none` 제외)
5. 각 worktree에 스크립트·헌법 복사 후 PATH 등록
6. tmux 세션 생성 → 각 세션에서 AI CLI 실행
7. 웹 대시보드 시작 (포트는 매번 랜덤, `.swarmforge/dashboard-url`에 기록)
8. 잠자기 방지 (macOS `caffeinate -dims` / Linux `systemd-inhibit`)
9. `handoffd` 우체부 데몬 시작
10. `window`(visible) 지정된 역할만 터미널 창 오픈

</details>

### 4) 대시보드 사용법

```
┌───────────────────────────────────────────────────┐
│ SwarmForge  ● live   [New Task] [Open] [Teardown] │ ← 헤더
├───────────────────────────────────────────────────┤
│ ⚠️ Attention — 승인 요청 / AI의 질문                │ ← 여기 뜨면 반응!
├───────────────────────────────────────────────────┤
│ specifier │ coder │ refactorer │ architect │ Done │ ← 칸반보드
├───────────────────────────────────────────────────┤
│ Work Queue — 역할별 상태 + 활동 막대그래프          │
├───────────────────────────────────────────────────┤
│ 💬 Chat — master 에이전트와 대화                    │
└───────────────────────────────────────────────────┘
```

- **작업 시작:** `New Task` → **name**(짧고 안 바뀌는 이름, 예: `login-api`) + **task**(요청 내용) → OK
  - ⚠️ name은 카드 추적 키. **절대 중간에 바꾸지 말 것**
- **승인/거절:** Attention에 Approval 카드 → `Documents`로 산출물 확인 → `Approve` / `Reject`
  - `two-pack`은 승인 게이트 없이 즉시 전달
- **질문 답변:** `Request clarification`이 뜨면 텍스트 박스에 답변 (Approve/Reject 쓰는 거 아님)
- **관찰:** 카드 클릭 → 작업 내용 창 / Work Queue 역할명 클릭 → 해당 AI 화면 실시간 보기
- **완료:** 마지막 역할이 전체 브로드캐스트를 보내면 카드가 `Done`으로 이동

### 5) 종료

```sh
# 방법 1: 대시보드 우상단 Teardown 버튼 → 확인
# 방법 2: 터미널
cd ~/test-swarm && close-swarm
```
작업한 파일은 그대로 남음.

### 6) 설정 커스터마이징 (`swarmforge/swarmforge.conf`)

```conf
window-invisible specifier  claude master
window-invisible coder      claude coder --yolo
window-invisible refactorer codex  refactorer batch
window           architect  grok   architect batch
```

| 순서 | 필드 | 설명 |
|---|---|---|
| 1 | `window-invisible` / `window` | 터미널 창 숨김 / 표시 |
| 2 | 역할 이름 | `swarmforge/roles/<이름>.prompt`와 짝 |
| 3 | AI 종류 | `claude` `codex` `copilot` `grok` |
| 4 | 작업 폴더 | `master` = 메인 폴더, 그 외 = `.worktrees/<이름>` |
| 5 | `task` / `batch` (선택) | 하나씩 처리 / 같은 우선순위 몰아서 처리 |
| 6~ | 추가 옵션 (선택) | AI CLI에 그대로 전달 |

역할별로 다른 AI를 섞어 쓸 수 있음.

### 7) 환경변수

```sh
SWARMFORGE_OPEN_BROWSER=0 ./swarm      # 브라우저 자동 열기 끄기
SWARMFORGE_PREVENT_SLEEP=0 ./swarm     # 잠자기 방지 끄기
SWARMFORGE_TERMINAL=ghostty ./swarm    # ghostty|terminal-app|windows-terminal|none
SWARMFORGE_REPO_URL=...                # 설치 시 다른 저장소 사용
SWARMFORGE_BASE_BRANCH=...             # 공용 엔진을 가져올 베이스 브랜치
```

### 8) ⚠️ 꼭 알아야 할 주의사항

**① 권한 플래그가 자동으로 붙는다 🔓**
설정에 안 써도 런처가 알아서 붙임 (`swarmforge.bb` 확인):
- `claude` → `--permission-mode bypassPermissions`
- `codex` / `copilot` → `--yolo`

즉 **AI가 확인 없이 파일 수정·명령 실행**함. 반드시 격리된 실험용 폴더에서 시작할 것.

**② 요금 💸**
AI 4~6명이 동시에 돌고, 감사 게이트 때문에 같은 일을 두 번씩 함. 토큰 소모가 매우 큼.
입문은 `two-pack` + 작은 작업 하나로 시작 권장.

**③ 로컬 전용 💻**
컴퓨터를 계속 켜둬야 함. 그래서 잠자기 방지 기능이 내장되어 있음.

### 9) 문제 해결

| 증상 | 해결 |
|---|---|
| `get-swarm-forge: command not found` | PATH 등록 확인 → `source ~/.zshrc` |
| `missing required file after composing swarm` | 브랜치 이름 오타 확인 (`four-pack` 등) |
| 브라우저 안 열림 | `cat .swarmforge/dashboard-url` → 직접 열기 |
| "Swarm disconnected" 표시 | `close-swarm` 후 `./swarm` 재시작 |
| AI 화면 직접 보기 | `tmux -S "$(cat .swarmforge/tmux-socket)" ls` 후 attach |
| 창이 멈춘 것처럼 보임 | tmux 복사 모드일 수 있음. `q` 또는 `Esc` |

---

## 5️⃣ ⚖️ 라이선스 이슈 (중요!)

저장소 전체를 확인한 결과 **`LICENSE` 파일이 없고, 저작권 표시도 없음.**

> **라이선스 없음 = All Rights Reserved** (기본 저작권법 적용)
> 저자가 아무 권리도 명시적으로 부여하지 않은 상태

| 하려는 것 | 가능 여부 |
|---|---|
| 개인적으로 사용, 포크 | ✅ (GitHub 약관 범위) |
| 이걸로 만든 **결과물(내 코드)** 판매 | ✅ **완전 자유** |
| 코드 재배포 / 제품에 포함 / SaaS 판매 | ❌ **위험** |

**권장 대응**
1. 원본 저장소 이슈로 문의 → "상업적 사용 가능한가요? 라이선스 추가 계획 있나요?"
2. 엔진 코드는 복사하지 말고, 사용자가 `get-swarm-forge`로 직접 받게 할 것
3. 상업화 전 변호사 상담

---

## 6️⃣ 💰 수익화 아이디어

### 🟢 Tier 1 — 지금 당장 가능 (법적 이슈 없음)

**① 도구가 아니라 "생산성"을 판다** — 현실성 ★★★★★
SwarmForge는 작업 도구일 뿐. 개발 속도를 올려서 **외주 / MVP 대행 / 제품**을 판매.
결과물 코드는 100% 내 소유.

**② 콘텐츠 선점** — 현실성 ★★★★☆
한국어 자료가 거의 없음. 유튜브 / 블로그 연재 / 강의.
"Uncle Bob 이름값 + AI 에이전트 트렌드" 조합. 여기 모인 사람이 ③④의 고객이 됨.

### 🟡 Tier 2 — 내가 만든 것만 판매

**③ 도메인 특화 "팩" 판매** ⭐ 추천 — 현실성 ★★★★☆
```
main 브랜치  = 엔진 (Uncle Bob 저작물)  ← 안 건드림
pack 브랜치  = 역할 프롬프트 + 설정      ← 내가 쓴 것 = 내 저작물 ✅
```
사용자는 엔진을 원본에서 자동으로 받고(`get-swarm-forge`), 나는 **팩만** 판매.

예시: 유니티 게임팩 / 쇼핑몰팩 / React Native팩 / 국내 SI팩(한국 컨벤션+한국어 문서) / 데이터 분석팩
판매 방식: Gumroad 1회 결제 or 월 구독(프롬프트 업데이트 제공)

**④ 비용·성능 추적 도구** — 현실성 ★★★☆☆
**치명적 구멍 발견:** 대시보드에 **토큰/비용 표시가 전혀 없음.**
AI 4~6명 × 감사로 2회 작업 = 비용 폭탄인데 아무도 모름.

`.swarmforge/handoffs/`에 모든 기록이 파일로 남으므로 이를 파싱하면:
- 작업당 실제 비용 / 역할별 병목 / 감사 반려 횟수 통계

로그 파일만 읽는 별개 프로그램이라 **라이선스 문제 없음** ✅

### 🔵 Tier 3 — 아이디어만 차용해 새로 구현

**⑤ "AI 코드 감사 게이트"를 독립 제품으로** — 시장 크기 ★★★★★
`AUDIT_REQUIRED` 개념(첫 제출 무조건 반려 → 재검증 → 통과)은 아이디어라서 저작권 대상이 아님.
- GitHub Action: AI가 만든 PR 자동 2차 검증
- pre-commit hook: 커밋 전 자기검토 강제
- VSCode 확장: 완료 여부 체크리스트 자동 생성

"AI가 짠 코드 믿어도 되나?" 불안에 정확히 꽂히는 아이템.

**⑥ 아직 비어 있는 땅**
`platoon-brainstorm.md`의 "중대(Platoon)" 개념 — 여러 팀을 지휘관이 총괄 — 이 **미구현 상태**.
먼저 구현해 기여하면 평판 확보 → ①②로 직결.

### 추천 실행 순서

```
1단계 (이번 달)  →  ② 콘텐츠로 씨 뿌리기 + 직접 써보며 노하우 축적
                     ↓
2단계 (2~3개월)  →  ③ 도메인 팩 판매 + ④ 비용추적 도구 무료 배포(미끼)
                     ↓
3단계 (6개월~)   →  ① 외주로 수익화 + ⑤ 독립 제품 확장
```

---

## 7️⃣ 🐘 PHP로 재구현 가능한가?

### 결론: **가능. 오히려 유리한 부분이 많음.**

### 기능별 이식 난이도

| 기능 | 현재 구현 | PHP 대응 | 난이도 |
|---|---|---|---|
| 설정파일 파싱 | Babashka | `file()` + `preg_split()` | ★☆☆☆☆ |
| git worktree 생성 | 쉘 명령 | `symfony/process` | ★☆☆☆☆ |
| tmux 세션 제어 | 쉘 명령 | 동일하게 쉘 호출 | ★☆☆☆☆ |
| AI CLI 실행 | 쉘 명령 | `symfony/process` | ★☆☆☆☆ |
| 핸드오프 큐(파일 이동) | Babashka | `rename()` | ★☆☆☆☆ |
| 우체부 데몬(상시 실행) | Babashka 루프 | ReactPHP / Artisan 데몬 | ★★★☆☆ ⚠️ |
| **웹 대시보드 (2,591줄)** | 직접 만든 HTTP 서버 | **Laravel** | ★☆☆☆☆ 🎉 |
| 실시간 갱신 | JS 폴링 | SSE / Livewire | ★★☆☆☆ |

### PHP가 유리한 점
1. **대시보드가 압도적으로 쉬워짐** — 전체의 37%가 대시보드인데 이건 PHP 주특기. 드래그앤드롭까지 가능
2. **SQLite 붙이면 아이디어 ④가 공짜로 해결** — 소요시간/병목/비용 통계가 그냥 나옴
3. **SaaS 전환이 쉬움** — 웹 배포 생태계 최강
4. **개발자 구하기 쉬움** — Clojure/Babashka보다 인력 풀이 훨씬 넓음

### PHP가 불리한 점 (해결책 포함)
1. **상시 실행 데몬** — PHP는 요청-응답 모델이 기본
   - 해결 A: 단순 `while(true) { deliver(); sleep(1); }` + Supervisor 재시작 (충분함)
   - 해결 B: ReactPHP 이벤트 루프
   - 해결 C: Laravel Queue Worker (`--max-time`)
   - 에이전트는 몇 초 단위로 움직이므로 1초 폴링으로 충분
2. **메모리 누수** — 주기적 자동 재시작으로 해결
3. **`pcntl` / `posix` 확장 필요** — CLI PHP에는 보통 포함됨

→ **치명적인 문제 없음** ✅

### 추천 구조

```
swarmforge-php/
├── bin/
│   ├── swarm                    # 진입점
│   └── agent-tools/             # 에이전트 PATH에 꽂히는 얇은 쉘 래퍼
│       ├── handoff              #   → php artisan swarm:handoff
│       ├── ready-for-next       #   → php artisan swarm:next
│       └── done                 #   → php artisan swarm:done
├── app/
│   ├── Console/Commands/
│   │   ├── SwarmStart.php       # 스웜 기동
│   │   ├── SwarmDaemon.php      # 우체부 데몬
│   │   └── SwarmDown.php        # 정리
│   ├── Swarm/
│   │   ├── ConfigParser.php
│   │   ├── WorktreeManager.php
│   │   ├── TmuxDriver.php
│   │   ├── HandoffQueue.php
│   │   ├── AuditGate.php        # ⭐ 핵심 아이디어
│   │   └── CostTracker.php      # 💰 차별화 기능
│   └── Livewire/Board.php       # 칸반보드
└── database/swarm.sqlite
```

**설계 포인트:** AI가 쓰는 명령어는 얇은 쉘 래퍼로 두고 내부에서 PHP 호출.
에이전트 입장에선 구현 언어를 알 필요가 없음.

### 맛보기 코드

```php
// ConfigParser.php
public function parse(string $path): array {
    $roles = [];
    foreach (file($path, FILE_IGNORE_NEW_LINES | FILE_SKIP_EMPTY_LINES) as $line) {
        if (!str_starts_with($line, 'window')) continue;
        [$directive, $role, $agent, $worktree, ...$rest] = preg_split('/\s+/', trim($line));
        $roles[] = new RoleConfig(
            role:      $role,
            agent:     $agent,
            worktree:  $worktree,
            visible:   $directive === 'window',
            mode:      in_array($rest[0] ?? '', ['task','batch']) ? array_shift($rest) : 'task',
            extraArgs: $rest,
        );
    }
    return $roles;
}
```

```php
// HandoffQueue.php — 배달은 rename() 한 줄 (원자적이라 안전)
public function deliver(Handoff $h): void {
    foreach ($h->recipients as $to) {
        copy($h->path, $this->inboxPath($to, $h->filename()));
        $this->tmux->wakeUp($to);
    }
    rename($h->path, $this->sentPath($h->filename()));
}
```

### 🎉 라이선스 보너스
**PHP로 처음부터 새로 짜면 = 클린룸 재구현 = 100% 내 저작물**
저작권은 **표현(코드)**을 보호하지 **아이디어(아키텍처)**를 보호하지 않음.

| 해도 되는 것 ✅ | 하면 안 되는 것 ❌ |
|---|---|
| 아키텍처 개념 차용 (worktree 격리, 파일 큐, 감사 게이트) | 코드 복사·번역 붙여넣기 |
| README 읽고 이해해서 새로 설계 | 프롬프트 문구 그대로 사용 |
| 더 나은 방식으로 재구현 | 문서 복붙 |
| 다른 이름으로 배포 | 동일 이름 사용 |

→ 앞서 말한 **Tier 3 전략의 실행 방법이 바로 이것**. 단, 상업화 전 변호사 확인 권장.

### MVP 로드맵

| 단계 | 내용 | 결과 |
|---|---|---|
| 1주차 | conf 파싱 → worktree 생성 → tmux에 AI 띄우기 | AI 2명이 각자 방에서 실행됨 |
| 2주차 | 파일 큐 + 우체부 데몬 + 최소 대시보드 | 카드가 실제로 이동 |
| 3주차 | 감사 게이트 + 승인 UI | 품질 통제 작동 |
| 4주차~ | 비용 추적 + 통계 | 차별화 완성 |

### ⚠️ 마지막 조언
**tmux를 PHP로 대체하려 하지 말 것.** tmux는 이미 완성된 도구고 직접 구현하면 그것만 몇 달 걸림.
`shell_exec('tmux ...')`가 정답. 새로 만들 것은 **"지휘 계층"이지 "터미널"이 아님.**

---

## 📌 핵심 요약

1. SwarmForge = **AI 에이전트를 팀으로 굴리는 로컬 오케스트레이션 도구** (Uncle Bob 제작)
2. 핵심 3요소 = **worktree 격리** + **파일 기반 핸드오프 큐** + **감사 게이트(AUDIT_REQUIRED)**
3. 설치 = `get-swarm-forge <pack> <agent>` → `./swarm` → 브라우저 대시보드
4. ⚠️ **라이선스 없음** → 코드 재배포/판매 위험, 결과물 판매는 자유
5. 💰 수익화는 **콘텐츠 → 도메인 팩 → 비용추적 도구 → 독립 제품** 순서 추천
6. 🐘 **PHP 재구현 가능**하며 대시보드·통계는 오히려 유리 + 라이선스 문제까지 해결
