# Mark-of-the-Web (MOTW) 정리 — 언제 / 누가 / 어디에 붙나

> PowerShell "not digitally signed" 차단의 원인이 되는 MOTW의 정체·부착 시점·저장 위치 정리.

---

## MOTW가 무엇인가

**"이 파일은 신뢰할 수 없는 위치(주로 인터넷)에서 왔다"** 는 출처 꼬리표.
실체는 NTFS 파일에 딸려 붙는 **대체 데이터 스트림(ADS)** `파일명:Zone.Identifier`. 내용 형식:

```
[ZoneTransfer]
ZoneId=3
ReferrerUrl=https://...
HostUrl=https://...
```

`ZoneId` 값:

| 값 | 존 | 차단? |
|---|---|---|
| 0 | 내 PC(로컬) | X |
| 1 | 로컬 인트라넷 | 대체로 X |
| 2 | 신뢰할 수 있는 사이트 | X |
| **3** | **인터넷** | **O** |
| **4** | 제한된 사이트 | **O** |

---

## 언제 / 누가(무엇이) 붙이나

파일을 받아오는 **애플리케이션**이 Windows 보안 API(Attachment Execution Service)를 통해 부착. OS가 전부 자동으로 붙이는 게 아니라, "신뢰할 수 없는 존에서 가져온다"고 **앱이 표시**하는 방식.

| 붙는 경우 | 붙이는 주체 |
|---|---|
| 웹 다운로드 | Edge / Chrome / Firefox 등 브라우저 |
| 메일 첨부 저장 | Outlook 등 메일 클라이언트 |
| 메신저로 받은 파일 | Teams / 카톡PC 등 |
| zip 압축 해제 | 탐색기·최신 7-Zip 등이 압축 안 파일로 MOTW 전파 |
| 인터넷/제한 존 네트워크 공유(UNC)에서 가져옴 | 셸/앱 |
| 일부 클라우드 동기화·다운로드 | 해당 클라이언트 |

---

## 언제 안 붙나 / 사라지나

- **로컬에서 직접 생성·저장**한 파일 → 안 붙음
- **FAT/exFAT**(일부 USB 등)로 복사 → ADS 미지원이라 자동 소멸
- `Unblock-File` 또는 파일 속성창의 **"차단 해제"** 체크 → 제거
- MOTW 전파를 안 하는 옛 압축 도구로 풀면 안 붙기도 함

### 확인 예시 (로컬 생성 파일 → MOTW 없음)
```
Get-Item Clear-FirefoxCache.ps1 -Stream *
# Stream 결과: :$DATA 만 존재 (Zone.Identifier 없음 = MOTW 안 붙음)
```

---

## 어디에 저장되나

- **NTFS 전용.** 파일 본문(`:$DATA`)과 별개로 `파일:Zone.Identifier` 스트림에 저장.
- 탐색기 파일 크기엔 안 잡히고, 기본적으로 안 보임.

---

## 누가 읽고 반응하나 (효과)

- **PowerShell ExecutionPolicy**(RemoteSigned/AllSigned) → "not digitally signed" 차단
- **SmartScreen**, **Office 보호된 보기**, 실행 시 "다른 컴퓨터에서 온 파일" 경고

---

## 확인 · 제거 명령

```powershell
# 확인 (스트림 목록 / 내용)
Get-Item "파일.ps1" -Stream *
Get-Content "파일.ps1" -Stream Zone.Identifier

# 제거
Unblock-File "파일.ps1"
```
cmd에서는 `dir /r` 로 `:Zone.Identifier` 확인 가능.

---

## 한 줄 요약
**MOTW = 파일을 "받아온" 앱이, NTFS의 `Zone.Identifier` 스트림에, 다운로드/첨부/압축해제/원격경로 시점에 붙이는 출처 꼬리표.** 로컬에서 만든 파일엔 안 붙는다.
