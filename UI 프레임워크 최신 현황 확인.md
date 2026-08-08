## 📊 Windows GUI 개발 방법 비교표

| 방법 | 언어 | 렌더링 방식 | 난이도 | 성능 | 배포 크기 | 2026년 상태 |
|------|------|------------|--------|------|----------|------------|
| **Win32 API** | C/C++ | GDI / DirectX | ⭐⭐⭐⭐⭐ 매우 어려움 | ★★★★★ 최고 | ~100KB | 유지보수 |
| **MFC** | C++ | GDI | ⭐⭐⭐⭐⭐ 매우 어려움 | ★★★★☆ | ~100KB | 레거시 |
| **Windows Forms** | C#, VB.NET | GDI+ | ⭐⭐ 쉬움 | ★★★☆☆ | ~5MB | 유지보수 |
| **WPF** | C#, XAML | DirectX | ⭐⭐⭐ 보통 | ★★★★☆ | ~5-80MB | 유지보수+개선 |
| **WinUI 3** | C#, C++, XAML | 컴포지터 | ⭐⭐⭐ 보통 | ★★★★★ | ~15-90MB | **MS 주력** |
| **UWP** | C#, C++, XAML | 컴포지터 | ⭐⭐⭐⭐ 어려움 | ★★★★☆ | - | **사실상 단종** |
| **.NET MAUI** | C#, XAML | 네이티브 | ⭐⭐⭐ 보통 | ★★★☆☆ | - | 크로스플랫폼 |
| **Avalonia UI** | C#, XAML | Skia | ⭐⭐⭐ 보통 | ★★★★☆ | ~20MB | 크로스플랫폼 |
| **Qt** | C++, Python, QML | 네이티브 / OpenGL | ⭐⭐⭐⭐ 어려움 | ★★★★★ | ~20MB+ | 산업 표준 |
| **Flutter** | Dart | Skia/Impeller | ⭐⭐⭐ 보통 | ★★★★☆ | ~8-10MB | 성장 중 |
| **Electron** | JS/HTML/CSS | Chromium | ⭐⭐ 쉬움 | ★★☆☆☆ | ~100MB+ | 성숙 |
| **Tauri** | Rust + JS | WebView2 | ⭐⭐⭐ 보통 | ★★★★☆ | ~5MB | 부상 중 |
| **Python (PyQt/PySide)** | Python | Qt 기반 | ⭐⭐ 쉬움 | ★★★☆☆ | ~30MB+ | 내부 도구 |

---

## 🔍 상세 비교

### 1. 네이티브 Windows (Microsoft 공식)

#### **WinUI 3 / Windows App SDK** ⭐ 현재 MS의 주력
- **장점**: Windows 11 Fluent Design 네이티브 지원, Mica/Acrylic 효과, 컴포지터 기반 60fps 렌더링, Windows App SDK 1.8 안정화, MSIX 배포 통합, XAML Islands로 기존 앱 점진적 현대화 가능 
- **단점**: Windows 7/8 미지원, 제3자 컨트롤 생태계가 WPF보다 작음, 일부 틈새 컨트롤 부재
- **적합**: 새로운 Windows 전용 앱, Microsoft Store 배포, 현대적 UI가 필요한 엔터프라이즈 앱

#### **WPF (Windows Presentation Foundation)**
- **장점**: 20년 이상의 성숙한 생태계, 방대한 제3자 컨트롤, .NET 9에서 Fluent 테마 및 하드웨어 가속 개선, rock-solid한 Visual Studio 디자이너 
- **단점**: MS의 적극적 투자는 WinUI 3으로 이동, Native AOT 지원은 아직 제한적
- **적합**: 기존 WPF 앱 유지보수, 복잡한 데이터 그리드/리포팅이 필요한 엔터프라이즈 앱

#### **Windows Forms**
- **장점**: 가장 빠른 프로토타이핑, 드래그앤드롭 디자이너, 방대한 레거시 코드베이스
- **단점**: GDI+ 기반으로 고해상도/DPI 지원이 열악, 현대적 UI 구현 어려움, 유지보수 모드
- **적합**: 간단한 내부 도구, 레거시 시스템 유지보수

#### **UWP (Universal Windows Platform)**
- **장점**: 샌드박스 보안, MSIX 배포, Live Tile
- **단점**: **사실상 단종** — Microsoft가 더 이상 적극 개발하지 않음, Win32 API 접근 제한, Store 배포 강제 
- **적합**: 기존 UWP 앱 유지보수만 권장, 신규 개발은 WinUI 3로 마이그레이션

#### **Win32 API / MFC**
- **장점**: 최소한의 오버헤드, 완전한 시스템 제어, Windows 모든 버전 지원
- **단점**: 코드량이 매우 많음, 현대적 UI 구현에 엄청난 노력 필요, 개발자 풀 감소
- **적합**: 시스템 유틸리티, 드라이버 UI, 극한의 성능/크기 최적화가 필요한 경우

---

### 2. 크로스플랫폼 네이티브

#### **Qt**
- **장점**: C++ 기반 네이티브 성능, 30년 이상의 산업 표준, QML로 현대적 UI 가능, 임베디드/자동차까지 지원, 방대한 내장 컨트롤 
- **단점**: 상업용 라이선스 비용이 높음, C++ 진입 장벽, 기본 UI가 다소 구식으로 보일 수 있음
- **적합**: 전문 크리에이티브 소프트웨어(Autodesk Maya, OBS Studio), 의료/산업 장비, 고성능 앱

#### **Flutter**
- **장점**: Impeller 엔진으로 픽셀 단위 UI 일관성, 60/120fps 애니메이션, Hot Reload로 빠른 개발, Windows Desktop 채택률 20.1% 
- **단점**: Dart 언어 풀 작음, 번들 크기 8-10MB 이상, 네이티브 기능 연결 시 플랫폼 채널 복잡
- **적합**: 소비자 대상 앱, 커스텀 UI/애니메이션이 중요한 앱, 모바일+데스크톱 통합

#### **.NET MAUI**
- **장점**: C#/XAML로 모바일(iOS/Android)+데스크톱(Windows/macOS) 한 번에 개발, Windows에서는 WinUI 3 기반, Azure/MS 365 통합 
- **단점**: Windows 데스크톱은 "부산물" 수준, 제3자 생태계 제한, Xamarin 마이그레이션 외에는 설득력 약함
- **적합**: .NET 팀이 모바일+Windows를 동시에 타겟팅할 때

#### **Avalonia UI**
- **장점**: WPF와 유사한 XAML, Windows/macOS/Linux 크로스플랫폼, 오픈소스, Skia 기반 렌더링
- **단점**: 생태계가 WPF보다 작음, 일부 WPF 기능과 미세한 차이
- **적합**: WPF 개발자가 Linux/macOS까지 지원하고 싶을 때

---

### 3. 웹 기술 기반

#### **Electron**
- **장점**: JS 생태계 무한 확장, VS Code/Slack/Discord 등 검증됨, 크로스플랫폼, Chrome DevTools 디버깅 
- **단점**: "Hello World"도 100-300MB, 메모리 200-500MB 소모, 냉启动 3-5초, 보안 표면적 큼
- **적합**: 복잡한 IDE, 협업 도구, 웹 기술 팀이 데스크톱을 빠르게 만들어야 할 때

#### **Tauri**
- **장점**: Rust 기반으로 번들 ~5MB, 메모리 효율성 뛰어남, WebView2 사용, 보안 우수
- **단점**: Rust 학습 곡선, 생태계가 Electron보다 작음
- **적합**: Electron의 리소스 문제를 해결하고 싶은 소규모-중규모 앱

---

### 4. 기타

#### **Python (PyQt / PySide / Tkinter)**
- **장점**: Python 개발자 접근성 최고, 데이터 분석/내부 도구 빠른 제작
- **단점**: 배포 크기 큼(~30MB+), 성능 한계, 상업용 라이선스 고려 필요(PyQt)
- **적합**: 데이터 시각화 도구, 내부용 스크립트, 프로토타입

---

## 🎯 선택 가이드

| 상황 | 추천 방법 |
|------|----------|
| **새로운 Windows 전용 앱, 현대적 UI** | **WinUI 3** |
| **기존 WPF 앱 유지보수/확장** | **WPF** (.NET 9+) |
| **간단한 내부 도구, 빠른 개발** | **Windows Forms** 또는 **Python + PyQt** |
| **모바일 + Windows 동시 개발** | **Flutter** 또는 **.NET MAUI** |
| **Linux/macOS까지 크로스플랫폼** | **Qt**, **Flutter**, **Avalonia UI**, **Tauri** |
| **웹 개발자가 데스크톱 진출** | **Electron** (복잡한 앱) 또는 **Tauri** (경량 앱) |
| **전문 크리에이티브/산업 소프트웨어** | **Qt (C++)** |
| **극한의 성능/최소 크기** | **Win32 API** |

2026년 현재, Microsoft의 공식 방향은 **WinUI 3**이며, WPF는 여전히 강력한 대안으로 유지보수되고 있습니다. UWP는 신규 개발을 피해야 하며, 크로스플랫폼이 필요하다면 Flutter나 Qt가 가장 현실적인 선택지입니다.
