# C++ UI 로직 분리 아키텍처

Kimi: Visual Studio로 **C++ Windows 앱**을 만들 때, 화면(UI)과 실제 일(시스템 작업)을 **명확히 분리**하는 구조를 잡는 게 중요합니다.

아래는 **Temp File Cleaner를 만들 때의 전형적인 3계층 구조**입니다.

---

## 1. 전체 구조도

```
┌──────────────────────────────────────┐
│  Presentation Layer (UI)             │  ← 사용자 눈에 보이는 부분
│  - 버튼, 체크박스, 진행률 바, 로그창 │
│  - MFC Dialog / WPF(C#) / WinUI 3    │
└──────────────┬───────────────────────┘
               │ 이벤트 전달 (버튼 클릭, 값 변경)
┌──────────────▼───────────────────────┐
│  Application Layer (Coordinator)     │  ← 앱의 두뇌
│  - 어떤 기능을 켤지 판단             │
│  - UI 업데이트 요청                  │
│  - 작업 스케줄링 (스레드 분리)       │
└──────────────┬───────────────────────┘
               │ "Temp 폴더 정리해줘" 명령
┌──────────────▼───────────────────────┐
│  System/Native Layer (C++ 핵심)      │  ← C++가 진짜 하는 일
│  - 파일 삭제 (Win32 API)             │
│  - 프로세스 종료                     │
│  - 서비스 제어                       │
│  - 권한 확인 (관리자 여부)           │
│  - 디스크 용량 계산                  │
└──────────────────────────────────────┘
```

---

## 2. 각 계층의 구체적인 역할

### Layer 1: Presentation (UI) — "사용자와 대화"

| 기술 | 특징 | C++ 관여도 |
|---|---|---|
| **MFC** | C++로 UI를 직접 그림. 다이얼로그 리소스 + `CDialog` 클래스 | C++가 UI까지 직접 담당 |
| **WPF (C#)** | XAML로 화면 선언. C#이 이벤트 처리 | C++는 **별도 DLL**로 분리 |
| **WinUI 3 / UWP** | 최신 Windows UI | C++로 XAML 코드비하인드 작성 가능 |
| **Win32 API** | `CreateWindowEx`로 창을 코드로 생성 | C++가 모든 것을 직접 |

> **핵심:** UI는 "눈에 보이는 것"만 담당합니다. **파일 하나도 직접 삭제하지 않습니다.** 버튼 클릭 → "삭제해줘" 신호만 아래로 던집니다.

### Layer 2: Application — "작업 지휘관"

이 계층은 보통 **C++ 클래스**로 만듭니다.

```cpp
// CleanerEngine.h
class CleanerEngine {
public:
    void SetOptions(bool clearBrowser, bool deepClean, int olderThanDays);
    bool StartCleaning();           // 전체 작업 시작
    void Cancel();                  // 취소 요청
    std::vector<LogEntry> GetLogs(); // 실시간 로그 수집
    
private:
    CleanerOptions m_options;
    std::atomic<bool> m_cancelled;  // 스레드 안전한 취소 플래그
};
```

**하는 일:**
- UI에서 넘어온 "청소 시작" 신호를 받음
- 어떤 모듈을 실행할지 순서대로 배치 (`TempCleaner` → `BrowserCleaner` → `DOCacheCleaner`)
- **백그라운드 스레드**에서 작업을 돌림 (UI가 멈추지 않게)
- 진행 상황을 UI에 다시 전달 (`PostMessage`, 콜백, 또는 C#의 `Progress<T>`)

### Layer 3: System/Native — "C++가 진짜 하는 일"

이 부분이 **C++의 전유물**입니다. PowerShell 스크립트의 각 기능이 C++에서는 다음처럼 바뀝니다.

| PowerShell 기능 | C++ 구현 방식 | 사용 API/라이브러리 |
|---|---|---|
| `Remove-Item` (파일 삭제) | `DeleteFileW`, `RemoveDirectoryW` | Win32 API |
| `Get-ChildItem -Recurse` | `FindFirstFileExW` 반복 | Win32 API |
| `Test-Path` | `GetFileAttributesW` | Win32 API |
| `Stop-Process` | `OpenProcess` → `TerminateProcess` | Win32 API |
| `Get-Service` / `Stop-Service` | `OpenSCManager` → `ControlService` | Win32 API |
| `Test-IsAdministrator` | `CheckTokenMembership` 또는 `IsUserAnAdmin` | Win32 API |
| `Get-Date`, 날짜 비교 | `GetFileTime` → `FileTimeToSystemTime` | Win32 API |
| `Format-Size` (용량 포맷) | 직접 계산 | — |
| `dism.exe` 실행 | `CreateProcessW`로 외부 프로세스 실행 | Win32 API |
| `Start-Transcript` (로그) | `std::ofstream` 또는 Windows ETW | C++ STL / Win32 |

#### C++ 파일 삭제 예시
```cpp
#include <windows.h>
#include <string>

bool DeleteFileSafe(const std::wstring& path) {
    // 1. 파일 존재 확인
    DWORD attr = GetFileAttributesW(path.c_str());
    if (attr == INVALID_FILE_ATTRIBUTES) return false;
    
    // 2. 읽기전용 속성 제거 (삭제 가능하게)
    if (attr & FILE_ATTRIBUTE_READONLY) {
        SetFileAttributesW(path.c_str(), attr & ~FILE_ATTRIBUTE_READONLY);
    }
    
    // 3. 실제 삭제
    return DeleteFileW(path.c_str()) != 0;
}
```

#### C++ 폴더 순회 예시
```cpp
void CleanDirectory(const std::wstring& path, int olderThanDays) {
    WIN32_FIND_DATAW fd;
    HANDLE hFind = FindFirstFileExW((path + L"\\*").c_str(), 
                                     FindExInfoBasic, &fd, 
                                     FindExSearchNameMatch, nullptr, 0);
    if (hFind == INVALID_HANDLE_VALUE) return;
    
    do {
        std::wstring name = fd.cFileName;
        if (name == L"." || name == L"..") continue;
        
        std::wstring fullPath = path + L"\\" + name;
        
        if (fd.dwFileAttributes & FILE_ATTRIBUTE_DIRECTORY) {
            CleanDirectory(fullPath, olderThanDays);  // 재귀
            RemoveDirectoryW(fullPath.c_str());       // 빈 폴더 삭제
        } else {
            // 날짜 필터링 (olderThanDays)
            FILETIME ft = fd.ftLastWriteTime;
            if (IsOlderThan(ft, olderThanDays)) {
                DeleteFileSafe(fullPath);
            }
        }
    } while (FindNextFileW(hFind, &fd));
    
    FindClose(hFind);
}
```

---

## 3. UI와 C++를 연결하는 3가지 방식

Visual Studio에서 C++ 클리너를 만들 때, UI 기술에 따라 **연결 구조**가 달라집니다.

### 방식 A: 순수 C++ (MFC)
```
[MFC Dialog (C++)] ←──직접 호출──→ [C++ Cleaner Engine]
```
- UI도 C++, 로직도 C++
- 하나의 프로젝트 안에 전부 들어있음
- **장점:** 배포가 단순함 (exe 하나)
- **단점:** UI 변경이 번거로움, 현대적 디자인 어려움

### 방식 B: C# UI + C++ DLL
```
[WPF/WinForms (C#)] ←──P/Invoke──→ [CleanerCore.dll (C++)]
```
- **C#**이 화면과 버튼 이벤트를 담당
- **C++ DLL**이 실제 파일 삭제/프로세스 제어를 담당
- C#에서 `DllImport`로 C++ 함수를 불러옴

```csharp
// C# WPF 코드
[DllImport("CleanerCore.dll", CallingConvention = CallingConvention.Cdecl)]
public static extern bool StartCleanup(string configJson);

private void BtnStart_Click(object sender, RoutedEventArgs e) {
    StartCleanup("{\"deepClean\":true}");
}
```

```cpp
// CleanerCore.dll (C++)
extern "C" __declspec(dllexport) bool StartCleanup(const char* jsonConfig) {
    // 실제 삭제 로직
    return true;
}
```

- **장점:** UI는 현대적(C# WPF), 핵심 로직은 고성능(C++)
- **단점:** DLL 배포 관리 필요, C++/C# 간 문자열/구조체 변환 번거로움

### 방식 C: C++/CLI (중간 다리)
```
[WPF (C#)] ←──참조──→ [C++/CLI Wrapper] ←──네이티브──→ [C++ Core]
```
- C++/CLI가 Managed(C#)와 Unmanaged(C++) 사이의 **번역사** 역할
- 복잡하지만 대형 프로젝트에서 사용

---

## 4. 스레드 구조 (매우 중요)

파일 삭제는 **시간이 오래 걸립니다.** UI 스레드에서 하면 창이 **"응답 없음"**이 됩니다.

```
[Main UI Thread]          [Worker Thread (C++)]
     │                              │
     ├─ "Start" 클릭 ──────────────→│
     │                              ├─ 폴더 스캔 시작
     │←──── 진행률 업데이트 ────────┤ (PostMessage)
     │                              ├─ 파일 삭제 중...
     │←──── 로그 한 줄 추가 ────────┤
     │                              ├─ 완료
     │←──── "Done" 신호 ────────────┤
```

C++에서 스레드를 만드는 방법:

```cpp
#include <thread>
#include <atomic>

std::atomic<bool> g_running{false};

void RunCleaningAsync(HWND hWndNotify) {
    std::thread worker([hWndNotify]() {
        g_running = true;
        
        // 1. 스캔
        auto targets = ScanTargets();
        PostMessage(hWndNotify, WM_SCAN_DONE, targets.size(), 0);
        
        // 2. 삭제 (한 파일씩)
        for (size_t i = 0; i < targets.size() && g_running; ++i) {
            DeleteTarget(targets[i]);
            PostMessage(hWndNotify, WM_PROGRESS, i + 1, targets.size());
        }
        
        PostMessage(hWndNotify, WM_CLEANUP_DONE, 0, 0);
        g_running = false;
    });
    
    worker.detach(); // 백그라운드로 독립 실행
}
```

---

## 5. 요약: C++의 역할

| 역할 | 설명 |
|---|---|
| **파일 시스템 접근** | Win32 API로 직접 파일/폴더를 삭제, 속성 변경, 날짜 비교 |
| **프로세스/서비스 제어** | `TerminateProcess`, `ControlService`로 시스템 제어 |
| **권한 관리** | 관리자 권한 확인, UAC 요청 (manifest 설정), ACL 변경 |
| **성능 임계 작업** | 대용량 파일 스캔, 해시 계산, 빠른 순회 (STL보다 Win32 API가 빠른 경우) |
| **하드웨어/드라이버 급 접근** | 디스크 섹터 단위 작업, 저수준 I/O |
| **외부 프로그램 실행** | `CreateProcess`로 `dism.exe`, `cleanmgr` 실행 및 출력 캡처 |

> **한 줄 요약:** UI는 "사용자의 클릭을 받아서" C++에게 지시하고, **C++는 "Windows 커널/Win32 API를 통해 실제 파일과 프로세스를 만지는"** 역할을 합니다.

---

## 6. 실제 프로젝트 폴더 구조 예시

```
TempCleaner/
├── UI/                          ← C# WPF (또는 MFC)
│   ├── MainWindow.xaml
│   └── App.xaml
├── Core/                        ← C++ 정적 라이브러리 (.lib)
│   ├── FileCleaner.cpp          ← 파일 삭제 엔진
│   ├── FileCleaner.h
│   ├── ProcessManager.cpp       ← 브라우저 프로세스 종료
│   ├── ServiceManager.cpp       ← Windows 서비스 제어
│   ├── PrivilegeChecker.cpp     ← 관리자 권한 확인
│   └── DiskInfo.cpp             ← 여유 공간 계산
├── Bridge/                      ← C++/CLI (선택)
│   └── CleanerWrapper.cpp
└── TempCleaner.sln
```

이 구조로 가시면, **PowerShell 스크립트(v4)의 각 함수**가 C++의 `Core` 폴더 아래 클래스 메서드로 하나씩 대응됩니다.

특정 부분(예: "C# WPF에서 C++ DLL 부르는 법" 또는 "MFC로 UI 만드는 법")을 더 구체적으로 코드로 보여드릴까요?
