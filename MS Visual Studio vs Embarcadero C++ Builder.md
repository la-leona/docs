# MS Visual Studio vs Embarcadero C++ Builder

Kimi: **MS Visual Studio**와 **Embarcadero C++ Builder**는 둘 다 Windows용 C++ 개발 도구지만, 탄생 배경부터 철학, 생태계가 완전히 다릅니다.

---

## 1. 기본 정보

| 항목 | **Visual Studio** | **C++ Builder** |
|---|---|---|
| **개발사** | Microsoft | Embarcadero (과거 Borland) |
| **첫 출시** | 1997년 (VS 5.0) | 1997년 (Borland C++ Builder 1.0) |
| **컴파일러** | MSVC (Microsoft C++) | 예전: Borland C++ / 현재: **Clang 기반** |
| **주력 언어** | C++, C#, VB, F#, JS/TS, Python | C++, Delphi (Object Pascal) |
| **IDE 통합도** | 거의 모든 MS 기술이 하나로 통합 | C++ + Delphi 중심의 독립 생태계 |

---

## 2. UI 개발 방식 (가장 큰 차이)

| | **Visual Studio** | **C++ Builder** |
|---|---|---|
| **네이티브 UI** | MFC, Win32 API, WinForms(C#), WPF(C#), UWP | **VCL** (Visual Component Library) |
| **크로스플랫폼 UI** | MAUI, Xamarin (C# 위주), Qt (3rd party) | **FireMonkey (FMX)** |
| **디자이너** | XAML 디자이너 / WinForms 드래그앤드롭 | **VCL/FMX 모두 강력한 드래그앤드롭** |
| **RAD 방식** | WinForms 정도만 해당 | **VCL이 RAD의 원조**, 속도 압도적 |

> **C++ Builder의 VCL**은 "코드 한 줄 없이 UI 완성"이 가능한 전통적인 RAD(Rapid Application Development) 도구입니다. Visual Studio의 WinForms보다 훨씬 빠르게 화면을 끌어모을 수 있어요.

---

## 3. 크로스플랫폼

| | **Visual Studio** | **C++ Builder** |
|---|---|---|
| **Windows** | ✅ 완벽 | ✅ 완벽 |
| **macOS** | ❌ C++는 직접 빌드 불가 (원격 빌드 복잡) | ✅ FireMonkey로 **네이티브 컴파일** |
| **Linux** | ❌ WSL/원격 빌드만 가능 | ✅ FireMonkey로 **네이티브 컴파일** |
| **iOS / Android** | ❌ (Xamarin/MAUI는 C#) | ✅ **FireMonkey로 모바일 네이티브** |
| **결론** | Windows 중심, 크로스플랫폼은 C#에 의존 | **C++ 하나로 데스크톱+모바일 동시 빌드** 가능 |

> C++ Builder는 **"C++로 iOS/Android 앱을 만든다"**는 게 실제로 가능합니다. FireMonkey가 플랫폼별 네이티브 컨트롤을 렌더링해줍니다.

---

## 4. 컴파일러 & 언어 표준

| | **Visual Studio** | **C++ Builder** |
|---|---|---|
| **컴파일러** | MSVC (`cl.exe`) | 예전: Borland / 현재: Win64용 Clang, Win32용 레거시 |
| **C++ 표준** | C++20 완벽, C++23 일부 | C++17 주력, 최신 표준 지원은 VS보다 느림 |
| **바이너리 호환성** | COFF (표준) | 예전: OMF (Borland 전용) / 현재: COFF로 개선 |
| **STL** | MSVC STL (완성도 높음) | Dinkumware + libc++ (혼합) |
| **빌드 속도** | 빠름 (IncrediBuild 지원) | VCL 프로젝트는 **컴파일 속도가 매우 빠름** (헤더 미리컴파일) |

> **주의:** C++ Builder의 컴파일러는 MSVC나 GCC와 **100% 바이너리 호환되지 않습니다.** Windows API는 사용 가능하지만, MSVC로 빌드한 `.lib`나 `.dll`을 그대로 링크하면 문제가 생길 수 있어요.

---

## 5. 디버거 & 도구

| | **Visual Studio** | **C++ Builder** |
|---|---|---|
| **디버거** | 세계 최강 수준 (IntelliTrace, 시간여행 디버깅 등) | 충분히 쓸 만함 |
| **IntelliSense** | 최고 수준 (코드 완성, 리팩토링) | LSP 기반, VS보다는 떨어짐 |
| **프로파일러** | VTune급 내장 프로파일러 | 기본 프로파일러 제공 |
| **디자이너** | WPF/MFC 디자이너 | **VCL/FMX 디자이너가 압도적으로 우수** |
| **Git 통합** | 완벽 | 기본 제공 |

---

## 6. 가격 & 라이선스

| | **Visual Studio** | **C++ Builder** |
|---|---|---|
| **무료 버전** | **Community Edition** (개인/소규모팀 무료) | 없음 (체험판만) |
| **유료 버전** | Professional ~$1,199 / Enterprise ~$5,999 | **매우 비쌈** (~$1,700~$5,000+) |
| **구독형** | Visual Studio Subscription | Update Subscription 별도 |
| **Delphi 번들** | 없음 | RAD Studio로 Delphi + C++ Builder 통합 구매 가능 |

> **Visual Studio Community**는 매우 강력한 기능을 무료로 제공하는 반면, C++ Builder는 **유료 진입장벽이 높습니다.**

---

## 7. 생태계 & 커뮤니티

| | **Visual Studio** | **C++ Builder** |
|---|---|---|
| **시장 점유율** | 압도적 1위 (Windows 개발의 표준) | 틈새 시장 (레거시 유지보수 + RAD 선호 개발자) |
| **취업 시장** | 필수 | 거의 없음 (Delphi/C++ Builder 유지보수 외) |
| **서드파티 라이브러리** | NuGet, vcpkg, Conan 등 풍부 | GetIt 패키지 매니저, 수는 적음 |
| **문서/커뮤니티** | 방대함 (MS Docs, Stack Overflow) | 공식 문서 + 소규모 커뮤니티 |
| **오픈소스** | GitHub/오픈소스 표준 | 상대적으로 폐쇄적 |

---

## 8. 언제 뭘 써야 할까?

### Visual Studio를 선택해야 할 때
- **Windows 전용** 네이티브/C++ 앱을 개발할 때
- **게임 개발** (DirectX, Unreal Engine 연동)
- **크로스플랫폼**이 필요하면서도 C++ 표준과 오픈소스 생태계를 중시할 때
- **회사/취업**을 고려할 때 (업계 표준)
- **무료로** 강력한 IDE를 쓰고 싶을 때 (Community Edition)

### C++ Builder를 선택해야 할 때
- **RAD 방식**으로 빠르게 Windows GUI를 만들고 싶을 때 (VCL)
- **C++ 하나로** Windows + macOS + iOS + Android를 동시에 타겟팅하고 싶을 때
- **레거시 시스템**이 Borland/Embarcadero 기반일 때 (유지보수)
- **델파이 개발자**가 C++로 확장하고 싶을 때 (RAD Studio 번들)
- **데이터베이스 연동**이 많은 업무용 프로그램을 빠르게 만들 때 (VCL의 DB 컴포넌트가 강력)

---

## 요약

| | **Visual Studio** | **C++ Builder** |
|---|---|---|
| **강점** | 생태계, 디버거, 표준 C++, 무료 Community | **RAD(VCL)**, **크로스플랫폼(FMX)**, 빠른 UI 개발 |
| **약점** | C++ GUI 개발은 MFC가 낡고 번거로움 | 가격, 생태계, 취업 시장, 최신 C++ 표준 |
| **이상적인 용도** | 게임, 시스템 프로그래밍, 대형 프로젝트 | 업무용 DB 앱, 멀티플랫폼 네이티브 앱, 레거시 유지보수 |

> **현실적인 조언:** 새 프로젝트를 시작한다면 거의 대부분의 경우 **Visual Studio(또는 VS Code + Clang/Clangd)**를 선택하는 것이 맞습니다. C++ Builder는 **특수한 상황** (VCL 레거시, FireMonkey 멀티플랫폼, 델파이 통합)에서만 고려하는 것이 일반적입니다.
