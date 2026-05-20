# Claude Code 워크북

> 각 단계가 "무엇을 한 것인지", "왜 필요한지", "앞으로 어떻게 활용하는지"까지 포함.
>
> 대상 OS: **Apple Silicon Mac(M1 이상) 기준**, Windows(WSL)도 병기.
> 작성일: 2026-05-18

---

## 워크북 한눈에 보기

이 워크북의 전체 그림은 **"바이브 코더가 한 명의 빌더가 되는 4단계"** 다.

| 단계 | 무엇을 얻는가 | 핵심 도구 |
|---|---|---|
| **1** | "CC로 앱 하나를 만들고 배포까지 해봤다"는 첫 경험 | Claude Code · Git · GitHub · Vercel · Cursor |
| **2** | "이게 어떻게 굴러가는지"의 아키텍처 감각 | Firebase Firestore · Auth · API · WebSocket |
| **3** | CC 자체를 내 입맛에 맞게 **확장**하는 능력 | Skill · Command · Hook · MCP · Plugin |
| **4** | CC가 **자동으로 일하게** 만드는 워크플로우 | tmux · 하네스 · /loop · wj-magic · papyrus · hermes |

---

## 1. 기본 세팅 — 환경 만들기

### 1-1. 사전 준비

#### ✅ Claude 계정 만들기

| 단계 | 내용 |
|---|---|
| 1 | [claude.ai](https://claude.ai)에 접속해 이메일로 가입 |
| 2 | Pro 또는 Max 플랜 가입 (Claude Code의 안정적 사용을 위해 권장) |
| 3 | 가입한 동일 이메일로 `claude.ai/code`에 로그인되는지 확인 |
| 4 | 결제 카드 등록 — 무료 플랜으로는 Claude Code 사용량 한계가 빠르게 옴 |

> 💡 **무엇을 한 것?** Claude Code(CC)가 Claude 계정의 API/사용량을 인식하도록 하는 단계. CC 첫 실행 시 브라우저로 자동 로그인 창이 떠서, 이 계정을 연결한다.

#### ✅ Windows 사용자만: WSL 설치

**Mac 사용자는 건너뛰기.** 맥은 이미 유닉스 기반이라 추가 설치 불필요.

| 단계 | 내용 |
|---|---|
| 1 | Microsoft Store에서 "Windows Terminal" 설치 |
| 2 | Windows Terminal을 **관리자 권한**으로 실행 → `wsl --install` |
| 3 | 컴퓨터 재부팅 |
| 4 | 자동으로 Ubuntu 설치됨 → 사용자 이름/비밀번호 설정 |
| 5 | Windows Terminal에서 Ubuntu 탭으로 진입하면 리눅스 환경 |

> ⚠️ 비밀번호 입력 시 화면에 아무것도 안 보이는 게 정상이다. 그냥 치고 Enter.

> 💡 **WSL이 뭔지?** "Windows Subsystem for Linux" — 윈도우 안에서 리눅스를 돌리는 가상 환경. 이 워크북에서 다루는 도구들이 리눅스 기반이라 통일성을 위해 사용한다.

---

### 1-2. 터미널 & 패키지 매니저

#### ✅ 터미널 열기

| OS | 방법 |
|---|---|
| Mac | Spotlight(`Cmd+Space`) → "터미널" 검색 → Terminal.app 실행 |
| Windows | Windows Terminal → Ubuntu 탭 선택 |

#### ✅ 터미널 복사·붙여넣기 단축키

| OS | 복사 | 붙여넣기 |
|---|---|---|
| Mac | `Cmd + C` | `Cmd + V` |
| WSL | `Ctrl + Shift + C` | `Ctrl + Shift + V` |

> ❌ WSL에서 그냥 `Ctrl + C`는 "중단(실행 중지)" 명령이라 복사가 아니다. 진행 중인 프로그램이 꺼져버릴 수 있으니 **반드시 Shift를 같이 누른다.**

#### ✅ Homebrew 설치 (Mac만)

**Homebrew = 맥용 앱스토어의 터미널 버전.** 프로그램을 한 줄 명령어로 깔게 해준다.

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

> ⚠️ 반드시 **일반 터미널에서 실행**. Claude Code 안에서 실행하면 sudo 비밀번호 입력이 불가능하다.
> ⚠️ 설치 중 비밀번호를 묻는다. **화면에 안 보이는 게 정상.** 그냥 치고 Enter.

Apple Silicon Mac(M1/M2/M3/M4)은 설치 후 PATH 등록이 추가로 필요하다:

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

> 💡 **PATH 등록을 안 하면** "command not found: brew" 에러가 난다. `~/.zprofile`은 로그인 시 1회 실행되는 설정 파일이고, 위 명령어로 brew 경로를 거기에 박아두면 영구 적용된다.

#### ✅ WSL: 패키지 목록 업데이트

WSL의 Ubuntu에는 `apt`라는 패키지 매니저가 이미 들어 있다. 목록만 최신화:

```bash
sudo apt update && sudo apt upgrade -y
```

---

### 1-3. 기본 도구 설치

> ℹ️ 이미 설치된 도구가 있다면 `which 도구이름` 으로 확인 가능. 워크북 표준(fnm 등)으로 통일 권장. 환경이 같아야 트러블슈팅이 쉽다.

#### ✅ Node.js (fnm 사용)

**fnm = Node.js 버전 관리 도구.** Node를 직접 깔지 않고 fnm을 통해 깐다.

| OS | 명령 |
|---|---|
| Mac | `brew install fnm` |
| WSL | `curl -fsSL https://fnm.vercel.app/install \| bash` |

설치 후 셸 설정 파일에 fnm 환경변수를 추가한다:

**Mac (`nano ~/.zshrc`):**
```bash
eval "$(fnm env --use-on-cd --shell zsh)"
```

**WSL (`nano ~/.bashrc`):**
```bash
eval "$(fnm env --use-on-cd --shell bash)"
```

저장(`Ctrl+O` → Enter), 종료(`Ctrl+X`) 후 적용:

```bash
# Mac
source ~/.zshrc
# WSL
source ~/.bashrc

fnm install --lts
```

확인:
```bash
node --version
npm --version
```

> 💡 **왜 fnm을 쓰나?** Node 공식 사이트에서 직접 깔면 버전 변경이 어렵다. fnm을 쓰면 프로젝트마다 다른 Node 버전을 쉽게 전환할 수 있다.

#### ✅ Claude Code 설치

```bash
npm install -g @anthropic-ai/claude-code
```

확인:
```bash
claude --version
```

#### ✅ Git / Python /  설치

| 도구 | Mac | WSL |
|---|---|---|
| Git | `brew install git` | `sudo apt install git` |
| Python | `brew install python` | `sudo apt install python3 python3-pip` |
|  | `brew install ` | `sudo apt install tmux` |

#### ✅ Git 초기 설정 (Mac/WSL 동일)

다음 정보를 입력:

```bash
git config --global user.name "사용자이름"
git config --global user.email "사용자이메일@example.com"
```

> ⚠️ Git config를 안 하면 첫 커밋 시 "Author identity unknown" 에러. **반드시 설정.**

#### ✅ gh CLI (GitHub 명령어 도구)

**Mac:**
```bash
brew install gh
```

**WSL** (별도 저장소 추가 후 설치):
```bash
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update
sudo apt install gh
```

> ⚠️ WSL에서 `sudo apt install gh`만으로는 설치 안 됨. 위 4줄을 순서대로 다 실행해야 한다.

---

### 1-4. alias 설정 & 첫 실행

CC를 매번 길게 치지 않도록 단축어(alias)를 설정한다.

설정 파일 열기:
| OS | 명령 |
|---|---|
| Mac | `nano ~/.zshrc` |
| WSL | `nano ~/.bashrc` |

파일에 추가:
```bash
alias cc="claude --dangerously-skip-permissions"
alias ccr="claude --dangerously-skip-permissions --resume"
alias work="cd ~/Documents/projects"
```

저장(`Ctrl+O` → Enter), 종료(`Ctrl+X`), 적용:
```bash
source ~/.zshrc   # Mac
source ~/.bashrc  # WSL
```

> 💡 **alias가 뭔지?**
> - `cc` → 매번 `claude --dangerously-skip-permissions` 안 쳐도 됨 (bypass 모드)
> - `ccr` → 이전 대화 이어서(resume) 시작
> - `work` → 작업 폴더로 빠르게 이동
>
> ⚠️ `--dangerously-skip-permissions`는 CC가 매번 "이 명령어 실행해도 될까요?"를 묻지 않게 한다. 빌더가 모든 명령어를 일일이 확인할 필요가 없게 되지만, **로컬 환경에 한해 사용**할 것.

#### ✅ CLAUDE.md 만들기

프로젝트마다 최상위에 `CLAUDE.md` 파일을 둔다. CC에게 "이 프로젝트는 이런 거야"를 알려주는 파일.

```bash
mkdir my-first-app
cd my-first-app
nano CLAUDE.md
```

예시 내용:
```markdown
# My First App

간단한 메모장 앱.

- 프론트엔드: React (Vite)
- 데이터 저장: 로컬 (브라우저 localStorage)
- 디자인: 미니멀, 화이트 베이스
```

> 💡 **왜 필요한가?** CLAUDE.md가 없으면 CC가 매번 맥락 없이 일반적인 답변을 한다. 이 파일이 있으면 CC가 "아 이 사람은 이런 프로젝트에서 일하는구나"를 알고 일관된 답을 한다.

#### ✅ CC 첫 실행

```bash
cc
```

→ 브라우저가 열리며 로그인 요청 → Claude 계정으로 로그인 → 터미널로 자동 복귀.

CC가 켜졌으면 한 번 쳐보자:
```
간단한 메모장 웹앱 하나 만들어줘. 메모를 추가하고 삭제할 수 있으면 돼.
```

CC가 파일을 만들고, 패키지를 설치하고, 서버를 띄울 것이다. **"와 진짜 되네"** 이 경험이 1단계의 핵심 목표.

> ℹ️ CC 종료: `/exit` 또는 `Ctrl+C`

---

### 1-5. 첫 앱 빌드 + 깃 + 배포

#### ✅ 빌더의 기본 습관

| 상황 | 이렇게 |
|---|---|
| 뭔가 안 될 때 | 에러 메시지를 CC에게 그대로 붙여넣기 |
| 원하는 게 안 나올 때 | 더 구체적으로. "파란 버튼" 대신 "왼쪽 상단에 파란 배경 흰 글씨 버튼" |
| CC가 엉뚱한 걸 할 때 | "아니, 그게 아니라 이거야" 하고 교정 |
| 처음부터 다시 | `/clear` 또는 새 터미널에서 `cc` 다시 |

> 💡 **CC가 잘하는 것**: 코드 작성, 에러 수정, 구조 잡기, 반복 작업.
> **CC가 못하는 것**: 디자인 감각, 비즈니스 판단, "예쁘게 해줘"의 정확한 의미.

#### ✅ Git/GitHub 연동

**왜 필요한가?** Git 없이 작업하다 CC가 코드를 잘못 고치면 되돌릴 방법이 없다. 게임의 세이브포인트와 같다.

| 단계 | 내용 |
|---|---|
| 1 | github.com에서 계정 생성 |
| 2 | GitHub Desktop 설치 (Mac/Windows 모두 지원) — [desktop.github.com](https://desktop.github.com) |
| 3 | GitHub Desktop에서 레포(저장소) 생성 |
| 4 | CC에게 "지금까지 작업한 거 커밋해줘" / "변경사항 깃에 저장해줘" |

> ⚠️ 커밋 = 세이브. 기능 하나 완성할 때마다 커밋하는 습관을 들일 것.

#### ✅ Vercel로 첫 배포

**왜 필요한가?** 내가 만든 웹앱을 인터넷에 올려서 누구나 볼 수 있게 하는 단계. 무료.

| 단계 | 내용 |
|---|---|
| 1 | [vercel.com](https://vercel.com) 가입 (GitHub 계정으로 로그인) |
| 2 | GitHub 레포를 Vercel에 연결 |
| 3 | 자동으로 빌드 + 배포 |
| 4 | URL이 생김 → 누구나 접속 가능 |

#### ✅ .env (환경변수)

비밀번호나 API 키처럼 **남에게 보여주면 안 되는 값**을 보관하는 파일.

```
GOOGLE_API_KEY=abc123...
DATABASE_PASSWORD=secret...
```

> ❌ **.env는 절대 GitHub에 올리면 안 된다.** 비밀번호가 전 세계에 공개됨.
> CC에게 "**.gitignore에 .env 추가해줘**"라고 처음에 말해둘 것.

#### ✅ 3가지 영역 (개념 이해)

| 영역 | 역할 | 비유 |
|---|---|---|
| 프론트엔드 | 사용자가 보는 화면 | 식당의 홀 |
| 백엔드 | 데이터 처리, 로직 | 식당의 주방 |
| 클라이언트 | 사용자의 브라우저/앱 | 식당에 오는 손님 |

메모장 앱에서:
- 메모가 화면에 보이는 것 = **프론트엔드**
- 메모가 저장/삭제되는 것 = **백엔드(또는 로컬 저장소)**
- 브라우저에서 앱을 여는 것 = **클라이언트**

---

### 1-6. 환경 완성 (Cursor / tmux / Termius / Tailscale)

#### ✅ Cursor — CC와 함께 쓰는 코드 편집기

[cursor.com](https://cursor.com)에서 다운로드 (Mac/Windows 모두 지원).

- VS Code 기반이라 비슷하게 생김
- Cursor 안에서 터미널 열기: **`Ctrl + ` (백틱)**

> ⚠️ **Windows 사용자만**: Cursor 내장 터미널 기본이 PowerShell이라 WSL로 바꿔야 함.
> 설정(`Ctrl+,`) → "terminal.integrated.defaultProfile.windows" 검색 → "Ubuntu (WSL)" 선택.

| 기능 | 설명 |
|---|---|
| bypass 모드 | CC가 파일 수정 시 매번 안 물어보고 자동 진행. `cc` alias 사용하면 자동 적용 |
| `/btw` | CC에게 "참고로 이건 이래" 하고 추가 맥락을 주는 명령어 |

#### ✅ 컴퓨터 항상 켜져 있게

**왜 필요한가?** 4단계에서 CC를 24시간 돌리려면 컴퓨터가 꺼지면 안 된다.

| OS | 방법 |
|---|---|
| Mac | App Store에서 **Amphetamine** 무료 설치 → 활성화 |
| Windows | 설정 → 시스템 → 전원 → 절전 모드 "없음" |

> ⚠️ 노트북 덮개를 닫으면 절전에 들어가서 CC가 중단될 수 있다. Mac은 Amphetamine으로 덮개 닫아도 안 꺼지게 설정 가능.
> ⚠️ M1+ MacBook은 Amphetamine 메뉴바 아이콘이 노치 뒤로 숨을 수 있다. **Cmd를 누른 채 메뉴바 아이콘을 드래그**해서 노치 밖으로 꺼낼 것.

#### ✅ tmux — 터미널 세션 유지

**왜 필요한가?**
- tmux 없이 터미널을 닫으면 실행 중이던 CC가 즉시 종료됨
- tmux 안에서 CC를 돌리면 터미널을 닫아도 백그라운드에서 계속 돌아감
- 4단계의 자율 루프를 돌리려면 필수

설치는 1-3에서 이미 했음. 확인:
```bash
which tmux
```

기본 사용법:
```bash
tmux new -s mywork      # 새 세션 생성 (이름: mywork)
cc                      # 그 안에서 CC 실행
# Ctrl+B 다음 D 누르면 세션에서 빠져나옴 (CC는 계속 돌아감)
tmux attach -t mywork   # 다시 들어가기
tmux ls                 # 세션 목록 보기
tmux kill-session -t mywork  # 세션 종료
tmux set -g mouse on #마우스 스크롤 제어
Ctrl + b를 누른 후 % #화면 쪼개쓰기
Ctrl + b 누른 후 손 떼고 ➡️ Spacebar #화면 균등분할하기
```

> 💡 **앞으로 어떻게 활용?** CC에게 긴 작업을 시킬 때(예: "전체 코드베이스 리뷰해줘") 반드시 tmux 안에서 실행. 그래야 잠깐 자리를 비우거나 노트북을 닫아도 작업이 안 끊긴다.

#### ✅ Termius — 외부에서 내 컴퓨터 접속

**왜 필요한가?** 메인 PC에 CC를 돌려놓고, 외부에서 스마트폰으로 진행 상황을 확인하고 싶을 때.

- 스마트폰/태블릿에서 메인 PC의 터미널 접속 (Mac/Windows 모두 지원)
- 앱스토어/플레이스토어에서 "Termius" 검색해 무료 설치
- 메인 PC의 SSH 정보 등록

> 💡 **활용 시나리오**: 메인 PC에 tmux로 CC 세션 띄워놓고 → 외출 → 이동 중인 곳에서 스마트폰 Termius로 접속 → CC가 어디까지 했는지 확인 → 지시 추가.

#### ✅ Tailscale — 어디서든 안전하게 연결

**왜 필요한가?** Termius로 외부에서 메인 PC에 접속하려면 보안 연결이 필요하다. Tailscale은 그 보안 연결을 한 줄로 설정해주는 도구.

- [tailscale.com](https://tailscale.com) 가입 (무료 플랜으로 충분)
- 메인 PC와 스마트폰에 모두 설치 → 같은 계정으로 로그인
- 두 기기가 마치 같은 네트워크에 있는 것처럼 동작

> 💡 **이 모든 셋팅이 완료되면**: 메인 PC에 CC를 돌려놓고, 외부에서 스마트폰으로 진행 상황을 확인하고 지시를 내릴 수 있다.

---

## 2. 아키텍처 이해 — 진짜 서비스로

1단계에서 만든 메모장 앱을 **계속 업그레이드하면서** 아키텍처를 체득한다. 이론 먼저가 아니라 기능을 붙여보고 "아 이래서 이게 필요하구나"를 느끼는 것.

| 순서 | 주제 | 핵심 경험 |
|---|---|---|
| 1 | Firebase Firestore | 브라우저 닫아도 데이터가 사라지지 않는다 |
| 2 | Firebase Auth | 내 메모만 내가 본다 (로그인) |
| 3 | API 연동 | 외부 서비스의 기능을 빌려쓴다 |
| 4 | 실시간 통신 (WebSocket) | 새로고침 없이 화면이 자동으로 바뀐다 |
| 5 | 프론트엔드 / 백엔드 분리 | "화면 담당"과 "데이터 담당"이 나뉜다 |
| 6 | 빌더 마인드셋 | 제품에 대한 책임감 |

### 2-1. Firebase Firestore — 데이터 저장

**왜 필요한가?** 지금까지 만든 메모장은 브라우저를 닫으면 데이터가 사라진다. Firestore를 연결하면 데이터가 구글 서버에 저장된다.

**실습:**
1. CC에게 "이 메모장 앱에 Firebase Firestore 연결해줘" 요청
2. Firebase 콘솔([console.firebase.google.com](https://console.firebase.google.com))에서 프로젝트 생성
3. `.env` 파일에 Firebase 설정 키 넣기

> ⚠️ `.env`는 프로젝트 폴더 최상위 (CLAUDE.md와 같은 위치)에 만든다. 그리고 `.gitignore`에 `.env`가 추가되어 있는지 반드시 확인.
> ℹ️ Firebase Spark 플랜은 완전 무료. 이 워크북 범위에서 과금 걱정 없음.

### 2-2. Firebase Auth — 로그인

**왜 필요한가?** "이 앱은 아무나 쓸 수 있나?"에 대한 답. 내 메모를 다른 사람이 보면 안 되니까.

**실습:**
1. CC에게 "Google 로그인 기능 추가해줘"
2. Firebase 콘솔 → Authentication → Google 로그인 활성화

> ⚠️ Firebase 콘솔에서 "Google" 로그인 제공업체를 직접 활성화해야 한다. CC가 코드는 만들어주지만 콘솔 설정은 직접.

### 2-3. API 연동

**왜 필요한가?** 내 앱이 다른 서비스의 기능을 빌려쓰는 것. 예: 날씨 앱이 기상청에 "오늘 날씨 알려줘"라고 요청하는 것.

- **API 키** = 그 서비스를 쓸 수 있는 비밀 열쇠
- CC에게 "이 앱에 날씨 API 연동해줘" 또는 "번역 API 붙여줘"

> ⚠️ API 키는 비밀번호와 같다. 코드에 직접 쓰지 말고 반드시 `.env` 파일에 보관할 것. GitHub에 올라가면 누군가 내 키로 과금을 발생시킬 수 있다.

### 2-4. 실시간 통신 (WebSocket)

**왜 필요한가?** 데이터가 바뀌면 자동으로 반응. 새로고침 없이 화면이 바뀌는 경험.

- **이벤트** = "뭔가 일어난 것" (데이터 변경, 버튼 클릭, 메시지 도착 등)
- **WebSocket** = 실시간으로 데이터를 주고받는 방식
  - 비유: 전화 통화처럼 연결이 계속 유지됨 (일반 API는 편지처럼 한 번 보내고 끝)
- Firestore의 `onSnapshot`: 데이터가 바뀌면 화면이 자동으로 업데이트

> ⚠️ 실시간 통신은 연결이 계속 유지되므로 서버 자원을 더 쓴다. 모든 기능에 실시간이 필요한 건 아님. "이걸 실시간으로 해야 하나?"를 판단하는 것도 빌더의 역할.

### 2-5. 프론트엔드 / 백엔드 분리

1단계에서 배운 "식당 홀/주방" 구조를 실제로 분리해본다.

| 단계 | 역할 | 예시 |
|---|---|---|
| 클라이언트 (브라우저) | 사용자가 보고 조작하는 곳 | "새 메모" 버튼 클릭 |
| 서버 (백엔드) | 요청을 받아 처리하는 곳 | 메모 데이터를 검증하고 저장 명령 |
| DB (Firestore) | 데이터가 실제로 저장되는 곳 | 메모 내용 저장 |

**실습:** CC에게 "프론트엔드와 백엔드를 분리해줘" 요청 → 분리 전과 후의 차이를 직접 체감.

### 2-6. 빌더 마인드셋

> ※ 1단계에서 했으면 와닿지 않았을 이야기. 앱을 실제로 키워본 뒤에 다루는 마인드셋.

- **빌더로서의 자세**: 내가 만든 것에 대한 주인의식. "CC가 만들어줬으니 난 모르겠다"가 아님.
- **한계 인식의 책임**: 모르는 건 모른다고 아는 것. 모르는 걸 아는 척하고 배포하면 사고가 남.
- **제품 배포의 책임**: URL이 생기면 누구나 접속할 수 있음.
  - ❌ 인증 없이 배포하면 누구나 데이터 볼 수 있고, `.env` 키가 노출되면 과금 발생.
  - "돌아가니까 됐다"가 아니라 **"안전한가?"** 를 반드시 확인.
- **제품 관리의 책임**: 만든 뒤에도 책임은 계속. 버그 발견, 보안 업데이트, 사용자 문의 대응.

> 💡 이 마인드셋은 3, 4단계로 갈수록 더 중요해진다. 자동화를 하더라도 **최종 책임은 빌더에게** 있다.

---

## 3. CC 확장 — 내 능력 부여하기

1~2단계에서는 CC를 "있는 그대로" 썼다면, 여기서는 CC에게 **새로운 능력을 부여**하고, 자동화하고, 외부 도구를 연결한다.

작은 단위에서 큰 단위로 확장하는 구조:
**스킬(가장 단순) → 커맨드 → 훅 → MCP(외부 연결) → 플러그인(묶음)**

| 순서 | 주제 | 비유 |
|---|---|---|
| 1 | 스킬 (Skill) | CC에게 새 기술을 가르치는 것 |
| 2 | 커맨드 (Command) | 자주 쓰는 기술에 단축키를 거는 것 |
| 3 | 훅 (Hook) | CC가 뭔가 할 때 자동으로 끼어드는 것 |
| 4 | MCP | CC에게 외부 도구를 손에 쥐어주는 것 |
| 5 | 플러그인 (Plugin) | 위 전부를 하나로 묶은 패키지 |

### 3-1. 스킬 (Skills)

**스킬 = CC에게 새로운 능력을 부여하는 것.** 마크다운 파일 하나로 CC의 행동 방식을 정의한다.

| 종류 | 설명 | 예시 |
|---|---|---|
| 빌트인 스킬 | CC에 기본 내장된 스킬 | `/commit` (커밋 메시지 작성) |
| 커스텀 스킬 | 직접 만드는 스킬 | `/review` (내 팀 코딩 규칙에 맞는 코드 리뷰) |

**스킬 만드는 법:**
1. 프로젝트의 `.claude/skills/` 폴더에 마크다운 파일 생성
2. 파일 안에 CC가 어떻게 행동해야 하는지 적음
3. CC 실행 시 자동으로 인식

> ⚠️ 스킬 파일은 반드시 `.claude/skills/` 폴더 안에 있어야 한다. 다른 위치에 만들면 CC가 인식하지 못함.

**실습 예시:** `/요약` 스킬 — 긴 텍스트를 3줄로 요약해주는 스킬

### 3-2. 커맨드 (Commands)

**커맨드 = 자주 쓰는 작업을 `/슬래시` 한 줄로 실행하는 것.**

- CC 대화 중에 `/`를 치면 사용 가능한 커맨드 목록이 나옴
- 예: `/commit`, `/clear`, `/help`
- 자주 반복하는 작업을 커맨드로 만들면 매번 길게 설명할 필요 없음

**실습 예시:** `/주간보고` → 이번 주 작업 내역을 정리해서 포맷에 맞게 작성

> 💡 스킬이 "CC에게 능력을 가르치는 것"이라면, 커맨드는 "그 능력을 한 줄로 부르는 단축키".

### 3-3. 훅 (Hooks)

**훅 = CC가 특정 동작을 할 때 자동으로 끼어드는 것.**

> ⚠️ 2단계에서 배운 "이벤트 감지"와는 다른 개념. 2단계의 이벤트 감지는 **앱의 데이터 변화**를 감지하는 것이고, 여기서 말하는 훅은 **CC 자체의 동작**에 거는 것.

| 종류 | 시점 | 예시 |
|---|---|---|
| Pre 훅 | CC가 뭔가 하기 **전**에 실행 | 파일 수정 전에 위험한 파일인지 체크 |
| Post 훅 | CC가 뭔가 한 **후**에 실행 | 코드 작성 후 자동으로 품질 검사 |

비유:
- Pre 훅: 문을 열기 전에 "잠겨있는지" 확인
- Post 훅: 문을 나서면서 "불 껐는지" 자동 체크

**실습 예시:**
- 커밋 전 자동 체크: 커밋하기 전에 코드에 문제가 없는지 자동 검사
- 응답 후 자동 검증: CC가 코드를 작성할 때마다 파일 크기/규칙 위반 자동 체크

> ⚠️ 훅은 `settings.json`에 설정한다. CC 대화에서 "이거 자동으로 해줘"라고 말하는 것과는 다름. **훅은 CC가 의식하지 못하는 사이에 시스템 레벨에서 실행됨.**

### 3-4. MCP (Model Context Protocol)

**MCP = CC가 외부 도구/서비스와 통신하는 방식. CC의 손에 도구를 쥐어주는 것.**

- CC 혼자서는 이메일을 읽거나 브라우저를 조작할 수 없음
- MCP 서버를 연결하면 CC가 외부 도구를 직접 사용할 수 있게 됨
- 예: Gmail MCP를 연결하면 CC가 메일을 읽고 분류할 수 있음

| 용어 | 설명 |
|---|---|
| MCP 서버 | 외부 도구를 CC에 연결해주는 중간 다리 |
| MCP 클라이언트 | CC 자체 (도구를 사용하는 쪽) |
| 마켓플레이스 | 다른 사람이 만든 MCP 서버를 찾아서 설치할 수 있는 곳 |

**설정 방법 (Mac/WSL 동일):**

```bash
# CLI로 MCP 서버 추가 (권장)
claude mcp add 서버이름 명령어

# 또는 설정 파일 직접 편집
# 위치: ~/.claude.json 의 mcpServers 항목
```

> ⚠️ `settings.json`을 직접 편집할 경우 JSON 형식이라 쉼표 하나 빠져도 에러 남. 불안하면 CC에게 "이 MCP 서버 settings.json에 추가해줘"라고 요청.

### 3-5. 실시간 검색 확장 — Brave Search + Smithery + 소켓

> ※ 3-4의 MCP를 가장 흔하게 만나는 실전 조합. 터미널의 CC에게 **'실시간 인터넷 검색 능력'** 을 부여하기 위해 결합되는 일종의 "세트 상품"이다.

CC는 과거 시점까지의 데이터로 학습되어 있어 "오늘의 날씨"나 "최신 fnm 설치 가이드"를 스스로 검색하지 못한다. 이 한계를 풀기 위해 세 프로그램이 협업한다.

| 구성요소 | 역할 | 한 줄 비유 |
|---|---|---|
| **Brave Search** | AI 친화적 검색 API를 제공하는 검색 엔진 | 지식의 원천 (구글/네이버 위치) |
| **Smithery.ai** | MCP 서버를 한 줄 명령으로 깔게 해주는 마켓플레이스 | 설치·관리 매니저 |
| **소켓 (Socket)** | CC와 Brave MCP 사이를 잇는 통신 통로 | 통신 파이프라인 |

> 💡 AI 생태계에서는 이 통신 규격을 **MCP(Model Context Protocol)** 라고 부른다. 결국 프로그램끼리 터미널 내부에서 데이터를 주고받는 소켓 통신을 의미한다.

#### ✅ 작동 시나리오 — "오늘 실시간 검색어 1위가 뭐야?"

| 순서 | 일어나는 일 |
|---|---|
| 1 | **CC**가 "최신 정보가 필요하다"고 판단 |
| 2 | **소켓**을 통해 Smithery가 설치해 둔 **Brave Search** MCP를 깨움 |
| 3 | **Brave Search**가 실시간으로 인터넷 결과를 수집 |
| 4 | 검색 결과가 다시 **소켓**을 타고 CC에 전달 |
| 5 | CC가 결과를 조합해 터미널에 답변 |

#### ✅ 설치 흐름 (개요)

```bash
# Smithery로 Brave Search MCP 추가 (예시 명령)
smithery mcp add brave
```

> ⚠️ 실제 명령·패키지명은 Smithery 사이트의 최신 안내를 따를 것. 마켓플레이스 항목은 갱신될 수 있다.

#### ✅ 한 줄 정리

- **Smithery.ai** = 도구를 쉽게 설치해 주는 도구 (마켓플레이스)
- **Brave Search** = 도구의 알맹이 (검색 엔진)
- **소켓** = 내 터미널의 CC와 그 알맹이를 이어주는 신경망

> ※ 실시간 웹 검색뿐 아니라 **외부 파일 연동, 이메일 조작, 캘린더, 디자인 툴 연결** 같은 기능을 CC에게 붙이려 할 때 이 세트 조합을 반복해서 만나게 된다. 4단계의 `papyrus`·`hermes`도 같은 구조다.

### 3-6. 플러그인 (Plugins)

**플러그인 = 스킬 + 훅 + MCP를 하나로 묶은 패키지.**

| 개별 설치 | 플러그인 |
|---|---|
| 스킬 파일 따로 만들고 | 설치 한 번이면 |
| 훅 따로 설정하고 | 스킬 + 훅 + MCP가 |
| MCP 따로 연결하고 | 한꺼번에 추가됨 |

**마켓플레이스에서 설치:**
```bash
# 마켓플레이스 등록
claude plugin marketplace add 소유자/이름

# 플러그인 설치
claude plugin install 플러그인이름

# 설치된 플러그인 목록
claude plugin list

# 플러그인 제거
claude plugin uninstall 플러그인이름
```

> ⚠️ 같은 플러그인이 local scope와 user scope에 동시 등록될 수 있다. `claude plugin list`에서 같은 이름이 두 번 보이면, `uninstall`을 두 번 실행해야 양쪽 다 제거된다.

> ※ "나도 이런 걸 만들 수 있다" — 4단계에서 실제로 만든 플러그인(wj-magic)을 써보게 된다. 지금은 "이런 게 있구나"를 아는 것이 목표.

---

## 4. 실전 도구 + 자동화 — 돌려놓고 자리 비우기

3단계에서 CC 확장의 **개념**(스킬, 훅, MCP, 플러그인)을 배웠다면, 여기서는 그것들이 **실제로 조합된 도구**를 쓰고, 자동화 워크플로우를 구성한다.

### 4-1. 고도화 워크플로우

#### 하네스 (Harness)

**하네스 = CC의 행동을 자동으로 제어하는 설정 체계.** `settings.json`에 훅/규칙을 설정하면 CC가 매번 같은 방식으로 동작하게 만들 수 있다.

- 비유: CC에게 "매뉴얼"을 주는 것. 매번 말로 설명하지 않아도 규칙대로 움직임
- 3단계에서 배운 훅이 여기서 실전으로 쓰임

#### 랄프 루프 (Ralph Loop)

**"기획 → CC가 구현 → 자동 검증 → 다시 기획" 사이클을 CC가 알아서 반복한다.** 사람이 방향만 잡아주면 CC가 계속 돌아감.

- 비유: 사람이 "이런 앱 만들어"라고 지시하면, CC가 PM이 되어 알아서 팀을 꾸리고 구현하고 검수하고 다음 작업으로 넘어감
- 사람은 결과만 확인하면 됨

#### /loop + 스케줄링

CC가 주기적으로 작업을 반복하게 만드는 것.

| 방식 | 설명 | 예시 |
|---|---|---|
| `/loop` | CC가 일정 간격으로 같은 작업 반복 | "5분마다 빌드 상태 체크해줘" |
| cron 스케줄링 | 특정 시간에 CC가 자동 실행 | "매일 아침 9시에 이슈 정리해줘" |
| tmux + loop | 터미널 안 꺼지게 + 루프 조합 | 자면서도 CC가 일하는 환경 |

> ※ 이 세 가지를 조합하면: **자리를 비울 때 CC에게 작업을 시키고, 다음 날 돌아오면 결과가 나와 있는 환경**이 만들어진다.

### 4-2. wj-magic

CC를 PM(프로젝트 매니저)으로 만들어서, 에이전트 팀을 조직하고 자동으로 개발을 돌리는 올인원 플러그인.

**전체 구성:**

| 구성요소 | 수량 | 역할 |
|---|---|---|
| 커맨드 | 5개 | 루프 제어, 검증, 초기화 |
| 스킬 | 13개 | 기획→설계→구현→검수 전 과정 |
| 에이전트 | 13개 | 도메인별 전문가 (프론트/백엔드/QA/보안 등) |
| 훅 | 7개 | 자동 품질 게이트 (위험 명령 차단, 민감 파일 보호, 코드 품질 체크) |

**핵심 커맨드:**

| 커맨드 | 역할 |
|---|---|
| `/wj:init` | 프로젝트 초기 구조 생성 (`docs/`, `.dev/`, `CLAUDE.md`) |
| `/wj:loop plan` | 요구사항 → PRD + 태스크 목록 자동 생성 |
| `/wj:loop start` | 자율 개발 루프 시작 |
| `/wj:verify` | 전체 빌드 + 테스트 최종 검증 |
| `/wj:check` | 코드베이스 품질 전수 점검 |

**주요 스킬:**

| 스킬 | 하는 일 |
|---|---|
| `/wj:brainstorm` | 막연한 아이디어를 1:1 대화로 정제 → 설계 문서 |
| `/wj:plan` | 설계를 파일 단위 구현 계획으로 분해 |
| `/wj:devrule` | 코드 구현 (규모에 따라 직접 또는 에이전트 투입) |
| `/wj:design` | 새 UI를 비주얼 방향부터 구현까지 |
| `/wj:polish` | 기존 UI 진단 → 처방 → 개선 |
| `/wj:investigate` | 버그를 5개 에이전트가 병렬 조사 → 근본 원인 규명 |
| `/wj:team` | 전문가 에이전트 팀을 선별해 병렬 투입 |
| `/wj:cto-review` | 코드베이스 전수 점검 + 자동 리팩토링 |
| `/wj:ideation` | PM/UX/사업/마케팅/데이터 5명이 병렬 리서치 |
| `/wj:commit` | 한글 커밋 메시지 자동 작성 |

**에이전트 팀 구성:**

| 역할 | 에이전트 | 담당 |
|---|---|---|
| 구현 | frontend-dev, backend-dev, engine-dev, design-dev | 각 도메인별 코드 작성 |
| 검수 | qa-reviewer, design-reviewer, security-auditor, test-engineer | 코드/디자인/보안/테스트 리뷰 |
| 분석 | code-analyst, perf-analyst, regression-hunter, web-researcher | 이슈 조사 전문 |
| 운영 | docs-keeper | 문서 동기화, 교훈 기록 |

**자동 품질 게이트:**
- 위험한 명령어 자동 차단 (`rm -rf`, `sudo`, force push 등)
- 민감 파일 보호 (`.env`, `.pem` 등 수정 방지)
- 코드 작성 후 자동 품질 체크 (파일 크기, 타입 안전성, 에러 처리)
- 턴 종료 시 L1(문법) → L2(타입) → L3(테스트) 3단계 검증

#### wj-magic 설치

```bash
# 1. 마켓플레이스 등록 (먼저!)
claude plugin marketplace add brandonnoh/woojoo-magic

# 2. 플러그인 설치
claude plugin install wj-magic
```

> ⚠️ 마켓플레이스 등록을 먼저 해야 plugin install이 동작한다. 순서가 바뀌면 "플러그인을 찾을 수 없습니다" 에러.

#### 실습 시나리오

**① 프로젝트 초기화:**
```bash
/wj:init --with-prd
```
→ `docs/`, `.dev/`, `CLAUDE.md` 구조가 자동 생성

> ⚠️ `/wj:init`은 프로젝트당 한 번만 실행. 이미 구조가 있는 프로젝트에서 다시 실행하면 기존 파일을 덮어쓸 수 있음.

**② 아이디어 → 설계:**
```bash
/wj:brainstorm
```
→ "할 일 관리 앱 만들고 싶어" 같은 막연한 아이디어로 시작하면 CC가 1:1 대화로 질문하면서 설계 문서를 만들어줌

**③ 자율 루프로 앱 만들기:**
```bash
/wj:loop plan
/wj:loop start
```
→ CC가 PM이 되어 에이전트를 투입하고 자동으로 앱을 만드는 과정 관찰
→ 품질 게이트가 자동으로 동작하는 걸 확인
→ 태스크가 하나씩 완료되며 커밋이 쌓이는 흐름 체감

> ⚠️ 루프가 돌아가는 동안 CC가 여러 에이전트를 동시에 실행할 수 있다. 컴퓨터가 느려질 수 있으니 다른 무거운 프로그램은 닫아두는 게 좋다.
> ⚠️ 루프 중에는 CC가 "계속할까요?"라고 묻지 않고 알아서 진행한다. 멈추고 싶으면 `/wj:loop stop`을 입력.

### 4-3. papyrus — 문서/리포트 자동 생성 MCP

마크다운을 작성하면 브랜드 A4 HTML 보고서로 변환해준다.

| 템플릿 | 용도 |
|---|---|
| 요약보고서 | 경영진 대상 핵심 사안 요약 |
| 회의록 | 회의 내용, 결정 사항, 액션 아이템 |
| 제안서 | 새 프로젝트 승인 요청 |
| 회고 | 프로젝트 사후 분석 |
| 업무보고 | 주간/월간 현황 공유 |
| 커스텀 | 자유 형식 |

**설치:**
```bash
claude mcp add papyrus -- npx -y @anthropic-ai/papyrus-mcp
```

**실습:** CC에게 "주간보고 만들어줘" → 템플릿 선택 → 마크다운 작성 → HTML 보고서 생성

### 4-4. hermes — Gmail AI 분류 MCP

Gmail을 AI로 분류/관리하는 MCP. 메일함을 스캔하고 규칙을 설정하면 자동으로 분류/삭제해준다.

| 기능 | 설명 |
|---|---|
| 메일함 스캔 | 발신자 패턴을 분석해서 마케팅/도구알림/업무 메일 자동 분류 |
| 규칙 설정 | "이 발신자는 삭제", "이건 업무 라벨" 같은 규칙 |
| 자동 분류 | 신규 메일이 들어오면 규칙에 따라 자동 처리 |
| 검색 | CC에게 "지난주 김팀장한테 온 메일 찾아줘" 같은 자연어 검색 |

> ❌ `hermes_delete(permanent=True)`는 복구 불가. 삭제 규칙을 설정할 때는 반드시 `dry_run=True`로 미리보기한 후 실행.

**설치:**
```bash
claude mcp add hermes -- npx -y @anthropic-ai/hermes-mcp
hermes_init
```

> ℹ️ 위 패키지 경로는 작성 시점 기준. 실제 패키지명이 다를 수 있으니 공식 안내를 따를 것.

### 4-5. 자동화 워크플로우 — 돌려놓고 자리 비우기

모든 도구를 조합해서 만드는 최종 환경:

| 요소 | 역할 |
|---|---|
| tmux | 터미널 안 꺼지게 |
| 하네스 | CC 행동 규칙 설정 |
| `/wj:loop` | 자율 개발 루프 |
| 스케줄링 | 매일 아침 자동 실행 |

**전체 흐름:**
```bash
tmux new -s overnight
cc
/wj:loop start
# Ctrl+B 다음 D 누르고 자리 비움
# 다음 날 돌아오면 앱이 만들어져 있다
```

---

## 부록 A. 도구별 "왜 깔았는지" 요약 카드

| 도구 | 무엇 | 왜 필요한가 | 안 깔면 |
|---|---|---|---|
| **WSL** | 윈도우용 리눅스 환경 | 이 워크북에서 다루는 도구가 리눅스 기반 | (윈도우만) 실습 진행 불가 |
| **Homebrew** | 맥용 패키지 매니저 | 모든 도구를 한 줄로 설치 | 도구마다 수동 설치 노가다 |
| **fnm** | Node.js 버전 관리 | 프로젝트마다 다른 Node 버전 전환 | Node 버전 충돌 시 해결 어려움 |
| **Claude Code** | CC 본체 | 워크북의 중심 도구 | 실습 진행 불가 |
| **Git** | 버전 관리 | 코드의 세이브포인트, CC가 잘못 고쳐도 되돌리기 | 망가지면 복구 불가 |
| **GitHub Desktop** | 깃의 GUI | 시각적으로 커밋 관리 | 깃 명령어 수동으로 다 쳐야 함 |
| **gh CLI** | GitHub 명령어 도구 | 터미널에서 PR/이슈 관리 | 매번 브라우저로 GitHub 가야 함 |
| **Python** | 파이썬 런타임 | 일부 MCP/스크립트가 파이썬 기반 | 일부 도구 동작 안 함 |
| **tmux** | 터미널 세션 유지 | 터미널 닫아도 CC가 계속 돌아감 | 노트북 닫으면 CC 즉시 종료 |
| **Cursor** | CC 친화적 코드 편집기 | 코드 보면서 CC 쓰는 게 더 편함 | 메모장으로 코드 보는 격 |
| **Amphetamine** (Mac) | 슬립 방지 | 노트북 덮개 닫아도 CC 계속 | 절전 모드로 작업 중단 |
| **Termius** | 모바일 SSH 클라이언트 | 외부에서 폰으로 메인 PC 접속 | 메인 PC 앞에서만 작업 가능 |
| **Tailscale** | 보안 메시 VPN | 어디서든 안전하게 메인 PC 연결 | 외부 접속 보안 위험 |
| **Vercel** | 무료 호스팅 | 내 앱을 인터넷에 공개 | 만든 앱을 남에게 보여줄 수 없음 |
| **Firebase Firestore** | 클라우드 DB | 데이터를 서버에 영구 저장 | 브라우저 닫으면 데이터 사라짐 |
| **Firebase Auth** | 로그인 시스템 | 사용자 인증 | 누구나 데이터 봄 (보안 사고) |
| **MCP 서버들** (papyrus/hermes 등) | CC 외부 도구 | CC에게 새 능력 부여 | CC가 못 하는 일이 많음 |
| **wj-magic 플러그인** | 올인원 자동화 | 자율 개발 루프 + 에이전트 팀 | 모든 걸 수동으로 지시해야 함 |

---

## 부록 B. 앞으로 활용하기 좋은 시나리오

### 디자이너 업무 자동화

| 상황 | 활용 |
|---|---|
| 새 디자인 시안에 대한 코드 변환 | `/wj:design` 으로 비주얼 방향부터 React 컴포넌트까지 |
| 기존 UI 다듬기 | `/wj:polish` 로 진단 → 처방 → 개선 |
| 주간 디자인 작업 보고 | papyrus의 `업무보고` 템플릿 |
| 디자인 회의록 정리 | papyrus의 `회의록` 템플릿 |
| 막연한 제품 아이디어를 구체화 | `/wj:brainstorm` |
| 디자인 시스템 컴포넌트 점검 | `/wj:cto-review` |

### 운영 / 관리

| 상황 | 활용 |
|---|---|
| Gmail 마케팅 메일 자동 분류 | hermes 규칙 설정 |
| 매일 아침 어제 작업 요약 자동 생성 | cron + `/wj:check` 조합 |
| 외부에서 메인 PC 작업 상황 확인 | Termius + Tailscale로 폰에서 접속 |
| 긴 보고서를 3줄 요약 | 커스텀 `/요약` 스킬 |

### 학습 / 실험

| 상황 | 활용 |
|---|---|
| 새로운 라이브러리 학습 | CC에게 "X 라이브러리로 간단한 예제 만들어줘" |
| 모르는 코드 이해 | `/wj:explain` (wj-magic 스킬) |
| 버그 원인 모르겠을 때 | `/wj:investigate` (5개 에이전트 병렬 조사) |
| 코드베이스 품질 점검 | `/wj:check` |

### 자동화 시나리오

> 패턴: **자리 비우기 전 시키고 → 자는 동안 CC가 일하고 → 돌아오면 결과 확인**

1. **밤사이 기능 한 개 만들기**:
   ```
   tmux new -s overnight
   cc
   /wj:loop plan   # 요구사항 입력
   /wj:loop start  # 자율 루프 시작
   # 자리 비움
   ```

2. **매일 아침 자동 리포트**:
   ```
   cron: 매일 08:00 → CC가 "어제 GitHub 활동 + 받은 메일 요약" → papyrus로 일일보고 생성 → 이메일 발송
   ```

3. **외출 중 진행 상황 체크**:
   ```
   Termius (스마트폰) → Tailscale 통해 메인 PC 접속 → tmux attach → CC 진행 상황 확인
   ```
