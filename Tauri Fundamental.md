# Tauri Fundamental

Kimi: **Tauri**는 **Rust + 웹 기술(HTML/CSS/JS)**로 데스크톱 앱을 만드는 프레임워크입니다.  
Electron처럼 웹으로 UI를 만들지만, 백엔드는 Rust라서 **가볍고 빠릅니다** (Electron보다 실행 파일이 1/100 수준으로 작아요).

---

## 1. 사전 준비 (Windows 기준)

```powershell
# 1. Rust 설치 (MSYS2 불필요)
winget install Rustlang.Rustup

# 2. Node.js 설치 (프론트엔드 빌드용)
winget install OpenJS.NodeJS

# 3. Visual Studio Build Tools (Rust 컴파일에 필요)
winget install Microsoft.VisualStudio.2022.BuildTools
# 설치 후 "Desktop development with C++" 워크로드 추가 필요
```

> **재부팅 후** 터미널을 새로 열어야 `cargo`, `node`, `npm` 명령어가 통합니다.

---

## 2. 프로젝트 생성

```powershell
# create-tauri-app으로 보일러플레이트 생성
cargo install create-tauri-app
cargo create-tauri-app
```

실행하면 대화형으로 물어봅니다:

```
Project name: my-app
Choose your UI template: vanilla / vue / react / svelte ...
Choose your UI flavor: ts / js
```

완료하면 폴더가 생깁니다.

---

## 3. 폴더 구조 이해

```
my-app/
├── src/                  ← 프론트엔드 (HTML/JS/CSS)
│   ├── index.html
│   └── main.js
├── src-tauri/            ← Rust 백엔드
│   ├── Cargo.toml        ← Rust 의존성
│   ├── tauri.conf.json   ← Tauri 설정
│   └── src/
│       └── main.rs       ← 진입점 (Rust 코드)
└── package.json
```

- **`src/`** = 웹 UI (브라우저에서 보는 화면 그대로)
- **`src-tauri/src/main.rs`** = Rust로 짜는 "진짜" 데스크톱 로직 (파일 접근, 시스템 명령, 트레이 아이콘 등)

---

## 4. 개발 서버 실행

```powershell
cd my-app
cargo tauri dev
```

- 프론트엔드 개발 서버 + Rust 백엔드가 같이 뜹니다
- **Hot Reload** 지원: HTML/JS 수정하면 창이 자동으로 새로고침됨

---

## 5. PowerShell 스크립트와 연결하기 (사용자 맥락)

Tauri의 강점은 **Rust가 시스템 명령을 자유롭게 실행**할 수 있다는 점입니다.

### Rust에서 PowerShell 실행

`src-tauri/src/main.rs`에 다음처럼 작성:

```rust
use std::process::Command;
use tauri::command;

#[command]
fn run_cleanup() -> Result<String, String> {
    let output = Command::new("powershell")
        .args(&[
            "-ExecutionPolicy", "Bypass",
            "-File", "C:\\Scripts\\clear-browser-and-windows-cache-v4.ps1",
            "-Force", "-Quiet", "-LogPath", "C:\\logs\\clean.log"
        ])
        .output()
        .map_err(|e| e.to_string())?;

    let stdout = String::from_utf8_lossy(&output.stdout);
    Ok(stdout.to_string())
}

fn main() {
    tauri::Builder::default()
        .invoke_handler(tauri::generate_handler![run_cleanup])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

### 프론트엔드에서 호출

`src/main.js`:

```javascript
import { invoke } from '@tauri-apps/api/core';

async function onCleanClick() {
    const result = await invoke('run_cleanup');
    document.getElementById('log').textContent = result;
}
```

**요약:** 버튼 클릭 → JavaScript가 Rust 함수 호출 → Rust가 PowerShell 실행 → 결과를 UI에 표시

---

## 6. 빌드 (.exe 만들기)

```powershell
cargo tauri build
```

출력:
```
src-tauri/target/release/my-app.exe     ← 단일 실행 파일 (약 5~15MB)
src-tauri/target/release/bundle/msi/    ← 설치 프로그램 (.msi)
```

**Electron 대비 파일 크기:**
- Electron: 100~300MB
- Tauri: **5~15MB** (Rust + OS 웹뷰 사용)

---

## 7. Tauri vs WPF vs Electron

| | Tauri | WPF | Electron |
|---|---|---|---|
| **언어** | Rust + JS | C# + XAML | JS (Node.js) |
| **파일 크기** | **~10MB** | ~100KB (단, .NET 필요) | ~150MB |
| **메모리** | **매우 적음** | 보통 | 많음 |
| **UI** | 웹 기술 (자유로움) | XAML (Windows 전용) | 웹 기술 |
| **크로스플랫폼** | Windows/macOS/Linux | Windows 전용 | Windows/macOS/Linux |
| **PowerShell 연동** | Rust의 `Command`로 실행 | 직접 내장 (같은 .NET) | Node.js `child_process` |

---

## 요약

> **Tauri = "웹으로 UI 그리고, Rust로 시스템 명령 실행"**

1. `cargo create-tauri-app`으로 생성
2. `src/`에서 HTML/CSS/JS로 화면 만들기
3. `src-tauri/src/main.rs`에서 `#[command]` 함수로 PowerShell/시스템 명령 실행
4. `cargo tauri build`로 10MB 수준의 .exe 생성

**PowerShell 스크립트(v4)를 Tauri로 감싸면**, 콘솔 없이 깔끔한 UI + 진행률 바 + 로그 출력창을 가진 **현대적 Windows 도구**로 만들 수 있습니다.

Tauri로 PowerShell 래퍼 UI를 실제로 만들어드릴까요? (예: "청소 시작" 버튼 + 로그 텍스트박스 + 결과 요약 창)
