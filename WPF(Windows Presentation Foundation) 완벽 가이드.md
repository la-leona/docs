# WPF(Windows Presentation Foundation) 완벽 가이드

WPF는 Microsoft에서 개발한 Windows 데스크톱 애플리케이션을 위한 현대적인 UI 프레임워크입니다. .NET Framework 3.0부터 소개되었으며, 현재는 오픈 소스로 .NET 플랫폼에서 계속 유지보수되고 있습니다 [1] [2].

## 1. WPF란 무엇인가?

WPF(Windows Presentation Foundation)는 다음과 같은 특징을 가진 UI 프레임워크입니다:

*   **해상도 독립적**: 화면 해상도와 관계없이 일관된 UI를 제공합니다.
*   **벡터 기반 렌더링**: DirectX를 기반으로 하는 고성능 그래픽 렌더링 엔진을 사용합니다.
*   **XAML 기반**: XML 기반의 마크업 언어인 XAML을 사용하여 UI를 선언적으로 정의합니다.
*   **C# 코드비하인드**: UI 로직은 C#으로 작성합니다.
*   **오픈 소스**: GitHub에서 오픈 소스로 관리되고 있습니다 [1].

## 2. WPF의 주요 특징

WPF는 다음과 같은 풍부한 기능을 제공합니다 [1] [2]:

| 기능 | 설명 |
| :--- | :--- |
| **XAML** | 확장 가능한 애플리케이션 마크업 언어 |
| **컨트롤** | 버튼, 텍스트박스, 데이터그리드 등 다양한 UI 컨트롤 |
| **데이터 바인딩** | 데이터와 UI 요소 간의 자동 동기화 |
| **레이아웃** | Grid, StackPanel, DockPanel 등 유연한 레이아웃 시스템 |
| **2D/3D 그래픽** | 2D 및 3D 그래픽 렌더링 지원 |
| **애니메이션** | 부드러운 애니메이션 효과 |
| **스타일 및 템플릿** | UI 요소의 모양과 동작을 재사용 가능하게 정의 |
| **문서** | 고급 문서 처리 기능 |
| **미디어** | 오디오, 비디오 재생 지원 |
| **텍스트 및 타이포그래피** | 고급 텍스트 처리 |

## 3. WPF vs 다른 UI 프레임워크

### 3.1 WPF vs WinForms

| 항목 | WPF | WinForms |
| :--- | :--- | :--- |
| **렌더링 엔진** | DirectX 기반 | GDI+ 기반 |
| **마크업 언어** | XAML | 없음 (코드 기반) |
| **성능** | 높음 (현대 그래픽 하드웨어 활용) | 낮음 (레거시 기술) |
| **학습 곡선** | 가파름 | 완만함 |
| **UI 유연성** | 매우 높음 | 제한적 |
| **데이터 바인딩** | 강력함 | 기본적 |
| **2D/3D 그래픽** | 지원 | 제한적 |

### 3.2 WPF vs WSL/Cross-platform

| 항목 | WPF | WSL/Cross-platform |
| :--- | :--- | :--- |
| **플랫폼** | Windows 전용 | 다중 플랫폼 |
| **성능** | 최적화됨 | 중간 |
| **기능** | 풍부함 | 제한적 |
| **개발 생산성** | 높음 (Windows 전용) | 중간 |

## 4. WPF 아키텍처

WPF의 주요 구성 요소는 다음과 같습니다 [2]:

```
┌─────────────────────────────────────────┐
│       Application Layer (C#)             │
├─────────────────────────────────────────┤
│   PresentationFramework (XAML Parser)    │
├─────────────────────────────────────────┤
│    PresentationCore (Core Rendering)     │
├─────────────────────────────────────────┤
│  Milcore (Unmanaged, DirectX Integration)│
├─────────────────────────────────────────┤
│  DirectX / Graphics Hardware             │
└─────────────────────────────────────────┘
```

*   **PresentationFramework**: XAML 파싱 및 UI 프레임워크
*   **PresentationCore**: 핵심 렌더링 엔진
*   **Milcore**: 비관리 코드로 DirectX와 통합
*   **DirectX**: 저수준 그래픽 렌더링

## 5. XAML과 코드비하인드

### 5.1 XAML (마크업)

XAML은 XML 기반의 마크업 언어로 UI의 모양을 선언적으로 정의합니다:

```xml
<Window
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    Title="My WPF Application"
    Width="400" Height="300">
    
    <StackPanel Margin="10">
        <TextBlock Text="Hello WPF" FontSize="20" FontWeight="Bold"/>
        <TextBox Name="InputBox" Margin="0,10,0,0"/>
        <Button Name="SubmitButton" Content="Submit" Margin="0,10,0,0"/>
    </StackPanel>
</Window>
```

### 5.2 코드비하인드 (C#)

UI의 동작을 C#으로 구현합니다:

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

        private void SubmitButton_Click(object sender, RoutedEventArgs e)
        {
            string input = InputBox.Text;
            MessageBox.Show($"You entered: {input}");
        }
    }
}
```

## 6. 주요 개념

### 6.1 데이터 바인딩

데이터와 UI 요소를 자동으로 동기화합니다:

```xml
<TextBox Text="{Binding UserName}"/>
<TextBlock Text="{Binding UserAge, StringFormat='Age: {0}'}"/>
```

### 6.2 MVVM 패턴

WPF 애플리케이션의 표준 아키텍처 패턴입니다:

```csharp
// Model - 데이터
public class User
{
    public string Name { get; set; }
    public int Age { get; set; }
}

// ViewModel - 중개자
public class UserViewModel : INotifyPropertyChanged
{
    private User _user = new User();

    public string Name
    {
        get { return _user.Name; }
        set
        {
            _user.Name = value;
            OnPropertyChanged(nameof(Name));
        }
    }

    public event PropertyChangedEventHandler PropertyChanged;

    protected void OnPropertyChanged(string name)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
    }
}

// View - UI (XAML)
// <TextBox Text="{Binding Name}"/>
```

### 6.3 명령(Commands)

사용자 상호작용을 처리하는 구조화된 방식입니다:

```csharp
public class RelayCommand : ICommand
{
    private Action<object> _execute;
    private Func<object, bool> _canExecute;

    public RelayCommand(Action<object> execute, Func<object, bool> canExecute = null)
    {
        _execute = execute;
        _canExecute = canExecute;
    }

    public event EventHandler CanExecuteChanged
    {
        add { CommandManager.RequerySuggested += value; }
        remove { CommandManager.RequerySuggested -= value; }
    }

    public bool CanExecute(object parameter) => _canExecute?.Invoke(parameter) ?? true;

    public void Execute(object parameter) => _execute(parameter);
}
```

## 7. WPF 컨트롤

### 7.1 기본 컨트롤

*   **Button**: 클릭 가능한 버튼
*   **TextBox**: 텍스트 입력
*   **TextBlock**: 텍스트 표시
*   **Label**: 레이블
*   **CheckBox**: 체크박스
*   **RadioButton**: 라디오 버튼
*   **ComboBox**: 드롭다운 목록
*   **ListBox**: 목록

### 7.2 데이터 표시 컨트롤

*   **DataGrid**: 테이블 형식의 데이터 표시
*   **ListView**: 목록 뷰
*   **TreeView**: 트리 구조 표시

### 7.3 레이아웃 컨트롤

*   **Grid**: 행과 열로 구성된 레이아웃
*   **StackPanel**: 수직 또는 수평으로 요소 배치
*   **DockPanel**: 가장자리에 요소 배치
*   **Canvas**: 절대 위치 지정
*   **WrapPanel**: 자동 줄바꿈

## 8. 레이아웃 시스템

WPF의 레이아웃 시스템은 상대적 위치 지정을 기반으로 합니다:

```xml
<Grid>
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="*"/>
        <ColumnDefinition Width="*"/>
    </Grid.ColumnDefinitions>
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="*"/>
    </Grid.RowDefinitions>

    <TextBlock Grid.Row="0" Grid.Column="0" Text="Name"/>
    <TextBox Grid.Row="0" Grid.Column="1"/>
    
    <TextBlock Grid.Row="1" Grid.Column="0" Text="Description"/>
    <TextBox Grid.Row="1" Grid.Column="1"/>
</Grid>
```

## 9. 스타일 및 템플릿

UI 요소의 모양을 재사용 가능하게 정의합니다:

```xml
<Window.Resources>
    <Style TargetType="Button">
        <Setter Property="Background" Value="LightBlue"/>
        <Setter Property="Padding" Value="10"/>
        <Setter Property="FontSize" Value="14"/>
    </Style>
</Window.Resources>

<Button Content="Click Me"/>
```

## 10. 애니메이션

부드러운 애니메이션 효과를 추가합니다:

```xml
<Button Name="MyButton" Content="Hover Me">
    <Button.Triggers>
        <EventTrigger RoutedEvent="MouseEnter">
            <BeginStoryboard>
                <Storyboard>
                    <DoubleAnimation
                        Storyboard.TargetProperty="(Button.Opacity)"
                        From="1" To="0.5" Duration="0:0:0.3"/>
                </Storyboard>
            </BeginStoryboard>
        </EventTrigger>
    </Button.Triggers>
</Button>
```

## 11. WPF 최적화 팁

*   **UI 트리 복잡성 최소화**: 깊게 중첩된 레이아웃 피하기
*   **가상화 사용**: 큰 목록에 `VirtualizingStackPanel` 사용
*   **데이터 바인딩 최적화**: 불필요한 바인딩 제거
*   **이미지 최적화**: 적절한 크기의 이미지 사용
*   **MVVM 패턴 준수**: 코드비하인드 최소화

## 12. WPF 애플리케이션 생성 단계

1. **Visual Studio에서 새 WPF 프로젝트 생성**
2. **XAML에서 UI 디자인**
3. **코드비하인드에서 이벤트 처리 구현**
4. **ViewModel 클래스 작성 (MVVM 패턴)**
5. **데이터 바인딩 설정**
6. **스타일 및 템플릿 적용**
7. **애니메이션 추가**
8. **테스트 및 배포**

## 13. 결론

WPF는 Windows 데스크톱 애플리케이션 개발을 위한 강력하고 유연한 프레임워크입니다. XAML과 C#의 조합, 풍부한 컨트롤, 강력한 데이터 바인딩, 그리고 MVVM 패턴 지원으로 인해 전문적이고 유지보수하기 쉬운 애플리케이션을 개발할 수 있습니다. 특히 Windows 플랫폼에서 고성능의 시각적으로 매력적인 데스크톱 애플리케이션이 필요한 경우 WPF는 최적의 선택입니다.

## References

1.  [Windows Presentation Foundation Overview - Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/desktop/wpf/overview/)
2.  [Windows Presentation Foundation documentation - Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/desktop/wpf/)
3.  [What is WPF? - GeeksforGeeks](https://www.geeksforgeeks.org/c-sharp/what-is-wpf/)
4.  [10 WPF Best Practices [2024] - PostSharp](https://postsharp.net/blog/wpf-best-practices-2024)
