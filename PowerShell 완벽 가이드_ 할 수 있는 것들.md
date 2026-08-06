# PowerShell 완벽 가이드: 할 수 있는 것들

PowerShell은 Microsoft에서 개발한 강력한 명령줄 셸이자 스크립팅 언어입니다. Windows 시스템 관리부터 자동화, 데이터 처리까지 광범위한 작업을 수행할 수 있습니다 [1] [2].

## 1. PowerShell의 주요 특징

PowerShell은 기존 명령 프롬프트(CMD)와 달리 다음과 같은 강력한 기능을 제공합니다:

*   **객체 지향 출력**: 텍스트 기반이 아닌 객체 기반의 데이터 처리로 더 정교한 작업이 가능합니다.
*   **파이프라인 기능**: 명령어의 출력을 다른 명령어의 입력으로 연결하여 복잡한 작업을 간단하게 처리합니다.
*   **스크립팅 언어**: 변수, 루프, 함수, 조건문 등 프로그래밍 기능을 지원합니다.
*   **원격 관리**: 로컬 및 원격 시스템에 대한 관리 작업을 수행할 수 있습니다.
*   **모듈 시스템**: 기능을 확장하기 위해 모듈을 설치하고 사용할 수 있습니다.
*   **자동화 엔진**: 반복적인 작업을 자동화하여 시간과 비용을 절감합니다.

## 2. 파일 및 폴더 관리

### 파일 목록 조회
```powershell
# 현재 디렉토리의 모든 파일 표시
Get-ChildItem

# 특정 폴더의 파일 표시
Get-ChildItem -Path "C:\Users\Documents"

# 하위 폴더까지 모두 표시
Get-ChildItem -Recurse

# 특정 확장자 파일만 표시
Get-ChildItem -Filter "*.txt"
```

### 파일 생성 및 삭제
```powershell
# 빈 파일 생성
New-Item -Path "C:\test.txt" -ItemType File

# 폴더 생성
New-Item -Path "C:\NewFolder" -ItemType Directory

# 파일 삭제
Remove-Item -Path "C:\test.txt"

# 폴더 및 하위 항목 모두 삭제
Remove-Item -Path "C:\NewFolder" -Recurse
```

### 파일 복사 및 이동
```powershell
# 파일 복사
Copy-Item -Path "C:\source.txt" -Destination "C:\destination.txt"

# 파일 이동
Move-Item -Path "C:\source.txt" -Destination "C:\destination.txt"

# 여러 파일 복사
Copy-Item -Path "C:\*.txt" -Destination "D:\" -Recurse
```

### 파일 내용 읽기 및 쓰기
```powershell
# 파일 내용 읽기
Get-Content -Path "C:\file.txt"

# 파일에 내용 쓰기 (덮어쓰기)
Set-Content -Path "C:\file.txt" -Value "Hello World"

# 파일에 내용 추가
Add-Content -Path "C:\file.txt" -Value "New Line"

# 파일 내용 검색
Select-String -Path "C:\file.txt" -Pattern "keyword"
```

## 3. 시스템 관리 및 정보 조회

### 시스템 정보 확인
```powershell
# 컴퓨터 정보 조회
Get-ComputerInfo

# 운영체제 정보
[System.Environment]::OSVersion

# 설치된 소프트웨어 목록
Get-ItemProperty HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, DisplayVersion
```

### 프로세스 관리
```powershell
# 실행 중인 프로세스 목록
Get-Process

# 특정 프로세스 찾기
Get-Process -Name "notepad"

# 프로세스 종료
Stop-Process -Name "notepad"

# 프로세스 시작
Start-Process -FilePath "C:\Program Files\Notepad++\notepad++.exe"
```

### 서비스 관리
```powershell
# 모든 서비스 목록
Get-Service

# 특정 서비스 상태 확인
Get-Service -Name "Windows Update"

# 서비스 시작
Start-Service -Name "Windows Update"

# 서비스 중지
Stop-Service -Name "Windows Update"
```

## 4. 네트워크 관리

### 네트워크 정보 조회
```powershell
# IP 주소 확인
Get-NetIPAddress

# 네트워크 어댑터 정보
Get-NetAdapter

# DNS 설정 확인
Get-DnsClientServerAddress

# 네트워크 연결 테스트
Test-Connection -ComputerName "google.com"
```

### 방화벽 관리
```powershell
# 방화벽 상태 확인
Get-NetFirewallProfile

# 특정 포트 규칙 확인
Get-NetFirewallRule -DisplayName "*port*"

# 방화벽 규칙 추가
New-NetFirewallRule -DisplayName "Allow Port 8080" -Direction Inbound -LocalPort 8080 -Protocol TCP -Action Allow
```

## 5. 작업 자동화

### 배치 처리 (반복 작업)
```powershell
# 폴더의 모든 JPG를 PNG로 변환 (ImageMagick 필요)
Get-ChildItem -Filter "*.jpg" | ForEach-Object {
    convert $_.FullName $_.BaseName".png"
}

# 특정 폴더의 모든 파일 이름 변경
Get-ChildItem -Filter "*.txt" | ForEach-Object {
    Rename-Item -Path $_.FullName -NewName ($_.Name -replace "old", "new")
}
```

### 예약된 작업 생성
```powershell
# 매일 오전 9시에 스크립트 실행하는 작업 생성
$trigger = New-ScheduledTaskTrigger -Daily -At 9am
$action = New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-File C:\script.ps1"
Register-ScheduledTask -TaskName "MyTask" -Trigger $trigger -Action $action -RunLevel Highest
```

### 파일 모니터링
```powershell
# 특정 폴더의 파일 변경 감시
$watcher = New-Object System.IO.FileSystemWatcher
$watcher.Path = "C:\Monitor"
$watcher.Filter = "*.*"
$watcher.IncludeSubdirectories = $true

Register-ObjectEvent -InputObject $watcher -EventName "Changed" -Action {
    Write-Host "File changed: $($Event.SourceEventArgs.FullPath)"
}
```

## 6. 데이터 처리 및 분석

### CSV 파일 처리
```powershell
# CSV 파일 읽기
$data = Import-Csv -Path "C:\data.csv"

# CSV 파일 쓰기
$data | Export-Csv -Path "C:\output.csv" -NoTypeInformation

# CSV 데이터 필터링
$data | Where-Object { $_.Age -gt 30 } | Select-Object Name, Age
```

### JSON 파일 처리
```powershell
# JSON 파일 읽기
$json = Get-Content -Path "C:\data.json" | ConvertFrom-Json

# JSON 파일 쓰기
$object | ConvertTo-Json | Set-Content -Path "C:\output.json"
```

### 데이터 정렬 및 필터링
```powershell
# 데이터 정렬
Get-ChildItem | Sort-Object -Property Length -Descending

# 데이터 필터링
Get-Process | Where-Object { $_.Memory -gt 100MB }

# 데이터 그룹화
Get-Process | Group-Object -Property ProcessName | Select-Object Name, Count
```

## 7. 텍스트 처리

### 문자열 검색 및 치환
```powershell
# 파일에서 특정 문자열 검색
Select-String -Path "C:\file.txt" -Pattern "error"

# 문자열 치환
(Get-Content "C:\file.txt") -replace "old", "new" | Set-Content "C:\file.txt"

# 여러 파일에서 검색
Get-ChildItem -Filter "*.txt" | Select-String -Pattern "keyword"
```

### 텍스트 분석
```powershell
# 파일의 라인 수 세기
(Get-Content "C:\file.txt").Count

# 특정 패턴의 라인 수 세기
(Select-String -Path "C:\file.txt" -Pattern "error").Count

# 파일의 첫 10줄 표시
Get-Content -Path "C:\file.txt" -TotalCount 10
```

## 8. 보안 및 권한 관리

### 파일 권한 설정
```powershell
# 파일 권한 확인
Get-Acl -Path "C:\file.txt"

# 파일 권한 변경
$acl = Get-Acl -Path "C:\file.txt"
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule("DOMAIN\User", "FullControl", "Allow")
$acl.SetAccessRule($rule)
Set-Acl -Path "C:\file.txt" -AclObject $acl
```

### 암호화
```powershell
# 파일 암호화 (EFS)
cipher /e C:\file.txt

# PowerShell에서 암호화
$file = "C:\file.txt"
cipher /e $file
```

## 9. 원격 관리

### 원격 컴퓨터 관리
```powershell
# 원격 컴퓨터에서 명령 실행
Invoke-Command -ComputerName "RemotePC" -ScriptBlock { Get-Process }

# 원격 세션 생성
$session = New-PSSession -ComputerName "RemotePC"
Invoke-Command -Session $session -ScriptBlock { Get-Service }

# 세션 종료
Remove-PSSession -Session $session
```

## 10. 실무 팁

### 1. 스크립트 실행 정책 변경
```powershell
# 현재 실행 정책 확인
Get-ExecutionPolicy

# 실행 정책 변경 (현재 사용자만)
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### 2. 도움말 조회
```powershell
# 특정 명령어 도움말
Get-Help Get-ChildItem

# 도움말 예시 보기
Get-Help Get-ChildItem -Examples

# 온라인 도움말 열기
Get-Help Get-ChildItem -Online
```

### 3. 명령어 별칭 사용
```powershell
# 별칭 목록 확인
Get-Alias

# 특정 명령어의 별칭 확인
Get-Alias -Definition Get-ChildItem
```

### 4. 오류 처리
```powershell
Try {
    Get-Item -Path "C:\NonExistent.txt"
}
Catch {
    Write-Host "파일을 찾을 수 없습니다: $_"
}
```

## 11. 결론

PowerShell은 단순한 명령줄 도구를 넘어, 시스템 관리, 자동화, 데이터 처리 등 다양한 분야에서 활용할 수 있는 강력한 도구입니다. 기본 명령어부터 시작하여 점차 복잡한 스크립트를 작성하면서 숙련도를 높일 수 있습니다.

## References

1.  [What is PowerShell? - Microsoft Learn](https://learn.microsoft.com/en-us/powershell/scripting/overview?view=powershell-7.6)
2.  [What is PowerShell? A Complete Guide to Its Features & Uses - Netwrix](https://netwrix.com/en/resources/blog/what-is-powershell-guide/)
