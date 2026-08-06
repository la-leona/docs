# WPF GUI 만들고 EXE로 컴파일하기 - 완벽 가이드

이 가이드는 WPF를 사용하여 간단한 GUI 애플리케이션을 만들고, 이를 실행 가능한 EXE 파일로 컴파일하는 전체 과정을 단계별로 설명합니다.

## 1. 필수 준비물

WPF 개발을 시작하기 전에 다음이 필요합니다:

*   **Visual Studio 2022** (또는 2019 이상): [공식 웹사이트](https://visualstudio.microsoft.com/downloads/)에서 다운로드
*   **.NET Framework** 또는 **.NET 6/7/8**: Visual Studio 설치 시 함께 설치됨
*   **기본적인 C# 지식**: 객체 지향 프로그래밍의 기초

## 2. Visual Studio에서 WPF 프로젝트 생성

### 2.1 새 프로젝트 생성

1. Visual Studio를 실행합니다.
2. **파일(File)** → **새로 만들기(New)** → **프로젝트(Project)** 클릭
3. **WPF 애플리케이션(.NET Framework)** 또는 **WPF 애플리케이션(.NET)** 선택
4. 프로젝트 이름 입력 (예: `MyWPFApp`)
5. 위치 선택 후 **만들기(Create)** 클릭

### 2.2 프로젝트 구조 이해

생성된 프로젝트의 주요 파일:

```
MyWPFApp/
├── MainWindow.xaml          # UI 디자인 (XAML)
├── MainWindow.xaml.cs       # UI 로직 (C# 코드비하인드)
├── App.xaml                 # 애플리케이션 설정
├── App.xaml.cs              # 애플리케이션 로직
└── MyWPFApp.csproj          # 프로젝트 설정 파일
```

## 3. 간단한 GUI 만들기

### 3.1 MainWindow.xaml 수정

기본 XAML 파일을 다음과 같이 수정합니다:

```xml
<Window x:Class="MyWPFApp.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="간단한 계산기" Height="300" Width="400"
        WindowStartupLocation="CenterScreen"
        Background="#F5F5F5">
    <Grid>
        <StackPanel VerticalAlignment="Center" HorizontalAlignment="Center" Width="300">
            <!-- 제목 -->
            <TextBlock Text="간단한 계산기" 
                       FontSize="24" 
                       FontWeight="Bold" 
                       Foreground="#333333"
                       Margin="0,0,0,20"
                       TextAlignment="Center"/>
            
            <!-- 첫 번째 숫자 입력 -->
            <TextBlock Text="첫 번째 숫자:" 
                       FontSize="14" 
                       Margin="0,0,0,5"/>
            <TextBox x:Name="FirstNumberBox" 
                     Height="35" 
                     Padding="10"
                     FontSize="14"
                     Margin="0,0,0,15"/>
            
            <!-- 두 번째 숫자 입력 -->
            <TextBlock Text="두 번째 숫자:" 
                       FontSize="14" 
                       Margin="0,0,0,5"/>
            <TextBox x:Name="SecondNumberBox" 
                     Height="35" 
                     Padding="10"
                     FontSize="14"
                     Margin="0,0,0,15"/>
            
            <!-- 버튼 그룹 -->
            <StackPanel Orientation="Horizontal" Margin="0,0,0,15">
                <Button Content="더하기" 
                        Click="AddButton_Click" 
                        Height="40" 
                        Width="70"
                        Margin="0,0,5,0"
                        Background="#4CAF50"
                        Foreground="White"
                        FontSize="12"
                        FontWeight="Bold"/>
                <Button Content="빼기" 
                        Click="SubtractButton_Click" 
                        Height="40" 
                        Width="70"
                        Margin="5,0,5,0"
                        Background="#2196F3"
                        Foreground="White"
                        FontSize="12"
                        FontWeight="Bold"/>
                <Button Content="곱하기" 
                        Click="MultiplyButton_Click" 
                        Height="40" 
                        Width="70"
                        Margin="5,0,5,0"
                        Background="#FF9800"
                        Foreground="White"
                        FontSize="12"
                        FontWeight="Bold"/>
                <Button Content="나누기" 
                        Click="DivideButton_Click" 
                        Height="40" 
                        Width="70"
                        Margin="5,0,0,0"
                        Background="#F44336"
                        Foreground="White"
                        FontSize="12"
                        FontWeight="Bold"/>
            </StackPanel>
            
            <!-- 결과 표시 -->
            <TextBlock Text="결과:" 
                       FontSize="14" 
                       Margin="0,0,0,5"/>
            <TextBlock x:Name="ResultBlock" 
                       Height="40" 
                       Padding="10"
                       Background="White"
                       FontSize="16"
                       FontWeight="Bold"
                       Foreground="#333333"
                       VerticalAlignment="Center"
                       Text="결과가 여기에 표시됩니다"/>
        </StackPanel>
    </Grid>
</Window>
```

### 3.2 MainWindow.xaml.cs 수정

코드비하인드 파일을 다음과 같이 수정합니다:

```csharp
using System.Windows;

namespace MyWPFApp
{
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
        }

        // 더하기 버튼 클릭 이벤트
        private void AddButton_Click(object sender, RoutedEventArgs e)
        {
            PerformCalculation((a, b) => a + b, "더하기");
        }

        // 빼기 버튼 클릭 이벤트
        private void SubtractButton_Click(object sender, RoutedEventArgs e)
        {
            PerformCalculation((a, b) => a - b, "빼기");
        }

        // 곱하기 버튼 클릭 이벤트
        private void MultiplyButton_Click(object sender, RoutedEventArgs e)
        {
            PerformCalculation((a, b) => a * b, "곱하기");
        }

        // 나누기 버튼 클릭 이벤트
        private void DivideButton_Click(object sender, RoutedEventArgs e)
        {
            if (double.TryParse(SecondNumberBox.Text, out double secondNum) && secondNum == 0)
            {
                ResultBlock.Text = "오류: 0으로 나눌 수 없습니다!";
                ResultBlock.Foreground = new System.Windows.Media.SolidColorBrush(System.Windows.Media.Colors.Red);
                return;
            }
            PerformCalculation((a, b) => a / b, "나누기");
        }

        // 계산 수행 메서드
        private void PerformCalculation(System.Func<double, double, double> operation, string operationName)
        {
            // 입력값 검증
            if (!double.TryParse(FirstNumberBox.Text, out double firstNum))
            {
                ResultBlock.Text = "오류: 첫 번째 숫자가 유효하지 않습니다!";
                ResultBlock.Foreground = new System.Windows.Media.SolidColorBrush(System.Windows.Media.Colors.Red);
                return;
            }

            if (!double.TryParse(SecondNumberBox.Text, out double secondNum))
            {
                ResultBlock.Text = "오류: 두 번째 숫자가 유효하지 않습니다!";
                ResultBlock.Foreground = new System.Windows.Media.SolidColorBrush(System.Windows.Media.Colors.Red);
                return;
            }

            // 계산 수행
            double result = operation(firstNum, secondNum);
            ResultBlock.Text = $"{firstNum} {operationName} {secondNum} = {result}";
            ResultBlock.Foreground = new System.Windows.Media.SolidColorBrush(System.Windows.Media.Colors.Green);
        }
    }
}
```

## 4. 애플리케이션 테스트

### 4.1 디버그 모드에서 실행

1. **F5** 키를 누르거나 **디버그(Debug)** → **디버깅 시작(Start Debugging)** 클릭
2. 애플리케이션이 실행되고 숫자를 입력하여 계산 기능 테스트
3. 정상 작동 확인 후 창 닫기

## 5. EXE 파일로 컴파일하기

### 5.1 Release 빌드 생성

1. **빌드(Build)** → **구성 관리자(Configuration Manager)** 클릭
2. **활성 솔루션 구성(Active solution configuration)**을 **Debug**에서 **Release**로 변경
3. **닫기(Close)** 클릭

### 5.2 프로젝트 빌드

1. **빌드(Build)** → **솔루션 빌드(Build Solution)** 클릭
2. 또는 **Ctrl + Shift + B** 단축키 사용
3. 빌드가 완료될 때까지 기다림 (출력 창에서 "빌드 완료" 메시지 확인)

### 5.3 EXE 파일 위치 확인

빌드가 완료되면 EXE 파일은 다음 위치에 있습니다:

```
프로젝트폴더\bin\Release\net6.0\MyWPFApp.exe
```

또는 .NET Framework를 사용한 경우:

```
프로젝트폴더\bin\Release\MyWPFApp.exe
```

## 6. EXE 파일 배포

### 6.1 필요한 파일 확인

Release 폴더에는 다음 파일들이 포함됩니다:

*   **MyWPFApp.exe**: 실행 파일 (메인)
*   **.dll 파일들**: 필요한 라이브러리
*   **config 파일**: 애플리케이션 설정

### 6.2 배포 방법

#### 방법 1: 폴더 전체 복사
Release 폴더의 모든 파일을 다른 컴퓨터에 복사하고 EXE 파일 실행

#### 방법 2: 자체 포함 배포 (Self-contained)

프로젝트 파일(.csproj)을 다음과 같이 수정:

```xml
<PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net6.0-windows</TargetFramework>
    <SelfContained>true</SelfContained>
    <RuntimeIdentifier>win-x64</RuntimeIdentifier>
    <PublishSingleFile>true</PublishSingleFile>
    <PublishReadyToRun>true</PublishReadyToRun>
</PropertyGroup>
```

그 후 다시 빌드하면 독립 실행형 EXE 파일 생성

#### 방법 3: 설치 프로그램 생성

Visual Studio Installer Projects 확장을 사용하여 MSI 설치 파일 생성

## 7. 실행 파일 최적화

### 7.1 파일 크기 줄이기

프로젝트 파일에 다음 추가:

```xml
<PropertyGroup>
    <DebugType>embedded</DebugType>
    <DebugSymbols>false</DebugSymbols>
    <TrimMode>link</TrimMode>
</PropertyGroup>
```

### 7.2 성능 최적화

1. Release 빌드 사용 (Debug 빌드는 성능이 느림)
2. 불필요한 라이브러리 제거
3. 이미지 등 리소스 최적화

## 8. 문제 해결

### 문제 1: "프로젝트를 빌드할 수 없습니다"

**해결책:**
- Visual Studio를 관리자 권한으로 실행
- **빌드** → **솔루션 정리(Clean Solution)** 실행 후 다시 빌드
- NuGet 패키지 복원: **도구** → **NuGet 패키지 관리자** → **패키지 관리자 콘솔** → `Update-Package -Reinstall`

### 문제 2: "필요한 DLL 파일이 없습니다"

**해결책:**
- Release 폴더의 모든 파일을 함께 배포
- 자체 포함 배포 옵션 사용

### 문제 3: "애플리케이션이 실행되지 않습니다"

**해결책:**
- 대상 컴퓨터에 .NET Framework 또는 .NET Runtime 설치 필요
- 관리자 권한으로 실행 시도
- 이벤트 뷰어에서 오류 메시지 확인

## 9. 추가 팁

### 9.1 애플리케이션 아이콘 설정

프로젝트 파일(.csproj)에 추가:

```xml
<PropertyGroup>
    <ApplicationIcon>icon.ico</ApplicationIcon>
</PropertyGroup>
```

### 9.2 버전 정보 설정

프로젝트 파일에 추가:

```xml
<PropertyGroup>
    <AssemblyVersion>1.0.0.0</AssemblyVersion>
    <FileVersion>1.0.0.0</FileVersion>
    <ProductVersion>1.0</ProductVersion>
</PropertyGroup>
```

### 9.3 EXE 파일 속성 확인

생성된 EXE 파일을 마우스 우클릭 → **속성** → **세부 정보** 탭에서 버전 정보 확인

## 10. 체크리스트

배포 전 확인 사항:

- [ ] 애플리케이션이 Release 모드에서 정상 작동
- [ ] 모든 필요한 파일이 Release 폴더에 있음
- [ ] EXE 파일이 다른 컴퓨터에서 실행됨
- [ ] 오류 메시지가 없음
- [ ] 성능이 만족스러움
- [ ] 아이콘 및 버전 정보 설정됨

## 11. 결론

WPF로 GUI 애플리케이션을 만들고 EXE로 컴파일하는 과정은 간단합니다. Visual Studio의 강력한 도구를 활용하면 전문적인 Windows 데스크톱 애플리케이션을 빠르게 개발하고 배포할 수 있습니다. 위의 단계를 따르면 첫 번째 WPF 애플리케이션을 성공적으로 만들고 배포할 수 있습니다.
