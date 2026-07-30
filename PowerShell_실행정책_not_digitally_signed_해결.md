# PowerShell .ps1 실행 오류 "not digitally signed" 해결

> 증상: `.ps1` 실행 시 `... cannot be loaded. The file ... is not digitally signed.` 오류로 실행 불가.

---

## 원인

PowerShell **실행 정책(ExecutionPolicy)** 때문. `Get-ExecutionPolicy -List` 확인 결과 예시:

```
        Scope ExecutionPolicy
        ----- ---------------
MachinePolicy       Undefined     ← 그룹정책(GPO) 강제 없음
   UserPolicy       Undefined
      Process          Bypass
  CurrentUser       Undefined
 LocalMachine    RemoteSigned     ← 실제 적용 정책
```

- **RemoteSigned**: 로컬에서 만든 스크립트는 실행되지만, **외부에서 받은(다운로드/메일/네트워크 공유) 스크립트**는 서명이 없으면 차단.
- 즉 해당 `.ps1`에 **"인터넷에서 가져옴"(Mark-of-the-Web, MOTW)** 표시가 붙어 "not digitally signed"가 발생.
- `MachinePolicy`/`UserPolicy`가 `Undefined`면 **GPO 강제가 없으므로** 아래 방법 모두 사용 가능. (GPO로 AllSigned가 강제된 환경이면 Bypass도 안 먹을 수 있음 → 그땐 Unblock/서명 필요.)

---

## 방법 1 — 파일 차단 해제 (근본 해결, 추천)

```powershell
Unblock-File -Path "C:\경로\스크립트.ps1"
```
이후 `.\스크립트.ps1` 로 정상 실행. 폴더 통째로:
```powershell
Get-ChildItem "C:\경로\*.ps1" | Unblock-File
```

## 방법 2 — 그 실행만 정책 우회 (간단, 영구변경 없음)

```powershell
powershell -ExecutionPolicy Bypass -File "C:\경로\스크립트.ps1"
```

## 방법 3 — 현재 세션에만 우회

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\스크립트.ps1
```
창 닫으면 원복.

## 방법 4 — 내 계정에 영구 적용 (관리자 권한 불필요)

```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```
단, MOTW가 붙어 있으면 RemoteSigned에서도 여전히 막히므로 **방법 1(Unblock-File)을 함께** 수행. 아예 안 막히게 하려면 `-ExecutionPolicy Bypass`로 설정.

---

## 추천 정리

| 상황 | 방법 |
|---|---|
| 한 번만 실행 | 방법 2 (`-ExecutionPolicy Bypass -File`) |
| 자주 실행할 스크립트 | 방법 1 (`Unblock-File`로 그 파일만 해제) |
| 이 세션 동안만 여러 개 실행 | 방법 3 (`-Scope Process Bypass`) |
| 내 계정 전체 영구 완화 | 방법 4 (+ 필요 시 Unblock-File) |

## 참고
- 로컬에서 생성한 스크립트는 원래 MOTW가 없어야 정상. 그래도 막히면 네트워크 드라이브/동기화 폴더를 거치며 MOTW가 붙은 것 → `Unblock-File`이 확실.
- 실행 정책은 "보안 경계"가 아니라 실수 방지용 가드레일. 신뢰하는 스크립트에만 우회 적용할 것.
