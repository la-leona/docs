# VBScript를 JavaScript로 변환하기 - 완벽 가이드

이 문서는 Windows 환경에서 VBScript를 JavaScript로 변환하는 방법과 주의사항을 설명합니다.

## 1. VBScript vs JavaScript (WSH 환경)

### 1.1 실행 방식

| 항목 | VBScript | JavaScript |
| :--- | :--- | :--- |
| **파일 확장자** | `.vbs` | `.js` |
| **실행 명령어** | `cscript.exe` 또는 `wscript.exe` | `cscript.exe` 또는 `wscript.exe` |
| **기본 호스트** | Windows Script Host (WSH) | Windows Script Host (WSH) |
| **문법** | VB 기반 | ECMAScript 기반 |

### 1.2 주요 차이점

| 기능 | VBScript | JavaScript |
| :--- | :--- | :--- |
| **변수 선언** | `Dim`, `Set` | `var`, `let`, `const` |
| **객체 생성** | `CreateObject()` | `new ActiveXObject()` |
| **메시지 박스** | `MsgBox()` | `WScript.Echo()` 또는 `Popup()` |
| **문자열 연결** | `&` | `+` |
| **줄바꿈** | `vbCrLf` | `\r\n` |
| **주석** | `'` | `//` 또는 `/* */` |
| **조건문** | `If...Then...End If` | `if...else` |
| **반복문** | `For...Next` | `for`, `while` |

## 2. 변환 규칙

### 2.1 변수 선언

**VBScript:**
```vbscript
Dim sh, fso, here, target, exe, cmd
Set sh = CreateObject("WScript.Shell")
Set fso = CreateObject("Scripting.FileSystemObject")
```

**JavaScript:**
```javascript
var sh = new ActiveXObject("WScript.Shell");
var fso = new ActiveXObject("Scripting.FileSystemObject");
```

### 2.2 문자열 연결

**VBScript:**
```vbscript
target = here & "\clear-cache-gui.ps1"
msg = "Error:" & vbCrLf & "File not found"
```

**JavaScript:**
```javascript
target = here + "\\clear-cache-gui.ps1";
msg = "Error:\r\n" + "File not found";
```

### 2.3 조건문

**VBScript:**
```vbscript
If Not fso.FileExists(target) Then
    MsgBox "File not found", 16, "Error"
    WScript.Quit 1
End If
```

**JavaScript:**
```javascript
if (!fso.FileExists(target)) {
    sh.Popup("File not found", 0, "Error", 16);
    WScript.Quit(1);
}
```

### 2.4 메시지 박스

**VBScript:**
```vbscript
MsgBox "Message", 16, "Title"
```

**JavaScript:**
```javascript
// 방법 1: Popup (권장)
sh.Popup("Message", 0, "Title", 16);

// 방법 2: Echo (콘솔 출력)
WScript.Echo("Message");
```

### 2.5 환경 변수 확장

**VBScript:**
```vbscript
exe = sh.ExpandEnvironmentStrings("%ProgramFiles%")
```

**JavaScript:**
```javascript
var exe = sh.ExpandEnvironmentStrings("%ProgramFiles%");
```

## 3. 실제 변환 예제

### 3.1 원본 VBScript

```vbscript
' clear-cache-gui.vbs
Option Explicit

Dim sh, fso, here, target, exe, cmd
Set sh  = CreateObject("WScript.Shell")
Set fso = CreateObject("Scripting.FileSystemObject")

here   = fso.GetParentFolderName(WScript.ScriptFullName)
target = here & "\clear-cache-gui.ps1"

If Not fso.FileExists(target) Then
    MsgBox "GUI script not found:" & vbCrLf & target, 16, "Cache Cleaner"
    WScript.Quit 1
End If

exe = sh.ExpandEnvironmentStrings("%ProgramFiles%") & "\PowerShell\7\pwsh.exe"
If Not fso.FileExists(exe) Then
    exe = sh.ExpandEnvironmentStrings("%WINDIR%") & "\System32\WindowsPowerShell\v1.0\powershell.exe"
End If

cmd = """" & exe & """ -NoProfile -ExecutionPolicy Bypass -File """ & target & """"

sh.Run cmd, 0, False
```

### 3.2 변환된 JavaScript

```javascript
// clear-cache-gui.js
(function() {
    'use strict';

    try {
        var sh = new ActiveXObject("WScript.Shell");
        var fso = new ActiveXObject("Scripting.FileSystemObject");

        var here = fso.GetParentFolderName(WScript.ScriptFullName);
        var target = here + "\\clear-cache-gui.ps1";

        if (!fso.FileExists(target)) {
            var msg = "GUI script not found:\r\n" + target;
            sh.Popup(msg, 0, "Cache Cleaner", 16);
            WScript.Quit(1);
        }

        var programFiles = sh.ExpandEnvironmentStrings("%ProgramFiles%");
        var windir = sh.ExpandEnvironmentStrings("%WINDIR%");
        
        var exe = programFiles + "\\PowerShell\\7\\pwsh.exe";
        if (!fso.FileExists(exe)) {
            exe = windir + "\\System32\\WindowsPowerShell\\v1.0\\powershell.exe";
        }

        var cmd = '"' + exe + '" -NoProfile -ExecutionPolicy Bypass -File "' + target + '"';

        sh.Run(cmd, 0, false);

    } catch (e) {
        var sh_error = new ActiveXObject("WScript.Shell");
        sh_error.Popup("An error occurred:\r\n" + e.message, 0, "Cache Cleaner - Error", 16);
        WScript.Quit(1);
    }
})();
```

## 4. 주요 WSH 객체 및 메서드

### 4.1 WScript 객체

```javascript
// 스크립트 정보
WScript.ScriptFullName      // 현재 스크립트의 전체 경로
WScript.ScriptName          // 현재 스크립트의 파일명
WScript.Arguments           // 명령줄 인수

// 실행 제어
WScript.Quit(code)          // 스크립트 종료 (0 = 성공)
WScript.Echo(message)       // 콘솔에 메시지 출력
WScript.Sleep(milliseconds) // 지정된 시간 대기
```

### 4.2 WScript.Shell 객체

```javascript
var shell = new ActiveXObject("WScript.Shell");

// 프로그램 실행
shell.Run(command, windowStyle, waitOnReturn);
// windowStyle: 0=숨김, 1=정상, 2=최소화, 3=최대화
// waitOnReturn: true=대기, false=대기하지 않음

// 환경 변수
shell.ExpandEnvironmentStrings("%VARIABLE%");

// 레지스트리
shell.RegRead("HKEY_LOCAL_MACHINE\\...");
shell.RegWrite("HKEY_LOCAL_MACHINE\\...", value);

// 메시지 박스
shell.Popup(text, secondsToWait, title, type);
// type: 0=OK, 1=OK/Cancel, 2=Abort/Retry/Ignore, 3=Yes/No/Cancel, 
//       4=Yes/No, 5=Retry/Cancel, 16=Error, 32=Question, 48=Warning, 64=Info
```

### 4.3 Scripting.FileSystemObject 객체

```javascript
var fso = new ActiveXObject("Scripting.FileSystemObject");

// 파일 확인
fso.FileExists(path);
fso.FolderExists(path);

// 파일 작업
fso.CreateTextFile(filename, overwrite);
fso.DeleteFile(filename, force);
fso.CopyFile(source, destination, overwrite);
fso.MoveFile(source, destination);

// 폴더 작업
fso.CreateFolder(path);
fso.DeleteFolder(path, force);
fso.CopyFolder(source, destination, overwrite);
fso.MoveFolder(source, destination);

// 경로 작업
fso.GetParentFolderName(path);
fso.GetFileName(path);
fso.GetBaseName(path);
fso.GetExtensionName(path);
fso.BuildPath(basePath, name);
```

## 5. 실행 방법

### 5.1 명령줄에서 실행

```batch
REM cscript.exe 사용 (콘솔 창 표시)
cscript.exe "C:\path\to\clear-cache-gui.js"

REM wscript.exe 사용 (콘솔 창 숨김)
wscript.exe "C:\path\to\clear-cache-gui.js"
```

### 5.2 바탕화면 바로가기 생성

1. 바탕화면에서 마우스 우클릭 → **새로 만들기** → **바로가기**
2. **위치** 입력: `wscript.exe "C:\path\to\clear-cache-gui.js"`
3. **이름** 입력: `Cache Cleaner`
4. **마침** 클릭
5. 바로가기 속성 → **고급** → **관리자 권한으로 실행** 체크

## 6. 주의사항

### 6.1 경로 처리

VBScript와 JavaScript 모두 Windows 경로에서 백슬래시(`\`)를 이스케이프해야 합니다:

```javascript
// 잘못된 예
var path = "C:\Program Files\...";  // 오류!

// 올바른 예
var path = "C:\\Program Files\\...";  // 올바름
```

### 6.2 따옴표 처리

PowerShell 명령어에 따옴표가 포함될 때:

```javascript
// VBScript: """" = 하나의 따옴표
// JavaScript: \" = 하나의 따옴표

// PowerShell 명령어 예
var cmd = '"' + exe + '" -NoProfile -ExecutionPolicy Bypass -File "' + target + '"';
```

### 6.3 오류 처리

JavaScript에서는 try-catch를 사용하여 오류를 처리할 수 있습니다:

```javascript
try {
    // 코드
} catch (e) {
    WScript.Echo("Error: " + e.message);
    WScript.Quit(1);
}
```

## 7. 변환 체크리스트

변환할 때 다음 항목을 확인하세요:

- [ ] 모든 `Dim` 선언을 `var`로 변경
- [ ] 모든 `Set`을 `new ActiveXObject()`로 변경
- [ ] 모든 `&`를 `+`로 변경
- [ ] 모든 `vbCrLf`를 `\r\n`으로 변경
- [ ] 모든 `'`를 `//`로 변경
- [ ] 모든 `If...Then...End If`를 `if...else`로 변경
- [ ] 모든 `MsgBox`를 `Popup` 또는 `Echo`로 변경
- [ ] 모든 백슬래시를 이중 백슬래시(`\\`)로 변경
- [ ] 오류 처리를 위해 try-catch 추가
- [ ] 테스트 및 검증

## 8. 결론

VBScript를 JavaScript로 변환하는 것은 기본적인 문법 변환만으로 충분합니다. 두 언어 모두 Windows Script Host에서 실행되므로 동일한 COM 객체를 사용할 수 있습니다. 변환 후에는 반드시 테스트하여 모든 기능이 정상 작동하는지 확인하세요.
