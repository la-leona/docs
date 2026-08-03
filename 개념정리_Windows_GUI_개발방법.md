# Windows GUI 개발 방법 정리

Windows 데스크톱 GUI를 만드는 방법을 계층별로 정리하고 장단점을 비교한 문서입니다.

> 용어 정리: **Visual C++ 는 UI 프레임워크가 아니라 컴파일러/툴체인**입니다.
> C++ 로 GUI 를 만들 때 실제 UI 는 Win32 API / MFC / Qt 같은 것을 쓰고,
> Visual C++(MSVC)는 그것을 빌드하는 도구입니다.

---

## 1. .NET 계열 (Microsoft 공식, 주로 C#)

| 프레임워크 | 장점 | 단점 |
|---|---|---|
| **WinForms** | 가장 배우기 쉬움, 드래그&드롭 디자이너, 내부 도구 제작 속도 최고, .NET 8/9 에서도 계속 지원 | 디자인이 구식, 고DPI/해상도 대응 약함, 복잡한 UI/애니메이션에 부적합, Windows 전용 |
| **WPF** | XAML + 데이터 바인딩/MVVM 강력, 벡터 기반(GPU 렌더링), 커스터마이징 자유도 높음, 성숙한 생태계와 풍부한 자료 | 학습 곡선(XAML·바인딩), 기본 외형이 낡음(WPF-UI/MahApps/HandyControl 등 테마로 해결), Windows 전용 |
| **WinUI 3** (Windows App SDK) | Microsoft 가 밀고 있는 현재 표준, Fluent 디자인 기본, 최신 Windows 기능 접근 | 생태계·자료가 아직 얇음, 패키징(MSIX)·배포가 번거로움, Windows 10 1809+ 필요 |
| **.NET MAUI** | 하나의 코드로 Windows/macOS/iOS/Android | 모바일 우선 설계라 데스크톱 전용 앱에는 과함, Windows 타깃은 내부적으로 WinUI 3 |
| **Avalonia** | WPF 와 유사한 XAML 이면서 진짜 크로스플랫폼(Win/macOS/Linux), 자체 렌더링으로 OS 간 외형 동일, 최근 인기 상승 | Microsoft 공식 아님, WPF 보다 자료 적음 |
| **UWP** | (참고용) | 신규 개발 비권장 — 사실상 WinUI 3 로 대체됨 |

---

## 2. 웹 기술로 감싸기

| 프레임워크 | 장점 | 단점 |
|---|---|---|
| **Electron** | 웹 개발 지식 그대로 활용, 생태계 최강(VS Code/Slack/Discord), 크로스플랫폼, 어디서나 동일 렌더링 | 용량 큼(설치본 100~200MB+), 메모리 사용 많음, 시작 속도 느림, Chromium 보안 업데이트 추적 필요 |
| **Tauri** | 매우 작음(수 MB), 메모리 적음, 백엔드는 Rust 로 안전·빠름, OS 내장 웹뷰 사용(Windows 는 WebView2 = Chromium 계열) | 백엔드 작업 시 Rust 학습 필요, OS 별 웹뷰 차이(macOS/Linux 는 렌더링 다름), 플러그인 생태계가 Electron 보다 얇음 |
| **Wails**(Go) / **Photino**(.NET) / **Neutralino**(JS) | Tauri 와 같은 OS 웹뷰 방식이며 백엔드 언어만 다름 | 커뮤니티 규모 작음 |
| **Blazor Hybrid** (.NET MAUI/WPF + WebView2) | C# 으로 웹 UI 작성, 기존 .NET 자산 재사용 | 웹뷰 오버헤드, 디버깅 경험이 다소 번거로움 |

> Windows 만 대상이면 Tauri 의 "OS 별 렌더링 차이" 단점이 사라져 Electron 대비 이득이 큽니다
> (WebView2 가 Chromium 기반이므로 렌더링이 사실상 동일).

---

## 3. C/C++ 네이티브

| 방식 | 장점 | 단점 |
|---|---|---|
| **Win32 API** (raw) | 런타임 의존성 없음, 최소 용량, 최고 성능/제어, 시스템 프로그래밍에 필수 | 생산성 최악(버튼 하나에 수십 줄), 현대적 UI 구현 매우 어려움 |
| **MFC** | Win32 를 C++ 로 감싼 고전, Visual Studio 에 아직 포함 | 사실상 레거시, 디자인 구식, 신규 프로젝트 비권장 |
| **Qt** | C++ 진영 최고 완성도, 크로스플랫폼, 위젯과 QML 두 방식, 도구(Qt Designer) 우수 | 덩치 큼, 라이선스 주의(LGPL 또는 상용), 배포 시 DLL 관리 필요 |
| **Dear ImGui** | 초경량, 즉시 모드(immediate mode), 개발자 도구/디버그 UI 에 최적 | 일반 사용자용 앱 UI 로는 부적합 |
| **wxWidgets / FLTK** | 네이티브 룩, 라이선스 자유로움 | 생태계·자료 적음 |

---

## 4. 스크립트 / RAD (빠른 내부 도구)

| 방식 | 장점 | 단점 |
|---|---|---|
| **PowerShell + WinForms/WPF** | 빌드 불필요, `.ps1` 하나로 배포, Windows 관리 도구에 최적 | UI 스레딩 취약(작업 중 창이 멈춤), 복잡해지면 유지보수 어려움, 실행 정책(ExecutionPolicy) 이슈 |
| **Python + PySide6/PyQt** | 개발 빠름, 데이터·자동화 연계 좋음 | 배포 번거로움(PyInstaller 등), 용량 큼, Qt 라이선스 |
| **Python + tkinter** | 표준 내장, 즉시 시작 가능 | 외형이 매우 투박 |
| **Delphi / Lazarus** | RAD 생산성, 단일 네이티브 exe, 빠름 | 생태계 축소(Lazarus 는 무료), 인력 확보 어려움 |
| **AutoHotkey** | 간단한 유틸/매크로 제작이 극단적으로 빠름 | 규모가 커지면 관리 불가 |

기타
- **Flutter** : Windows 데스크톱 지원. 커스텀 디자인에 강하지만 네이티브 느낌은 덜함.
- **JavaFX / Swing** : 기존 자바 자산이 있을 때 고려.

---

## 5. 상황별 추천

| 목표 | 추천 |
|---|---|
| Windows 전용 업무용 앱, 빠르게 만들기 | **WinForms** → 복잡해지면 **WPF** |
| Windows 전용, 제대로 만들기 | **WPF**(자료 많고 안정) 또는 **WinUI 3**(최신 디자인) |
| Windows + macOS/Linux 까지 | **Avalonia**(C#) 또는 **Tauri**(웹 기술) |
| 웹 프론트엔드에 익숙함 | **Tauri**(가벼움) / **Electron**(생태계·안정성) |
| 시스템 저수준, 의존성 없는 단일 exe | **Win32** 또는 **Qt** |
| PowerShell 스크립트에 GUI 얹기 | **PowerShell + WPF**(XAML 문자열 로드) |

---

## 6. 선택 시 함께 볼 것

- **배포 방식**: .NET 은 self-contained 단일 exe 가능(용량 증가), MSIX 패키징, 코드 서명 여부.
- **런타임 의존성**: .NET 런타임 설치 필요 여부, Electron/Tauri 는 WebView2 런타임(Windows 11 기본 포함).
- **라이선스**: Qt(LGPL/상용), Delphi(상용) 등은 배포 조건 확인 필요.
- **UI 스레딩**: 오래 걸리는 작업은 UI 스레드와 분리해야 창이 멈추지 않음
  (WPF/WinForms 는 async/await 또는 BackgroundWorker, PowerShell 은 Runspace).
- **고DPI 대응**: WinForms 는 별도 설정 필요, WPF/WinUI/Avalonia 는 기본 대응.
