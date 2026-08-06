# Gradle task 정리 (build / compileJava / compileTestJava, clean, --console=plain)

로그 추가 같은 간단 수정 후 확인할 때 어떤 task 를 쓸지 정리.

## 1. 세 task 차이

| task | 컴파일 대상 | 추가로 하는 일 | 속도 |
|------|-----------|--------------|------|
| `compileJava` | `src/main/java` 만 | 없음 | 가장 빠름 |
| `compileTestJava` | `src/test/java` (+ 자동으로 main 먼저) | 없음 | 중간 |
| `build` | main + test 전부 | spotless(포맷)·`test` 실행·jacoco·bootJar 조립·check | 가장 느림 |

의존 체인:
- `compileTestJava` dependsOn `compileJava` (테스트가 main 클래스를 참조 -> main 먼저 컴파일)
- `build` -> `check` -> `test` + `spotlessCheck` + `assemble(bootJar)` ... 전부 끌고 옴

## 2. "로그 추가 같은 간단 수정 확인" 이면?

**`compileJava` 만으로 충분** — "문법 오류 없이 컴파일되는가" 를 가장 빠르게 확인.
```
.\gradlew.bat :user-web-service:compileJava --console=plain
```

주의 3가지:
1. **컴파일 != 실행.** compileJava 는 코드를 돌리지 않으므로 **로그가 실제 찍히는지는 검증 안 됨.**
   -> 앱을 띄워 화면으로 확인해야 함.
2. **메서드 시그니처를 건드렸다면** `compileTestJava` 까지. 테스트가 그 메서드를 참조하면 main 만 컴파일해선 테스트 깨짐을 못 잡음.
   ```
   .\gradlew.bat :user-web-service:compileTestJava --console=plain
   ```
   (이거 하나로 main+test 둘 다 컴파일 확인됨)
3. **커밋 직전**엔 `build`(또는 최소 `spotlessApply`). build 가 spotless·test·jacoco 수행 -> 포맷 안 맞으면 커밋 후 spotlessCheck 에서 걸림.

정리:
- 컴파일만 확인 -> `compileJava` (제일 빠름)
- 시그니처 변경/테스트 참조 있음 -> `compileTestJava`
- 로그가 실제 찍히는지 -> 앱 실행 (컴파일로는 불가)
- 커밋 준비 -> `build`

## 2-1. ⚠️ 정적 리소스(js/css)·SQL 매퍼는 `compileJava` 로 반영되지 않는다

**`compileJava` 는 `.java` 만 처리한다.** `src/main/resources` 아래(정적 js/css, SQL 매퍼 xml, yml)는 **`processResources`** 가 별도로 `build/resources/main` 으로 복사한다. 서버는 **classpath = `build/resources/main`** 을 읽으므로, `src` 만 고치면 **서버는 옛 파일을 계속 서빙**한다.

```
src/main/resources/static/js/...      ← 편집하는 곳
        ↓  processResources
build/resources/main/static/js/...    ← 서버가 실제 서빙하는 곳
```

| 고친 것 | 필요한 task | 재기동 |
|--|--|--|
| `src/main/java/**.java` | `compileJava` | 필요 |
| **`src/main/resources/static/**.js`, `.css`** | **`processResources`** | 보통 불필요(정적 파일) |
| **`src/main/resources/sqlmap/**.xml`** (MyBatis) | **`processResources`** | 필요 |
| `src/main/resources/*.yml` | `processResources` | 필요 |
| `src/main/webapp/**.jsp` | 없음(웹앱 경로) | 보통 자동 반영 |

```powershell
.\gradlew.bat :user-web-service:processResources --console=plain
```

### 증상으로 알아채기
- 파일을 고쳤는데 **브라우저에서 옛 내용**이 보인다 → 캐시로 오해하기 쉽다
- **시크릿모드·하드리로드(Ctrl+Shift+R)에서도 옛 내용**이면 캐시가 아니라 **이 문제**다
- 확인법: 서빙 URL 을 직접 열어본다 → `http://localhost:8030/userweb/js/terms/TermsListV4.js`
- 또는 두 파일을 비교한다
  ```powershell
  Select-String -Path 'user-web-service\src\main\resources\static\js\terms\TermsListV4.js'   -Pattern '찾는코드'
  Select-String -Path 'user-web-service\build\resources\main\static\js\terms\TermsListV4.js' -Pattern '찾는코드'
  ```
  → build 쪽에 없으면 `processResources` 미실행

> 실제 사례(2026-08-05): 약관 화면 JS 를 고쳤는데 반영되지 않아 캐시로 의심했으나, `build/resources/main` 에 옛 파일(16:07)이 남아 있던 것이었다. `processResources` 후 즉시 해결.
> 덧붙여 **어느 파일이 실제로 로드되는지도 확인해야 한다** — 같은 화면에 `TermsList.js`/`TermsListV4.js` 처럼 버전이 여럿이면 컨트롤러가 반환하는 뷰(`setViewName`)를 따라가 확인할 것. Network 탭에 파일이 아예 안 잡히면 그 파일은 로드되지 않는 것이다.

## 3. compileJava 전에 clean 하는 게 좋을까? -> 대부분 아니오

- Gradle 은 **증분 컴파일**을 함. 바뀐 파일 + 영향 범위만 재컴파일 -> 로그 한 줄 수정이면 거의 즉시.
- `clean` 은 `build/` 전체 삭제 -> 매번 전체 재컴파일 -> 큰 멀티모듈에선 크게 느려짐. 증분 캐시 이득을 스스로 버리는 셈.

clean 이 필요한 경우 (예외):
1. 빌드가 이상하게 깨질 때(고쳤는데 옛 에러 남, stale 클래스 의심)
2. 파일 삭제/이름변경/패키지 이동 후 옛 `.class` 잔재
3. 의존성·버전 대폭 변경 후
4. 리포트 신뢰성 중요할 때 — jacoco 커버리지 (측정 전 `build/reports/jacoco` 삭제 규칙)
5. CI/릴리즈 빌드(재현성)

```
# 평소
.\gradlew.bat :user-web-service:compileJava --console=plain
# 이상할 때만
.\gradlew.bat :user-web-service:clean :user-web-service:compileJava --console=plain
```
참고: IntelliJ 로 재기동해 로그 확인할 거면 IntelliJ 자체 증분 빌드가 있어 gradlew compileJava 조차 매번 안 돌려도 되는 경우 많음. 결과가 꼬일 때만 clean.

## 4. --console=plain 이란?

Gradle 콘솔 **출력 형식**을 단순 텍스트로 바꾸는 옵션 (기능/빌드 결과엔 영향 없음).
- 기본(auto/rich): 애니메이션 진행바·컬러·실시간 갱신 상태라인(ANSI 제어문자).
- `--console=plain`: 그런 동적 표시 없이 **로그를 한 줄씩 평문**으로.

왜 쓰나 (특히 비대화형/자동화/로그캡처):
- rich 모드 진행바의 커서 제어문자가 파이프/로그파일/도구 실행 시 깨진 문자로 엉킴 -> plain 은 깨끗해 판독 쉬움.
- 재현성/로그 저장에 유리, 불필요한 갱신 억제.

관련 로그 상세도 옵션:
- `-q`(quiet): 경고/에러만
- `-i`(info), `-d`(debug): 더 자세히
- `-s`/`--stacktrace`: 실패 시 스택트레이스

즉 사람이 터미널에서 직접 볼 땐 굳이 안 붙여도 되고, 자동화/로그캡처엔 붙이는 게 좋음.
