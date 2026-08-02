# ps-cache-cleaner-tauri

Kimi: Tauri로 PowerShell v4 스크립트를 감싸는 **완전한 GUI 프로젝트**를 만들었습니다.

**기능:**
- **스크립트 경로** 입력
- **모든 v4 파라미터** 체크박스로 ON/OFF
- **실시간 로그** 출력 (PowerShell stdout/stderr를 스트리바�)
- **실행 결과** 표시
- **다크 테마** UI

---

## 프로젝트 구조

```
ps-cache-cleaner/
├── src/
│   ├── index.html
│   ├── main.js
│   └── styles.css
├── src-tauri/
│   ├── Cargo.toml
│   ├── tauri.conf.json
│   └── src/
│       └── main.rs
├── package.json
└── README.md
```

---

## 설치 & 실행 방법

```powershell
# 1. Tauri 프로젝트 생성 (vanilla + vite)
cargo install create-tauri-app
cargo create-tauri-app --template vanilla

# 2. 생성된 프로젝트 폴더로 이동
cd ps-cache-cleaner

# 3. 아래 제공한 파일들로 덮어쓰기
#    (src/*, src-tauri/*, package.json, Cargo.toml)

# 4. 의존성 설치
npm install

# 5. 개발 모드 실행
cargo tauri dev

# 6. 빌드 (.exe 생성)
cargo tauri build
# 결과: src-tauri/target/release/ps-cache-cleaner.exe
```

---

## 소스 코드

아래 파일들을 복사해서 사용하세요.

### `src-tauri/Cargo.toml`
```toml
[package]
name = "ps-cache-cleaner"
version = "4.0.0"
description = "PowerShell Cache Cleaner GUI"
authors = ["you"]
edition = "2021"

[build-dependencies]
tauri-build = { version = "2", features = [] }

[dependencies]
tauri = { version = "2", features = [] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
tokio = { version = "1", features = ["process", "io-util", "rt-multi-thread"] }

[features]
custom-protocol = ["tauri/custom-protocol"]
```

### `src-tauri/tauri.conf.json`
```json
{
  "$schema": "https://schema.tauri.app/config/2",
  "productName": "PS Cache Cleaner",
  "version": "4.0.0",
  "identifier": "com.example.ps-cache-cleaner",
  "build": {
    "frontendDist": "../dist",
    "devUrl": "http://localhost:1420"
  },
  "app": {
    "windows": [
      {
        "title": "PS Cache Cleaner",
        "width": 950,
        "height": 780,
        "resizable": true,
        "center": true
      }
    ],
    "security": {
      "csp": null
    }
  },
  "bundle": {
    "active": true,
    "targets": "all",
    "icon": [
      "icons/32x32.png",
      "icons/128x128.png",
      "icons/128x128@2x.png",
      "icons/icon.icns",
      "icons/icon.ico"
    ]
  }
}
```

### `src-tauri/src/main.rs`
```rust
use serde::Deserialize;
use std::process::Stdio;
use tauri::Emitter;
use tokio::io::{AsyncBufReadExt, BufReader};
use tokio::process::Command;

#[derive(Deserialize, Debug)]
#[serde(rename_all = "camelCase")]
struct CleanupParams {
    script_path: String,
    preview: bool,
    force: bool,
    clear_browser_cache: bool,
    force_close_browsers: bool,
    deep_windows_cache: bool,
    include_windows_update_cache: bool,
    empty_recycle_bin: bool,
    cleanup_component_store: bool,
    run_disk_cleanup: bool,
    rebuild_explorer_cache: bool,
    clear_delivery_optimization_cache: bool,
    clear_user_traces: bool,
    older_than_days: i32,
}

#[tauri::command]
async fn run_cleanup(params: CleanupParams, app: tauri::AppHandle) -> Result<String, String> {
    let mut args = vec![
        "-NoProfile".to_string(),
        "-ExecutionPolicy".to_string(),
        "Bypass".to_string(),
        "-File".to_string(),
        params.script_path,
    ];

    if params.preview { args.push("-Preview".to_string()); }
    if params.force { args.push("-Force".to_string()); }
    if params.clear_browser_cache { args.push("-ClearBrowserCache".to_string()); }
    if params.force_close_browsers { args.push("-ForceCloseBrowsers".to_string()); }
    if params.deep_windows_cache { args.push("-DeepWindowsCache".to_string()); }
    if params.include_windows_update_cache { args.push("-IncludeWindowsUpdateCache".to_string()); }
    if params.empty_recycle_bin { args.push("-EmptyRecycleBin".to_string()); }
    if params.cleanup_component_store { args.push("-CleanupComponentStore".to_string()); }
    if params.run_disk_cleanup { args.push("-RunDiskCleanup".to_string()); }
    if params.rebuild_explorer_cache { args.push("-RebuildExplorerCache".to_string()); }
    if params.clear_delivery_optimization_cache { args.push("-ClearDeliveryOptimizationCache".to_string()); }
    if params.clear_user_traces { args.push("-ClearUserTraces".to_string()); }
    
    if params.older_than_days > 0 {
        args.push("-OlderThanDays".to_string());
        args.push(params.older_than_days.to_string());
    }

    let _ = app.emit("log-line", format!("> Starting: powershell {:?}", args));

    let mut child = Command::new("powershell")
        .args(&args)
        .stdout(Stdio::piped())
        .stderr(Stdio::piped())
        .spawn()
        .map_err(|e| format!("Failed to start PowerShell: {}", e))?;

    let stdout = child.stdout.take().ok_or("Failed to capture stdout")?;
    let stderr = child.stderr.take().ok_or("Failed to capture stderr")?;

    let app_stdout = app.clone();
    tokio::spawn(async move {
        let reader = BufReader::new(stdout);
        let mut lines = reader.lines();
        while let Ok(Some(line)) = lines.next_line().await {
            let _ = app_stdout.emit("log-line", line);
        }
    });

    let app_stderr = app.clone();
    tokio::spawn(async move {
        let reader = BufReader::new(stderr);
        let mut lines = reader.lines();
        while let Ok(Some(line)) = lines.next_line().await {
            let _ = app_stderr.emit("log-line", format!("[STDERR] {}", line));
        }
    });

    let status = child.wait().await.map_err(|e| e.to_string())?;
    
    if status.success() {
        Ok("✅ Cleanup completed successfully".to_string())
    } else {
        Err(format!("❌ PowerShell exited with code: {:?}", status.code()))
    }
}

fn main() {
    tauri::Builder::default()
        .invoke_handler(tauri::generate_handler![run_cleanup])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

### `package.json`
```json
{
  "name": "ps-cache-cleaner",
  "private": true,
  "version": "4.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "tauri": "tauri"
  },
  "dependencies": {
    "@tauri-apps/api": "^2.0.0",
    "@tauri-apps/cli": "^2.0.0"
  },
  "devDependencies": {
    "vite": "^5.0.0"
  }
}
```

### `src/index.html`
```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>PS Cache Cleaner</title>
  <link rel="stylesheet" href="./styles.css" />
</head>
<body>
  <div class="container">
    <header>
      <h1>🧹 PS Cache Cleaner</h1>
      <p>PowerShell Cache Cleaner v4 — Tauri GUI</p>
    </header>

    <section class="panel">
      <label class="field-label">PowerShell Script Path</label>
      <input type="text" id="script-path" value="C:\opt\ps1\clear-browser-and-windows-cache-v4.ps1" />
    </section>

    <section class="panel">
      <h2>⚙️ Parameters</h2>
      <div class="grid">
        <label class="chk"><input type="checkbox" id="chk-preview" /> <span>Preview (dry run, no delete)</span></label>
        <label class="chk"><input type="checkbox" id="chk-force" /> <span>Force (skip confirmation)</span></label>
        <label class="chk"><input type="checkbox" id="chk-browser" /> <span>Clear Browser Cache</span></label>
        <label class="chk"><input type="checkbox" id="chk-force-browser" /> <span>Force Close Browsers</span></label>
        <label class="chk"><input type="checkbox" id="chk-deep" /> <span>Deep Windows Cache</span></label>
        <label class="chk"><input type="checkbox" id="chk-wu" /> <span>Include Windows Update Cache</span></label>
        <label class="chk"><input type="checkbox" id="chk-bin" /> <span>Empty Recycle Bin</span></label>
        <label class="chk"><input type="checkbox" id="chk-dism" /> <span>DISM Component Store</span></label>
        <label class="chk"><input type="checkbox" id="chk-cleanmgr" /> <span>Run Disk Cleanup (cleanmgr)</span></label>
        <label class="chk"><input type="checkbox" id="chk-explorer" /> <span>Rebuild Explorer Cache</span></label>
        <label class="chk"><input type="checkbox" id="chk-do" /> <span>Clear Delivery Optimization</span></label>
        <label class="chk"><input type="checkbox" id="chk-traces" /> <span>Clear User Traces (recent, jump lists)</span></label>
      </div>
      <div class="row-inline">
        <label>Older Than <input type="number" id="older-days" value="0" min="0" /> days</label>
      </div>
    </section>

    <section class="panel actions">
      <button id="start-btn">▶ Start Cleanup</button>
      <div id="result-area"></div>
    </section>

    <section class="panel log-panel">
      <h2>📋 Live Log</h2>
      <textarea id="log-area" readonly placeholder="PowerShell output will appear here..."></textarea>
    </section>
  </div>

  <script type="module" src="./main.js"></script>
</body>
</html>
```

### `src/main.js`
```javascript
import { invoke } from '@tauri-apps/api/core';
import { listen } from '@tauri-apps/api/event';

const logArea = document.getElementById('log-area');
const resultArea = document.getElementById('result-area');
const startBtn = document.getElementById('start-btn');

// 실시간 로그 수신
listen('log-line', (event) => {
  logArea.value += event.payload + '\n';
  logArea.scrollTop = logArea.scrollHeight;
});

startBtn.addEventListener('click', async () => {
  logArea.value = '';
  resultArea.textContent = '';
  startBtn.disabled = true;
  startBtn.textContent = '⏳ Running...';

  const params = {
    scriptPath: document.getElementById('script-path').value,
    preview: document.getElementById('chk-preview').checked,
    force: document.getElementById('chk-force').checked,
    clearBrowserCache: document.getElementById('chk-browser').checked,
    forceCloseBrowsers: document.getElementById('chk-force-browser').checked,
    deepWindowsCache: document.getElementById('chk-deep').checked,
    includeWindowsUpdateCache: document.getElementById('chk-wu').checked,
    emptyRecycleBin: document.getElementById('chk-bin').checked,
    cleanupComponentStore: document.getElementById('chk-dism').checked,
    runDiskCleanup: document.getElementById('chk-cleanmgr').checked,
    rebuildExplorerCache: document.getElementById('chk-explorer').checked,
    clearDeliveryOptimizationCache: document.getElementById('chk-do').checked,
    clearUserTraces: document.getElementById('chk-traces').checked,
    olderThanDays: parseInt(document.getElementById('older-days').value) || 0,
  };

  try {
    const result = await invoke('run_cleanup', { params });
    resultArea.textContent = result;
    resultArea.className = 'success';
  } catch (e) {
    resultArea.textContent = String(e);
    resultArea.className = 'error';
  } finally {
    startBtn.disabled = false;
    startBtn.textContent = '▶ Start Cleanup';
  }
});
```

### `src/styles.css`
```css
* { box-sizing: border-box; margin: 0; padding: 0; }

body {
  background: #0b1623;
  color: #dce4ec;
  font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
  font-size: 14px;
  line-height: 1.5;
  padding: 16px;
}

.container { max-width: 920px; margin: 0 auto; }

header { text-align: center; margin-bottom: 20px; }
header h1 {
  color: #5391fe;
  font-size: 1.6rem;
  font-weight: 700;
  letter-spacing: -0.5px;
}
header p { color: #7a95b5; font-size: 0.85rem; margin-top: 4px; }

.panel {
  background: #132238;
  border: 1px solid #1e3a5f;
  border-radius: 10px;
  padding: 16px;
  margin-bottom: 14px;
}
.panel h2 {
  color: #5391fe;
  font-size: 1rem;
  font-weight: 600;
  margin-bottom: 12px;
  display: flex;
  align-items: center;
  gap: 6px;
}

.field-label {
  display: block;
  font-size: 0.8rem;
  font-weight: 600;
  color: #8baac7;
  margin-bottom: 6px;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

input[type="text"] {
  width: 100%;
  padding: 10px 12px;
  background: #0b1623;
  border: 1px solid #2a4a70;
  border-radius: 6px;
  color: #e0e8f0;
  font-size: 0.9rem;
  font-family: Consolas, monospace;
}
input[type="text"]:focus {
  outline: none;
  border-color: #5391fe;
}

.grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 6px;
}
label.chk {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 10px;
  border-radius: 6px;
  cursor: pointer;
  transition: background 0.15s;
  font-size: 0.9rem;
}
label.chk:hover { background: #1b3050; }
input[type="checkbox"] {
  width: 16px;
  height: 16px;
  accent-color: #5391fe;
  cursor: pointer;
}

.row-inline {
  margin-top: 12px;
  padding-top: 10px;
  border-top: 1px solid #1e3a5f;
  color: #8baac7;
  font-size: 0.9rem;
}
input[type="number"] {
  width: 64px;
  padding: 6px 8px;
  background: #0b1623;
  border: 1px solid #2a4a70;
  border-radius: 4px;
  color: #e0e8f0;
  margin-left: 6px;
  text-align: center;
}

.actions {
  display: flex;
  align-items: center;
  gap: 16px;
  flex-wrap: wrap;
}

#start-btn {
  background: #2e7d32;
  color: #fff;
  border: none;
  padding: 12px 28px;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 700;
  cursor: pointer;
  transition: filter 0.2s, transform 0.1s;
  box-shadow: 0 2px 8px rgba(46, 125, 50, 0.25);
}
#start-btn:hover { filter: brightness(1.1); }
#start-btn:active { transform: translateY(1px); }
#start-btn:disabled {
  background: #2a3f5f;
  color: #6b8cae;
  cursor: not-allowed;
  box-shadow: none;
}

#result-area {
  font-weight: 700;
  font-size: 0.95rem;
  min-height: 24px;
}
#result-area.success { color: #27c93f; }
#result-area.error { color: #ff5f56; }

.log-panel { display: flex; flex-direction: column; }
#log-area {
  width: 100%;
  height: 320px;
  background: #070f18;
  border: 1px solid #1e3a5f;
  border-radius: 8px;
  color: #b8cce0;
  font-family: Consolas, 'Courier New', monospace;
  font-size: 0.8rem;
  line-height: 1.4;
  padding: 12px;
  resize: vertical;
  white-space: pre-wrap;
}
#log-area::placeholder { color: #3d5a80; }
```

---

## 동작 설명

| 구성 요소 | 역할 |
|---|---|
| **Rust (`main.rs`)** | PowerShell 프로세스를 `tokio::process::Command`로 실행. stdout/stderr를 라인 단위로 읽어 `log-line` 이벤트로 프론트에 전송 |
| **JavaScript (`main.js`)** | `invoke('run_cleanup')`로 Rust 함수 호출. `listen('log-line')`으로 실시간 로그 수신 |
| **UI (`index.html` + CSS)** | 파라미터 체크박스, 스크립트 경로, 로그 뷰어, 실행 버튼 |

**빌드 결과:** `src-tauri/target/release/`에 **~10MB 단일 exe**가 생성됩니다.

---

**참고:** `cargo tauri build` 시 아이콘 파일이 필요합니다. `src-tauri/icons/` 폴더에 기본 아이콘을 넣거나, 이전에 만든 SVG를 PNG로 변환해서 넣으세요.

파일을 압축해서 다운로드 링크를 드릴까요? 아니면 위 코드를 복사해서 쓰시면 됩니다.




Kimi: 압축 완료했습니다!

**다운로드:**  
[ps-cache-cleaner-tauri.zip](sandbox:///mnt/agents/output/ps-cache-cleaner-tauri.zip)

**압축 내용 (7개 파일):**
- `src-tauri/Cargo.toml` — Rust 프로젝트 설정
- `src-tauri/tauri.conf.json` — Tauri 앱 설정
- `src-tauri/src/main.rs` — Rust 백엔드 (PowerShell 실행 + 실시간 로그)
- `package.json` — Node.js 의존성
- `src/index.html` — UI 마크업
- `src/main.js` — 프론트엔드 로직
- `src/styles.css` — 다크 테마 스타일

**사용 방법:**
1. 압축 풀기
2. `npm install` (의존성 설치)
3. `cargo tauri dev` (개발 모드 실행)
4. `cargo tauri build` (exe 빌드)

빌드 전에 `src-tauri/icons/` 폴더에 아이콘 파일(`.ico`, `.png`)을 넣어주세요. Tauri 기본 아이콘 생성 명령어: `cargo tauri icon path/to/icon.png`
