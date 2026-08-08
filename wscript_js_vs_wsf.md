# WScript에서 실행하는 JS와 WSF의 차이

Windows Script Host(WSH)에서는 `.js`와 `.wsf` 모두 `wscript.exe` 또는 `cscript.exe`를 통해 실행할 수 있습니다.

핵심적인 차이는 다음과 같습니다.

> **`.js`는 스크립트 코드 자체이고, `.wsf`는 하나 이상의 스크립트를 어떻게 실행할지 정의하는 XML 기반 컨테이너입니다.**

## 1. `.js`

일반적인 WSH JScript 파일입니다.

예:

```js
var sh = new ActiveXObject("WScript.Shell");

sh.Popup(
    "Cache Cleaner",
    0,
    "Cache Cleaner",
    64
);
```

실행:

```text
wscript.exe clear-cache-gui.js
```

### 특징

- 구조가 단순합니다.
- JScript 코드 자체를 담습니다.
- `wscript.exe` 또는 `cscript.exe`로 실행합니다.
- 작은 자동화 스크립트나 런처에 적합합니다.

---

## 2. `.wsf`

`.wsf`는 **Windows Script File**이며 XML 형식입니다.

예:

```xml
<job>
    <script language="JScript">
        var sh = new ActiveXObject("WScript.Shell");

        sh.Popup(
            "Hello",
            0,
            "Test",
            64
        );
    </script>
</job>
```

실행:

```text
wscript.exe test.wsf
```

`.wsf`는 스크립트 자체라기보다 **스크립트를 담고 실행 환경을 정의하는 컨테이너**라고 생각하면 이해하기 쉽습니다.

---

## 3. 가장 큰 차이

### `.js`

```text
JS 코드
```

그 자체입니다.

### `.wsf`

```text
WSF
├── 설정
├── JScript
├── VBScript
└── 기타 Script
```

처럼 여러 스크립트와 실행 정보를 담을 수 있습니다.

---

## 4. JScript와 VBScript를 함께 사용할 수 있음

`.wsf`의 중요한 장점 중 하나입니다.

```xml
<job>

    <script language="JScript">
        // JScript
    </script>

    <script language="VBScript">
        ' VBScript
    </script>

</job>
```

하나의 `.wsf` 안에 JScript와 VBScript를 함께 넣을 수 있습니다.

반면 `.js` 파일은 기본적으로 JScript 파일이므로 VBScript 코드를 그대로 넣어 사용할 수 없습니다.

---

## 5. 외부 스크립트를 참조할 수 있음

`.wsf`에서는 외부 스크립트를 참조할 수 있습니다.

```xml
<job>
    <script language="JScript" src="common.js" />
    <script language="JScript" src="main.js" />
</job>
```

여러 스크립트를 하나의 WSH 작업으로 묶는 데 유용합니다.

---

## 6. 실행 인자를 정의할 수 있음

`.wsf`에서는 `<runtime>`을 이용해 스크립트 인자를 정의할 수 있습니다.

```xml
<job>
    <runtime>
        <named
            name="Force"
            helpstring="Force cleanup"
            type="simple"
        />
    </runtime>

    <script language="JScript">
        if (WScript.Arguments.Named.Exists("Force")) {
            WScript.Echo("Force mode");
        }
    </script>
</job>
```

실행:

```text
wscript.exe cleanup.wsf /Force
```

---

## 7. 여러 Job을 하나의 WSF에 넣을 수 있음

하나의 `.wsf` 안에 여러 작업을 정의할 수도 있습니다.

```xml
<package>

    <job id="Cleanup">
        <script language="JScript">
            // cleanup
        </script>
    </job>

    <job id="Backup">
        <script language="JScript">
            // backup
        </script>
    </job>

</package>
```

필요한 Job을 지정하여 실행할 수 있습니다.

```text
wscript.exe script.wsf /job:Cleanup
```

---

## 8. 단순한 런처라면 JS가 적합

현재 만들고 있는 `clear-cache-gui.js` 같은 프로그램에는 `.js`가 더 적합합니다.

현재 구조는 다음과 같이 단순합니다.

```text
clear-cache-gui.js
       |
       +-- PowerShell 7 확인
       |
       +-- 없으면 PowerShell 5.1 사용
       |
       +-- clear-cache-gui.ps1 존재 여부 확인
       |
       +-- PowerShell 실행
```

이를 굳이 WSF로 만들면:

```text
clear-cache-gui.wsf
       |
       +-- JScript
              |
              +-- PowerShell 실행
```

이 되어 오히려 복잡해집니다.

---

## 9. 언제 WSF를 고려할까?

### 단순 스크립트

```text
cleanup.js
```

→ **JS 추천**

### 여러 스크립트를 묶어야 하는 경우

```text
common.js
cleanup.js
backup.js
```

→ **WSF 고려**

### JScript + VBScript를 함께 사용하는 경우

```text
JScript
+
VBScript
```

→ **WSF**

### 스크립트 실행 환경이나 인자를 XML로 정의해야 하는 경우

→ **WSF**

---

## 10. 한눈에 비교

| 특징 | `.js` | `.wsf` |
|---|---|---|
| WSH 실행 | 가능 | 가능 |
| JScript | 가능 | 가능 |
| VBScript | 불가 | 가능 |
| 여러 스크립트 포함 | 제한적 | 가능 |
| 외부 스크립트 참조 | 직접 구성 필요 | 가능 |
| XML 설정 | 불가 | 가능 |
| 여러 Job | 불가 | 가능 |
| 단순 런처 | 매우 적합 | 상대적으로 부적합 |
| 복잡한 WSH 프로젝트 | 적합 | 매우 적합 |

---

## 결론

현재 만들고 계신 `clear-cache-gui` 런처에는 **`.js`를 그대로 사용하는 것이 좋습니다.**

특히 다음과 같은 구조는 깔끔합니다.

```text
clear-cache-gui.js
        |
        v
clear-cache-gui.ps1
```

`.wsf`의 장점은 JScript 자체의 기능을 늘려주는 것이 아니라,

- 여러 스크립트를 하나로 묶고
- JScript와 VBScript를 함께 사용하고
- 실행 인자와 Job 등을 정의하여
- WSH 실행 환경을 구성할 수 있다는 것

입니다.

따라서 **작고 단순한 WSH 자동화에는 `.js`**, **여러 스크립트와 실행 구성을 관리해야 하는 복잡한 WSH 작업에는 `.wsf`**라고 생각하시면 됩니다.
