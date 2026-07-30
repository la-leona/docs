# IDE 검색/단축키 정리 (IntelliJ & VSCode, Windows)

## 1. IntelliJ - Ctrl+F 로 "있는" 메서드명을 못 찾을 때

Ctrl+F = **현재 파일 내 찾기**. 못 찾는 흔한 이유:

### (1) Find 바 옵션 토글 (가장 흔함)
입력창 오른쪽 아이콘 확인:
- **In Selection** — 켜지면 **선택 영역 안**에서만 검색. (이름을 드래그 선택한 상태면 그 밖은 검색 안 됨) → 끄기
- **Match Case (Cc)** — 대소문자 구분
- **Words (W)** — 단어 단위(부분일치 안 됨)
- **Regex (.\*)** — 정규식 모드(특수문자 해석)

→ 이 4개를 다 끄고 재검색.

### (2) Ctrl+F 는 현재 파일만
- 그 메서드가 **다른 파일**에 있으면 Ctrl+F로는 못 찾음. 탭 포커스 확인.

### (3) 검색어에 앞뒤 공백/이전 잔재
- 입력창 비우고 다시 타이핑.

### 메서드 찾을 때 더 확실한 방법
| 단축키 | 기능 |
|--------|------|
| Ctrl+Shift+F | 프로젝트 전체(파일들) 텍스트 검색 |
| 더블 Shift | Search Everywhere (클래스/파일/심볼/텍스트 통합) |
| Ctrl+Alt+Shift+N | Go to Symbol (메서드명으로 정의 점프) ← 딱 이 용도 |
| Ctrl+F12 | File Structure (현재 파일 메서드 목록 필터) |
| Alt+F7 | Find Usages (사용처 찾기) |
| Ctrl+B | 선언으로 이동 |

=> 결론: 대개 **In Selection 토글**이 원인. 메서드 점프는 **Ctrl+Alt+Shift+N**.


## 2. VSCode - F1 검색창에 ">" 가 붙어있는 이유

같은 입력창을 **명령 팔레트**와 **빠른 열기(파일 검색)**가 공유. 앞의 `>` = 명령 모드 표시.

### 접두어별 모드
| 접두어 | 모드 |
|--------|------|
| `>` | 명령 실행 (Command Palette) |
| (없음) | **파일 이름 검색 (Quick Open)** |
| `@` | 현재 파일 내 심볼 |
| `#` | 워크스페이스 심볼 |
| `:` | 특정 줄로 이동 |
| `?` | 모드 도움말 |

### 왜 ">" 로 열리나
- **F1 = Ctrl+Shift+P = 명령 팔레트** → 원래 항상 `>` 로 시작(바뀐 게 아님).
- **파일 검색은 Ctrl+P** → 접두어 없이 열림.
- 예전 기억은 아마 Ctrl+P 를 썼거나 팔레트가 직전 모드를 기억한 것.

### 해결
- 파일 찾을 땐 **Ctrl+P** (>` 없이 바로).
- F1으로 열렸으면 맨 앞 **`>` 한 글자만 삭제**.
- 굳이 F1으로 파일검색 원하면 Keyboard Shortcuts(Ctrl+K Ctrl+S)에서 `workbench.action.quickOpen` 을 F1에 재할당.


## 3. 알아두면 유용한 키바인딩

### IntelliJ (Windows)
| 단축키 | 기능 |
|--------|------|
| 더블 Shift | Search Everywhere (전부 통합 검색) |
| Ctrl+N | 클래스 찾기 |
| Ctrl+Shift+N | 파일 찾기 |
| Ctrl+Alt+Shift+N | 심볼(메서드/필드) 찾기 |
| Ctrl+F / Ctrl+R | 현재 파일 찾기 / 바꾸기 |
| Ctrl+Shift+F / Ctrl+Shift+R | 프로젝트 전체 찾기 / 바꾸기 |
| Ctrl+B (또는 Ctrl+클릭) | 선언/정의로 이동 |
| Ctrl+Alt+B | 구현(implementation)으로 이동 |
| Alt+F7 | 사용처 찾기(Find Usages) |
| Ctrl+F12 | 파일 구조(메서드 목록) |
| Ctrl+E | 최근 파일 |
| Ctrl+Shift+Backspace | 마지막 편집 위치로 |
| Alt+Left / Alt+Right | 이전/다음 탐색 위치 |
| Ctrl+/ , Ctrl+Shift+/ | 라인 주석 / 블록 주석 |
| Ctrl+Alt+L | 코드 포맷 |
| Shift+F6 | 이름 변경(Rename, 안전 리팩터) |
| Ctrl+P | 메서드 파라미터 힌트 |
| Ctrl+W / Ctrl+Shift+W | 선택 범위 확장 / 축소 |
| Ctrl+D / Ctrl+Y | 라인 복제 / 삭제 |

### VSCode (Windows)
| 단축키 | 기능 |
|--------|------|
| Ctrl+P | 파일 빠른 열기 (파일명 검색) |
| Ctrl+Shift+P / F1 | 명령 팔레트 (`>`) |
| Ctrl+T | 워크스페이스 심볼 검색 |
| Ctrl+Shift+O | 현재 파일 심볼(@) 로 이동 |
| Ctrl+G | 특정 줄로 이동 |
| Ctrl+F / Ctrl+H | 현재 파일 찾기 / 바꾸기 |
| Ctrl+Shift+F / Ctrl+Shift+H | 전체 찾기 / 바꾸기 |
| F12 / Alt+F12 | 정의로 이동 / 정의 미리보기(Peek) |
| Shift+F12 | 참조(사용처) 찾기 |
| Alt+Left / Alt+Right | 이전/다음 위치 |
| Ctrl+B | 사이드바 토글 |
| Ctrl+` | 통합 터미널 토글 |
| Ctrl+/ , Shift+Alt+A | 라인 주석 / 블록 주석 |
| Shift+Alt+F | 코드 포맷 |
| F2 | 이름 변경(Rename) |
| Ctrl+D | 다음 동일 단어 선택(멀티커서) |
| Alt+Click | 커서 추가(멀티커서) |
| Ctrl+Shift+K / Alt+Up,Down | 라인 삭제 / 이동 |

### 공통 팁
- "정의로 점프"는 IntelliJ **Ctrl+B**, VSCode **F12**.
- "사용처 찾기"는 IntelliJ **Alt+F7**, VSCode **Shift+F12**.
- "전체 검색"은 IntelliJ **Ctrl+Shift+F**, VSCode **Ctrl+Shift+F** (동일).
- 텍스트로 못 찾겠으면 **심볼 검색**(IntelliJ Ctrl+Alt+Shift+N / VSCode Ctrl+T)이 훨씬 정확.


## 4. 디버깅 단축키

### IntelliJ (Windows)
| 단축키 | 기능 |
|--------|------|
| Ctrl+F8 | 브레이크포인트 토글 |
| Ctrl+Shift+F8 | 브레이크포인트 목록/조건 설정 |
| Shift+F9 | 디버그 실행 |
| Shift+F10 | 그냥 실행(Run) |
| F8 | Step Over (다음 줄) |
| F7 | Step Into (메서드 안으로) |
| Shift+F7 | Smart Step Into (호출 중 선택) |
| Shift+F8 | Step Out (메서드 밖으로) |
| Alt+F9 | Run to Cursor (커서까지 실행) |
| F9 | Resume (다음 브레이크포인트까지) |
| Alt+F8 | Evaluate Expression (식 평가) |
| Ctrl+F2 | 실행/디버그 중지(Stop) |
| Alt+클릭(변수) 또는 Quick Evaluate | 값 즉시 확인 |

### VSCode (Windows)
| 단축키 | 기능 |
|--------|------|
| F9 | 브레이크포인트 토글 |
| F5 | 디버그 시작 / 계속(Resume) |
| Ctrl+F5 | 디버그 없이 실행 |
| Shift+F5 | 디버그 중지 |
| Ctrl+Shift+F5 | 디버그 재시작 |
| F10 | Step Over |
| F11 | Step Into |
| Shift+F11 | Step Out |
| Ctrl+Shift+D | 디버그 뷰 열기 |
| Ctrl+K Ctrl+I | 호버로 값 보기(또는 마우스 오버) |
| (디버그 콘솔) Ctrl+Shift+Y | 디버그 콘솔에서 식 평가 |

공통: Step Over(한 줄 넘기기)는 IntelliJ **F8** / VSCode **F10**, Step Into는 IntelliJ **F7** / VSCode **F11**.


## 5. Git 단축키

### IntelliJ (Windows)
| 단축키 | 기능 |
|--------|------|
| Ctrl+K | 커밋 (Commit) |
| Ctrl+Shift+K | 푸시 (Push) |
| Ctrl+T | 업데이트/풀 (Update Project) |
| Alt+9 | Git 도구창 열기 |
| Alt+\` | VCS Operations 팝업(퀵 메뉴) |
| Ctrl+Alt+Z | 변경 되돌리기(Rollback/Revert) |
| Ctrl+D | (변경 파일 선택 후) Diff 보기 |
| Annotate(거터 우클릭) | Git Blame(라인별 작성자) |
| Ctrl+Shift+Alt+ (Show History) | 파일/선택 히스토리 (우클릭 Git > Show History) |

팁: `Alt+\`` (VCS Operations)에서 branch/stash/revert 등 대부분 접근 가능.

### VSCode (Windows)
| 단축키 | 기능 |
|--------|------|
| Ctrl+Shift+G | 소스 컨트롤(Git) 뷰 열기 |
| (SC뷰) Ctrl+Enter | 커밋 |
| Ctrl+Shift+P > "Git: Push/Pull/..." | 명령 팔레트로 Git 작업 |
| Ctrl+Shift+P > "Git: Checkout to" | 브랜치 전환 |
| 거터/우클릭 > Open Changes | 변경 Diff |
| GitLens 확장 | 라인 Blame/히스토리 강화(별도 설치) |

팁: VSCode Git은 전용 단축키가 적어 **소스컨트롤 뷰(Ctrl+Shift+G)** + **명령 팔레트("Git:")** 조합이 기본. Blame/히스토리는 **GitLens** 확장 추천.

### 공통 팁 (Git)
- 커밋: IntelliJ **Ctrl+K**, VSCode 소스컨트롤 뷰 **Ctrl+Enter**.
- 푸시: IntelliJ **Ctrl+Shift+K**, VSCode 명령 팔레트 "Git: Push".
- Blame(작성자 추적): IntelliJ 거터 우클릭 **Annotate**, VSCode **GitLens**.
- IntelliJ가 Git 통합 단축키는 더 풍부, VSCode는 확장(GitLens)으로 보완.
