# `wscript + JavaScript`와 `Node.js + JavaScript`의 차이

둘 다 JavaScript를 실행하지만, **실행 엔진·제공 기능·주 용도**가 다릅니다.

| 구분 | `wscript.exe` + JavaScript | Node.js + JavaScript |
|---|---|---|
| 정식 이름 | Windows Script Host (WSH) | Node.js 런타임 |
| 주 대상 | Windows 자동화·관리 작업 | 서버·CLI·웹 개발·범용 자동화 |
| 기본 실행 명령 | `wscript script.js` | `node script.js` |
| 콘솔 출력 | 기본적으로 콘솔 창 없음 | 콘솔 기반 |
| 출력 방법 | `WScript.Echo()` | `console.log()` |
| Windows COM 자동화 | 기본 제공 (`ActiveXObject`) | 기본 제공 아님; 별도 패키지/네이티브 연동 필요 |
| 파일 처리 | `Scripting.FileSystemObject` 등 COM 객체 | 내장 `fs` 모듈 |
| 외부 패키지 | 사실상 없음 | npm 패키지 생태계 |
| 비동기 처리 | 제한적이고 오래된 방식 | Promise, `async`/`await`, 이벤트 루프 |
| 최신 JavaScript | 매우 제한적 | 최신 JavaScript 지원 |
| 플랫폼 | Windows 전용 | Windows, macOS, Linux |

## 1. `wscript + JavaScript`

`wscript.exe`는 Windows에 기본 포함된 **Windows Script Host** 실행기입니다. `.js` 파일을 실행할 수 있지만, 여기서의 JavaScript는 브라우저나 Node.js의 JavaScript와는 실행 환경이 다릅니다.

주로 다음과 같은 Windows 작업에 어울립니다.

- COM 객체를 통한 Office, WMI, 레지스트리, 파일 시스템 자동화
- 오래된 사내 관리 스크립트 유지보수
- 창을 띄우지 않는 간단한 Windows 스크립트 실행

### 예시

```js
// hello-wscript.js
WScript.Echo("Hello from Windows Script Host");

var shell = new ActiveXObject("WScript.Shell");
shell.Popup("작업이 완료되었습니다.", 3, "WSH", 64);
```

실행:

```powershell
wscript.exe .\hello-wscript.js
```

`wscript`는 대화 상자 중심으로 동작합니다. 콘솔에서 결과를 보고 싶으면 같은 WSH 계열인 `cscript.exe`를 사용할 수 있습니다.

```powershell
cscript.exe //nologo .\hello-wscript.js
```

## 2. `Node.js + JavaScript`

Node.js는 JavaScript를 서버·명령줄 도구·자동화 프로그램으로 실행하기 위한 현대적인 런타임입니다. 파일, 네트워크, 프로세스, HTTP 서버, 비동기 작업을 다루기 좋고 npm을 통해 패키지를 설치할 수 있습니다.

### 예시

```js
// hello-node.js
const fs = require("node:fs");

console.log("Hello from Node.js");
fs.writeFileSync("result.txt", "작업이 완료되었습니다.\n", "utf8");
```

실행:

```powershell
node .\hello-node.js
```

최신 문법을 쓰면 다음처럼 작성할 수도 있습니다.

```js
import { readFile } from "node:fs/promises";

const text = await readFile("input.txt", "utf8");
console.log(text);
```

## 3. 무엇을 선택하면 좋은가?

| 원하는 작업 | 권장 방식 |
|---|---|
| Windows COM/WMI/레지스트리/Office를 간단히 자동화 | `wscript` 또는 `cscript` |
| 기존 `.js` WSH 스크립트를 유지보수 | `wscript`/`cscript` |
| 콘솔 도구, API 호출, 파일 대량 처리, JSON 처리 | Node.js |
| 웹 서버, 봇, npm 라이브러리 사용 | Node.js |
| macOS/Linux에서도 실행 | Node.js |
| 새 자동화 도구를 장기적으로 개발 | 대체로 Node.js |

## 4. 실무에서 주의할 점

- **코드를 그대로 서로 바꿔 실행할 수 없습니다.** `WScript.Echo`, `ActiveXObject`는 Node.js에 없고, `require`, `import`, `console.log` 기반 Node.js 코드는 WSH에서 그대로 동작하지 않습니다.
- `wscript`는 GUI형이라 오류나 로그가 눈에 잘 안 보일 수 있습니다. 스크립트의 출력을 확인해야 한다면 `cscript //nologo`가 더 편합니다.
- `ActiveXObject`는 강력하지만 Windows와 COM 환경에 묶입니다. 새 프로젝트라면 특별히 COM 자동화가 필요한 경우를 제외하고 Node.js가 보통 더 유지보수하기 쉽습니다.
- Windows 관리 작업은 PowerShell이 더 자연스러운 경우도 많습니다. 다만 JavaScript 생태계의 라이브러리나 HTTP/JSON 작업이 중심이면 Node.js가 좋은 선택입니다.

## 한 줄 요약

`wscript + JavaScript`는 **Windows에 내장된 COM 중심 자동화 도구**이고, `Node.js + JavaScript`는 **현대적인 범용 JavaScript 런타임**입니다. 새로 만드는 도구라면 보통 Node.js를, Windows COM 자동화가 핵심이면 WSH를 선택하면 됩니다.
