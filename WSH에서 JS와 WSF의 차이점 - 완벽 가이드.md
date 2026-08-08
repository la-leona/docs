# WSH에서 JS와 WSF의 차이점 - 완벽 가이드

Windows Script Host(WSH) 환경에서 JavaScript(`.js`)와 Windows Script File(`.wsf`) 파일의 차이점을 상세히 설명합니다 [1] [2] [3].

## 1. 기본 개념

### 1.1 JS (JavaScript) 파일

**JS**는 순수 JavaScript 코드를 포함하는 텍스트 파일입니다:

*   **파일 확장자**: `.js`
*   **형식**: 순수 텍스트 (코드만)
*   **실행 방식**: `wscript.exe` 또는 `cscript.exe`로 직접 실행
*   **구조**: 단순한 스크립트 코드

### 1.2 WSF (Windows Script File)

**WSF**는 XML 형식의 스크립트 파일입니다:

*   **파일 확장자**: `.wsf`
*   **형식**: XML 기반 구조
*   **실행 방식**: `wscript.exe` 또는 `cscript.exe`로 실행 (Job 지정 가능)
*   **구조**: 여러 Job과 Script 요소를 포함

## 2. 상세 비교표

| 항목 | JS | WSF |
| :--- | :--- | :--- |
| **파일 확장자** | `.js` | `.wsf` |
| **파일 형식** | 순수 텍스트 | XML |
| **구조** | 단순 | 복잡 (XML 계층) |
| **언어 혼합** | 불가능 | 가능 (JScript + VBScript) |
| **여러 Job** | 불가능 | 가능 |
| **외부 파일 포함** | 불가능 | 가능 (`<reference>`) |
| **상수 참조** | 수동 선언 필요 | 자동 참조 가능 |
| **오류 격리** | 전체 중단 | 모듈별 격리 |
| **명령줄 옵션** | 제한적 | 풍부함 |
| **학습 곡선** | 낮음 | 높음 |
| **사용 빈도** | 높음 | 낮음 |

## 3. 파일 형식 비교

### 3.1 JS 파일 구조

```javascript
// clear-cache-gui.js
// 순수 JavaScript 코드

var sh = new ActiveXObject("WScript.Shell");
var fso = new ActiveXObject("Scripting.FileSystemObject");

var here = fso.GetParentFolderName(WScript.ScriptFullName);
var target = here + "\\clear-cache-gui.ps1";

if (!fso.FileExists(target)) {
    sh.Popup("File not found", 0, "Error", 16);
    WScript.Quit(1);
}
```

### 3.2 WSF 파일 구조

```xml
<?xml version="1.0" encoding="UTF-8"?>
<package>
    <job id="MainJob">
        <script language="JScript">
            <![CDATA[
            var sh = new ActiveXObject("WScript.Shell");
            var fso = new ActiveXObject("Scripting.FileSystemObject");

            var here = fso.GetParentFolderName(WScript.ScriptFullName);
            var target = here + "\\clear-cache-gui.ps1";

            if (!fso.FileExists(target)) {
                sh.Popup("File not found", 0, "Error", 16);
                WScript.Quit(1);
            }
            ]]>
        </script>
    </job>
</package>
```

## 4. WSF의 주요 기능

### 4.1 여러 Job 정의

WSF에서는 하나의 파일에 여러 Job을 정의하고 선택적으로 실행할 수 있습니다:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<package>
    <job id="Job1">
        <script language="JScript">
            <![CDATA[
            WScript.Echo("This is Job 1");
            ]]>
        </script>
    </job>

    <job id="Job2">
        <script language="JScript">
            <![CDATA[
            WScript.Echo("This is Job 2");
            ]]>
        </script>
    </job>
</package>
```

**실행 방식:**

```batch
REM 첫 번째 Job 실행 (기본값)
cscript.exe script.wsf

REM 특정 Job 실행
cscript.exe //Job:Job1 script.wsf
cscript.exe //Job:Job2 script.wsf
```

### 4.2 언어 혼합 (Mixed Language)

WSF에서는 VBScript와 JScript를 같은 파일에서 혼합 사용할 수 있습니다:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<package>
    <job id="MixedLanguageJob">
        <!-- VBScript 부분 -->
        <script language="VBScript">
            <![CDATA[
            Dim myArray
            myArray = Array("apple", "banana", "cherry")
            ]]>
        </script>

        <!-- JScript 부분 (VBScript에서 정의한 변수 사용) -->
        <script language="JScript">
            <![CDATA[
            // VBScript에서 정의한 myArray 사용
            WScript.Echo("Array: " + myArray.join(", "));
            ]]>
        </script>
    </job>
</package>
```

### 4.3 외부 객체 참조

WSF에서는 COM 객체의 상수를 자동으로 참조할 수 있습니다:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<package>
    <job id="ADOJob">
        <!-- ADO 객체 참조 -->
        <reference object="ADODB.Recordset" />

        <script language="VBScript">
            <![CDATA[
            ' 상수를 직접 사용 가능 (값을 수동으로 선언할 필요 없음)
            MsgBox "adOpenForwardOnly = " & adOpenForwardOnly, vbInformation, "ADO Constants"
            ]]>
        </script>
    </job>
</package>
```

### 4.4 오류 격리 (Error Isolation)

WSF의 모듈식 구조는 한 스크립트의 오류가 다른 스크립트에 영향을 주지 않도록 격리합니다:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<package>
    <job id="ErrorHandlingJob">
        <!-- 첫 번째 스크립트: 오류 발생 -->
        <script language="VBScript">
            <![CDATA[
            On Error Resume Next
            WScript.Echo 4 / 0  ' 0으로 나누기 오류
            ]]>
        </script>

        <!-- 두 번째 스크립트: 계속 실행됨 -->
        <script language="VBScript">
            <![CDATA[
            WScript.Echo "This script still runs!"
            ]]>
        </script>
    </job>
</package>
```

## 5. 실행 방식 비교

### 5.1 JS 파일 실행

```batch
REM 콘솔 창 표시
cscript.exe script.js

REM 콘솔 창 숨김
wscript.exe script.js

REM 명령줄 인수 전달
cscript.exe script.js arg1 arg2

REM 타임아웃 설정 (30초)
cscript.exe //T:30 script.js
```

### 5.2 WSF 파일 실행

```batch
REM 첫 번째 Job 실행 (기본값)
cscript.exe script.wsf

REM 특정 Job 실행
cscript.exe //Job:JobName script.wsf

REM 명령줄 인수 전달
cscript.exe script.wsf arg1 arg2

REM 타임아웃 설정 (30초)
cscript.exe //T:30 script.wsf

REM 디버그 모드
cscript.exe //D script.wsf
```

## 6. 사용 시나리오별 선택 가이드

### 6.1 JS를 선택해야 할 때

*   **간단한 스크립트**: 단순한 작업 수행
*   **빠른 개발**: 빠르게 작성하고 테스트
*   **기존 코드**: 이미 작성된 JS 파일 활용
*   **호환성**: 다양한 환경에서 실행
*   **학습**: WSH 초보자

**예시:**
```javascript
// 간단한 파일 복사 스크립트
var fso = new ActiveXObject("Scripting.FileSystemObject");
fso.CopyFile("C:\\source.txt", "C:\\destination.txt");
WScript.Echo("File copied successfully!");
```

### 6.2 WSF를 선택해야 할 때

*   **복잡한 프로젝트**: 여러 Job과 모듈 필요
*   **언어 혼합**: VBScript와 JScript 함께 사용
*   **상수 참조**: COM 객체의 상수 자동 참조
*   **오류 격리**: 모듈별 오류 처리 필요
*   **대규모 스크립트**: 구조화된 코드 필요

**예시:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<package>
    <job id="ComplexJob">
        <reference object="ADODB.Connection" />
        <script language="VBScript">
            <![CDATA[
            ' 복잡한 데이터베이스 작업
            Dim conn
            Set conn = CreateObject("ADODB.Connection")
            conn.Open "Provider=SQLOLEDB;..."
            ]]>
        </script>
    </job>
</package>
```

## 7. 변환 가이드 (JS ↔ WSF)

### 7.1 JS를 WSF로 변환

**원본 JS:**
```javascript
var sh = new ActiveXObject("WScript.Shell");
var fso = new ActiveXObject("Scripting.FileSystemObject");

var path = "C:\\test.txt";
if (fso.FileExists(path)) {
    WScript.Echo("File exists");
} else {
    WScript.Echo("File not found");
}
```

**변환된 WSF:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<package>
    <job id="FileCheckJob">
        <script language="JScript">
            <![CDATA[
            var sh = new ActiveXObject("WScript.Shell");
            var fso = new ActiveXObject("Scripting.FileSystemObject");

            var path = "C:\\test.txt";
            if (fso.FileExists(path)) {
                WScript.Echo("File exists");
            } else {
                WScript.Echo("File not found");
            }
            ]]>
        </script>
    </job>
</package>
```

### 7.2 WSF를 JS로 변환

**원본 WSF (단일 Job):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<package>
    <job id="SimpleJob">
        <script language="JScript">
            <![CDATA[
            WScript.Echo("Hello from WSF");
            ]]>
        </script>
    </job>
</package>
```

**변환된 JS:**
```javascript
// simple.js
WScript.Echo("Hello from WSF");
```

## 8. WSF XML 구조 상세 설명

### 8.1 기본 요소

```xml
<?xml version="1.0" encoding="UTF-8"?>
<package>
    <!-- 패키지 전역 설정 (선택사항) -->
    <runtime version="VBScript.Regular" />

    <!-- 첫 번째 Job -->
    <job id="Job1">
        <!-- 외부 객체 참조 (선택사항) -->
        <reference object="ADODB.Recordset" />

        <!-- 스크립트 코드 -->
        <script language="JScript">
            <![CDATA[
            // 코드 작성
            ]]>
        </script>
    </job>

    <!-- 두 번째 Job -->
    <job id="Job2">
        <script language="VBScript">
            <![CDATA[
            ' 코드 작성
            ]]>
        </script>
    </job>
</package>
```

### 8.2 주요 XML 요소

| 요소 | 설명 | 필수 |
| :--- | :--- | :--- |
| `<package>` | 루트 요소 | 예 |
| `<job>` | 작업 정의 (id 속성 필수) | 예 |
| `<script>` | 스크립트 코드 (language 속성 필수) | 예 |
| `<reference>` | COM 객체 참조 | 아니오 |
| `<runtime>` | 런타임 버전 지정 | 아니오 |
| `<![CDATA[...]]>` | 문자 데이터 (XML 특수문자 포함 가능) | 권장 |

## 9. 주의사항

### 9.1 CDATA 사용

WSF에서 스크립트 코드를 XML 특수문자 없이 작성하려면 CDATA를 사용하세요:

```xml
<!-- 잘못된 예 (XML 파싱 오류 발생 가능) -->
<script language="JScript">
if (a < b && c > d) {
    WScript.Echo("Error!");
}
</script>

<!-- 올바른 예 (CDATA 사용) -->
<script language="JScript">
<![CDATA[
if (a < b && c > d) {
    WScript.Echo("OK!");
}
]]>
</script>
```

### 9.2 Job ID 지정

WSF에서 특정 Job을 실행하려면 `//Job:` 옵션을 사용하세요:

```batch
REM 첫 번째 Job만 실행 (기본값)
cscript.exe script.wsf

REM 특정 Job 실행
cscript.exe //Job:Job2 script.wsf
```

### 9.3 언어 지정

WSF에서 지원하는 언어:

*   `JScript`: Microsoft JavaScript
*   `VBScript`: Visual Basic Script
*   `Perl`: Perl 스크립트 (설치 필요)
*   `Python`: Python 스크립트 (설치 필요)
*   `Kixtart`: Kixtart 스크립트 (설치 필요)

## 10. 결론

| 상황 | 추천 | 이유 |
| :--- | :--- | :--- |
| **간단한 작업** | JS | 빠르고 간단함 |
| **복잡한 프로젝트** | WSF | 구조화되고 강력함 |
| **언어 혼합 필요** | WSF | 필수 기능 |
| **상수 참조 필요** | WSF | 자동 참조 가능 |
| **빠른 개발** | JS | 개발 속도 빠름 |
| **유지보수성** | WSF | 모듈화 가능 |

**최종 추천:**
- **개인 프로젝트**: **JS** (간단하고 빠름)
- **엔터프라이즈 환경**: **WSF** (강력하고 구조화됨)
- **기존 코드 유지**: 기존 형식 유지

## References

1.  [Using Windows Script Files (.wsf) - Inflectra](https://jsdoc.inflectra.com/HelpReadingPane.ashx?href=html/wsadvantagesofws.htm)
2.  [Windows Script File - Wikipedia](https://en.wikipedia.org/wiki/Windows_Script_File)
3.  [WSF - Windows Script File XML Format - Herong's Tutorial](https://www.herongyang.com/vb_script/WSH-WSF-Windows-Script-File-XML-Format.html)
