# PowerShell 히스토리 지우는 방법

PowerShell 엔 두 종류의 히스토리가 있음. 보통 "지우고 싶다" = **PSReadLine 영구 파일**.

## 종류 구분

| 종류 | 범위 | 저장 위치 |
|------|------|----------|
| 세션 히스토리 (`Get-History`) | 현재 창만, 닫으면 사라짐 | 메모리 |
| PSReadLine 히스토리 | 모든 세션 공유, 위/아래키·Ctrl+R 로 뜨는 것 | 파일(디스크) 영구 저장 |

## 1. PSReadLine 영구 파일 (보통 이걸 지우려는 것)

파일 위치 확인:
```powershell
(Get-PSReadLineOption).HistorySavePath
```
(보통 `C:\Users\<사용자>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`)

내용만 비우기:
```powershell
Clear-Content (Get-PSReadLineOption).HistorySavePath
```
파일째 삭제:
```powershell
Remove-Item (Get-PSReadLineOption).HistorySavePath
```

## 2. 현재 세션(메모리) 히스토리
```powershell
Clear-History                                              # Get-History 목록 비움
[Microsoft.PowerShell.PSConsoleReadLine]::ClearHistory()   # 위/아래키 버퍼(PSReadLine 메모리) 비움
```

## 중요한 함정
현재 창에서 파일만 지워도, **그 창을 닫을 때 이번 세션에 입력한 명령이 파일에 다시 append** 됨. 그래서:
- 확실히 지우려면: 파일을 지운 뒤 **메모리 버퍼(2번)도 비우고** -> 또는 **그 창을 닫고 새 창**을 열 것.
- 권장 순서:
  ```powershell
  Clear-Content (Get-PSReadLineOption).HistorySavePath
  [Microsoft.PowerShell.PSConsoleReadLine]::ClearHistory()
  ```
  그 후 창 닫기.

## 앞으로 아예 저장 안 하기
```powershell
Set-PSReadLineOption -HistorySaveStyle SaveNothing
```
(이 세션 한정. 영구로 하려면 PowerShell 프로필 `$PROFILE` 에 넣기)

## 민감정보만 지우고 싶다면
파일이 그냥 텍스트라 특정 줄만 편집기로 열어 지워도 됨:
```powershell
notepad (Get-PSReadLineOption).HistorySavePath
```

## 정리
영구 이력 -> HistorySavePath 파일 비우기/삭제 + 메모리 버퍼도 비우고 창 닫기 = 가장 깔끔.
