# 1장 Application과 Window

## SayHello

- 콘솔 애플리케이션 프로젝트 생성
- 어셈블리 참조 추가
	- PresentationFramework
	- PresentationCore
	- WindowsBase

```csharp
namespace SayHello
{
    public class Program
    {
        [STAThread]
        static void Main(string[] args)
        {
            Window win = new Window();
            win.Title = "Say Hello";
            win.Show();

            Application app = new Application();
            app.Run();
        }
    }
}
```
### Window
- `Window` 객체는 STA 에서 생성되야함
- SayHello 프로그램은 `Window` 객체를 생성하는 것으로 시작한다
- `Window.Show` 메소드는 화면에 창을 나타나게 한다

### Application
- `Run` 메소드는 메시지 루프를 생성하고 이를 통해 키보드나 마우스로 사용자 입력을 받을 수 있게 된다. 
---
- 프로그램 실행시 콘솔창이 실행되는데 프로젝트 출력 형식을 `Windows 애플리케이션` 으로 변경시 콘솔창이 뜨지 않는다

### 클래스 계층
```
- Object
	- DispatcherObject (abstract)
		- Application
- Object
	- DispatcherObject (abstract)
		- Visual (abstract)
			- UIElement
				- FrameworkElement
					- Control
						- ContentControl
							- Window
```
- 한 프로그램은 하나의 Application을 생성할 수 있고 프로그램 내 다른 부분에서도 항상 접근 가능하다. 
- Application 객체는 보이지 않는 반면에 Window 객체는 화면에 표시된다
 
 ## HandleAnEvent
 
```csharp
namespace HandleAnEvent
{
    public class Program
    {
        [STAThread]
        public static void Main(string[] args)
        {
            Application app = new Application();

            Window win = new Window();
            win.Title = "Handle An Event";
            win.MouseDown += WindowOnMouseDown;

            app.Run(win);
        }

        private static void WindowOnMouseDown(object sender, MouseButtonEventArgs e)
        {
            Window win = sender as Window;
            string strMessage =
                string.Format("Window clicked with {0} button at point ({1})", 
                                    e.ChangedButton, e.GetPosition(win));
            MessageBox.Show(strMessage, win.Title);
        }
    }
}         string.Format("Window clicked with {0} button at point ({1})", 
                                    e.ChangedButton, e.GetPosition(win));
            MessageBox.Show(strMessage);
        }
    }
}
```
-  `MouseEventArgs.GetPosition` 메소드는 값으로 넘긴 객체의 좌측 상단을 기준으로 한 마우스 위치를 반환한다
- `MessageBox` 클래스는 `System.Windows` 네임스페이스에 정의되어 있고 다양한 옵션을 제공한다
- `Application.Current.MainWindow` 로 Windows 객체를 얻을 수도 있다.

## ThrowWindowParty
```csharp
namespace ThrowWindowParty
{
    public class ThrowWindowParty : Application
    {
        [STAThread]
        public static void Main(string[] args)
        {
            ThrowWindowParty app = new ThrowWindowParty();
            app.Run();
        }

        protected override void OnStartup(StartupEventArgs e)
        {
            Window winMain = new Window();
            winMain.Title = "Main Window";
            winMain.MouseDown += WindowOnMouseDown;
            winMain.Show();

            for (int i = 0; i < 2; i++)
            {
                Window win = new Window();
                win.Title = "Extra Window No. " + (i + 1);
                win.Show();
            }
        }

        private void WindowOnMouseDown(object sender, MouseButtonEventArgs e)
        {
            Window win = new Window();
            win.Title = "Modal Dialog Box";
            win.ShowDialog();
        }
    }
}

```
- 모든 창이 닫혀야 `Run` 메소드가 반환된다
- `Application.ShutdownMode`를 `ShutdownMode.OnMainWindowClose`로 설정하면 MainWindow가 닫힐때 프로그램을 종료(`Run` 메소드 반환)할 수 있다.
- ShutdownMode
	- OnLastWindowClose : 마지막 창이 닫힐때 종료
    - OnMainWindowClose : MainWindow가 닫힐때 종료 
    - OnExplicitShutdown : `Application.Shutdown` 명시적으로 호출시에만 종료
- `win.Owner = winMain` 으로 winMain이 종료될때 win도 종료되도록 설정함
	- 소유자를 닫으면 소유단 창이 닫히는 모달리스 대화창이 된다
	- 모달리스 창과 소유자 창 모두 동작하지만 항상 모달리스 창이 앞에 표시된다  
### Application
- `WindowCollection` 타입의 `Windows` 프로퍼티가 있다.
- 각각의 Window는 작업표시줄에 표시되는데 `win.ShowInTaskBar = false;`로 안보이게 할 수 있다.

### Window
- FrameworkElement에서 double형 Width, Height 프로퍼티를 상속 받는다.
- 기본값은 double.NaN으로 실제 크기를 얻기 위해서는 ActualWidth, ActualHeight 프로퍼티를 사용해야하며 실제 화면에 표시되고 실제 값을 가지게 된다.
- Window.TopMost : 모든 창보다 앞에 나타남
	- 오직 한 창만 맨 앞에 나타나므로 주의

### 독립적 픽셀
- WPF에서는 모든 크기나 위치를 지정할때 장치 독립적 픽셀(device-independent pixcles) 또는 논리 픽셀(logical pixels)라는 단위를 사용한다.
- 1/96 인치이며 96dpi일때 실제 픽셀 값과 동일하다
- 1/96 * DPI 값
- 해상도에 상관없이 동일한 크기로 보여줄 수 있음
- `Parameters.PrimaryScreenWidth` `Parameters.PrimaryScreenHeight`는 화면 크기를 논리 픽셀 단위로 나타낸다
- `SystemParameters.WorkArea` 는 작업표시줄 영역을 `Rect` 타입으로 반환한다

### 각종 옵션
- `WindowsStartUpLocation` 열거형으로 화면 띄우는 위치를 바꿀 수 있다
	- Manual : 기본값
	- CenterScreen : 화면 가운데
	- CenterOwner : 소유창 중앙에 위치시킴
- WindowStyle
	- None : 제목 표시줄이 없음
	- SingleBorderWindow : 기본값
	- ToolWindow : 작은 닫기 버튼만 있음
- ResizeMode
	- CanResize : 창 크기 조절 가능, 최대화 최소화 가능
	- CanResizeWithGrip : 우측 하단에 크기 조절 표시 생김
	- CanMinimize : 최소화만 가능
	- NoResize
- WindowState : 창이 최초로 표시되는 방식
	- Normala
	- Minimized
	- Maximized
