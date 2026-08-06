# Git 치트시트

- 작성일: 2026-08-04
- 이 프로젝트(lgoneid) 작업 패턴 기준. PowerShell / Windows 환경.
- 위험한 명령은 ⚠️ 로 표시했다.

---

## 1. 매일 쓰는 것

```bash
git status                  # 변경 상태 (자세히)
git status --short          # 한 줄 요약 (= git status -s)
git status -sb              # 요약 + 브랜치/원격 대비(ahead/behind)

git diff                    # 아직 add 안 한 변경
git diff --staged           # add 한 변경 (= --cached)
git diff --stat             # 파일별 변경량만
git diff <파일>             # 특정 파일만

git add <파일>              # 스테이징
git add .                   # 전부 (권장하지 않음 - 로컬설정까지 들어감)
git commit                  # 에디터로 메시지 작성
git commit -m "메시지"      # 한 줄 메시지

git log --oneline -5        # 최근 5개
git push                    # 원격 반영
```

**변경 파일만 빠르게 보기** (로컬설정 제외)
```bash
git status --porcelain | grep -vE 'application-local|secret|logback|build.gradle|.gitignore'
```

---

## 2. 일부 파일만 커밋 (이 프로젝트의 기본 패턴)

로컬설정·테스트용 파일이 항상 섞여 있으므로 **`git add .` 대신 파일을 지정**한다.

```bash
# 1) 원하는 파일만 담기
git add user-web-service/src/main/java/com/lgoneid/userweb/login/controller/LoginJspController.java

# 2) 담긴 것 확인 (M 이 앞칸에 있으면 스테이징됨)
git status --short

# 3) 커밋
git commit
```

**스테이징 상태 읽는 법** (`git status --short`)
| 표시 | 뜻 |
|--|--|
| `M ` (앞칸) | 스테이징됨 — 커밋에 포함 |
| ` M` (뒷칸) | 스테이징 안 됨 — 커밋에 미포함 |
| `MM` | 일부만 스테이징 |
| `??` | 새 파일(추적 안 됨) |

**커밋 직전 최종 확인**
```bash
git diff --staged --stat        # 이번 커밋에 들어갈 파일·변경량
```

---

## 3. 되돌리기

### 커밋을 취소하고 싶다 (푸시 전)
```bash
git reset --soft HEAD~1     # 커밋만 취소, 변경은 스테이징 상태로 남음  ← 가장 안전
git reset HEAD~1            # 커밋 취소 + 스테이징 해제 (내용 유지, 기본값 --mixed)
git reset --hard HEAD~1     # ⚠️ 커밋 취소 + 변경 내용까지 삭제 (복구 어려움)
```
> 커밋 파일을 잘못 골랐을 때: `git reset --soft HEAD~1` → `git reset` → 원하는 파일만 `git add` → 다시 커밋

### 스테이징만 취소 (내용은 유지)
```bash
git reset                   # 전부 스테이징 해제
git restore --staged <파일>  # 특정 파일만 해제
```

### 파일 변경을 버리고 싶다
```bash
git restore <파일>          # ⚠️ 수정 내용 삭제 (= git checkout -- <파일>)
git restore .               # ⚠️ 전체 되돌리기
```

### 이미 푸시한 커밋
```bash
git revert <커밋해시>       # 되돌리는 새 커밋 생성 ← 협업 시 이 방법
# ⚠️ push --force 는 남이 받은 이력을 깨뜨린다. 공유 브랜치에서 쓰지 말 것
```

### 커밋 메시지만 고치기
```bash
git commit --amend          # ⚠️ 푸시 전에만. 푸시 후엔 이력이 바뀜
```

---

## 4. 조사·추적 (원인 찾을 때 유용)

```bash
git log --oneline -10                       # 간단 목록
git log --format='%h %ad %s' --date=short   # 해시/날짜/제목
git log --stat -1                           # 마지막 커밋의 변경 파일
git show <해시>                             # 커밋 전체 내용
git show --stat <해시>                      # 파일 목록만
git show <해시> -- <파일>                   # 그 커밋의 특정 파일 diff

git log --grep='IT200054A2-9732'            # 커밋 메시지로 검색
git log -S'retrieveMemberLoginAccListByCi'  # 그 문자열이 추가·삭제된 커밋 찾기 ← 강력
git log -p -- <파일>                        # 파일의 변경 이력 + diff
git log --follow -- <파일>                  # 파일명이 바뀌어도 추적

git blame <파일>                            # 줄별 마지막 수정자·커밋
git blame -L 100,120 <파일>                 # 특정 줄 범위만
```

**`git log -S` 활용 예** — "이 코드가 언제·어느 이슈에서 들어왔나"
```bash
git log --oneline -S'LIST_APPTEST_SVC_ID' -- affiliates-service/src/main
```

---

## 5. 브랜치

```bash
git branch                          # 로컬 브랜치 목록
git branch -a                       # 원격 포함
git switch <브랜치>                 # 이동 (= git checkout <브랜치>)
git switch -c <새브랜치>            # 생성 + 이동
git switch -c <새브랜치> origin/dev # 원격 기준으로 생성

git fetch                           # 원격 정보만 가져오기 (병합 안 함)
git pull                            # fetch + merge
git pull --rebase                   # fetch + rebase (이력 깔끔)

git branch -d <브랜치>              # 병합된 브랜치 삭제
git branch -D <브랜치>              # ⚠️ 강제 삭제
```

**현재 브랜치와 원격 차이 확인**
```bash
git status -sb                      # [ahead 6] 처럼 표시
git log origin/dev..HEAD --oneline  # 원격에 없는 내 커밋
```

---

## 6. 임시 저장 (stash)

작업 중인데 다른 브랜치로 가야 할 때.
```bash
git stash                   # 변경 임시 저장 (스테이징 안 된 것 포함)
git stash -u                # 새 파일(untracked)까지 포함
git stash list              # 목록
git stash pop               # 가장 최근 것 복원 + 목록에서 제거
git stash apply             # 복원하되 목록에 남김
git stash drop              # 목록에서 제거
```

---

## 7. 이 프로젝트 관례

### 커밋 메시지
```
<이슈번호> <이슈명>

- 실제 작업 내용을 bullet 로
- 변경 범위·주의사항도 명시 (예: src/main 미수정)
```
- 제목은 **Jira 제목 그대로**(영역 태그 포함): `IT200054A2-9732 [UserWeb] 소셜 로그인 ...`
- 중요한 변경이면 괄호로 덧붙임: `... (CI 수집로그, 세션만료 방어)`

### 커밋에서 항상 제외하는 것
| 파일 | 이유 |
|--|--|
| `application-local.yml` | 로컬 설정 |
| `secret/secret-local.yml` | 로컬 시크릿 |
| `logback-spring.xml` | 로컬 로그 레벨 |
| `build.gradle`, `.gitignore` | 로컬 실험 |
| `views/sample/AffTestView.jsp` | 테스트용 화면 |

### 커밋 전
```powershell
.\gradlew.bat :<모듈>:spotlessCheck --console=plain   # 포맷 위반 확인 (권장)
.\gradlew.bat :<모듈>:spotlessApply --console=plain   # 적용 (내 파일만 걸리는지 확인 후)
```
> `spotless` 에 `ratchetFrom` 설정이 없어 **모듈 전체**를 포맷한다. `spotlessCheck` 로 먼저 대상을 확인하고, 무관한 파일까지 바뀌면 별도 커밋으로 분리할 것. `**/*.md`, `**/*.yml` 도 대상이라는 점 주의.

---

## 8. 상황별 대처

| 상황 | 대처 |
|--|--|
| 커밋 파일을 잘못 골랐다 (푸시 전) | `git reset --soft HEAD~1` → 다시 add → 커밋 |
| 커밋 메시지 오타 (푸시 전) | `git commit --amend` |
| 이미 푸시했는데 되돌려야 한다 | `git revert <해시>` (force push 금지) |
| 수정을 다 버리고 원본으로 | `git restore <파일>` ⚠️ |
| 실수로 `reset --hard` 했다 | `git reflog` 로 이전 HEAD 찾아 `git reset --hard <해시>` (커밋된 것만 복구 가능) |
| 남의 변경이 내 파일에 묻어있다 | `git diff <파일>` 로 확인 → 내 것 아니면 `git restore <파일>` |
| 브랜치를 잘못 만들었다 | `git switch <원래브랜치>` → `git branch -d <잘못만든것>` |
| 충돌(conflict) 났다 | 파일에서 `<<<<<<<` 마커 정리 → `git add <파일>` → `git commit` (rebase 중이면 `git rebase --continue`) |
| 뭐가 뭔지 모르겠다 | `git status` 가 대부분 다음 할 일을 알려준다 |

### `git reflog` — 최후의 안전망
```bash
git reflog                      # HEAD 가 움직인 모든 이력
git reset --hard HEAD@{2}       # 2단계 전 상태로
```
커밋된 내용은 `reflog` 로 대개 복구된다. **커밋 안 한 변경은 복구 불가** → 중요한 작업은 자주 커밋하거나 `stash`.

---

## 9. 알아두면 좋은 개념

| 용어 | 뜻 |
|--|--|
| **워킹트리** | 지금 편집 중인 파일들 |
| **스테이징(index)** | `git add` 로 담아둔, 다음 커밋에 들어갈 목록 |
| **HEAD** | 현재 체크아웃된 커밋 (보통 브랜치의 최신) |
| `HEAD~1` | HEAD 의 부모 커밋 (한 단계 전) |
| **origin** | 기본 원격 저장소 이름 |
| **ahead / behind** | 원격 대비 내 브랜치가 앞선/뒤처진 커밋 수 |
| **tracked / untracked** | git 이 추적 중인 파일 / 아직 추가 안 된 새 파일 |

세 영역의 이동:
```
워킹트리  --git add-->  스테이징  --git commit-->  로컬 저장소  --git push-->  원격
   <--git restore--        <--git reset--            <--git reset HEAD~1--
```
