UniRx - Reactive Extensions for Unity
===
Created by Yoshifumi Kawai(neuecc)

[![CircleCI](https://circleci.com/gh/neuecc/UniRx.svg?style=svg)](https://circleci.com/gh/neuecc/UniRx)  
[![Become as Backer](https://opencollective.com/unirx/tiers/backer.svg?avatarHeight=50)](https://opencollective.com/unirx) [![Become as Sponsor](https://opencollective.com/unirx/tiers/sponsor.svg?avatarHeight=50)](https://opencollective.com/unirx)

What is UniRx?
---
UniRx(Unity용 리액티브 익스텐션)는 .NET 리액티브 익스텐션을 재구현한 것입니다. 공식 Rx 구현은 훌륭하지만 Unity에서 작동하지 않으며 iOS IL2CPP 호환성 문제가 있습니다. 이 라이브러리는 이러한 문제를 해결하고 Unity를 위한 몇 가지 특정 유틸리티를 추가합니다. 지원되는 플랫폼은 PC/Mac/Android/iOS/WebGL/WindowsStore/기타 라이브러리입니다.

UniRx는 Unity 에셋 스토어에서 무료로 다운로드할 수 있습니다. - http://u3d.as/content/neuecc/uni-rx-reactive-extensions-for-unity/7tT

업데이트 정보 블로그 - https://medium.com/@neuecc

Unity 포럼의 지원 스레드: 질문하기 - http://forum.unity3d.com/threads/248535-UniRx-Reactive-Extensions-for-Unity

릴리스 노트, [UniRx/releases](https://github.com/neuecc/UniRx/releases) 참조

UniRx는 코어 라이브러리(Port of Rx) + 플랫폼 어댑터(MainThreadScheduler/FromCoroutine/etc) + 프레임워크(ObservableTriggers/ReactiveProeperty/etc)로 구성됩니다.

> 참고: 비동기/대기 통합(UniRx.Async)은 버전 7.0 이후 [Cysharp/UniTask](https://github.com/Cysharp/UniTask)로 분리되어 있습니다.

Why Rx?
---
일반적으로 Unity에서 네트워크 작업을 하려면 `WWW`와 `Coroutine`을 사용해야 합니다. 하지만 다음과 같은 이유로 비동기 연산에는 `Coroutine`을 사용하지 않는 것이 좋습니다:

1. Coroutine은 반환 유형이 IEnumerator여야 하므로 어떤 값도 반환할 수 없습니다.
2. 반환 문을 try-catch 구조로 둘러쌀 수 없기 때문에 Coroutine은 예외를 처리할 수 없습니다.

이러한 종류의 컴포저빌리티 부족으로 인해 연산이 밀접하게 결합되어 거대한 모놀리식 IEnumerator가 생성되는 경우가 많습니다.

Rx는 이러한 "비동기 블루스"를 해결합니다. Rx는 관찰 가능한 컬렉션과 LINQ 스타일의 쿼리 연산자를 사용하여 비동기 및 이벤트 기반 프로그램을 작성하기 위한 라이브러리입니다. 
  
게임 루프(모든 업데이트, OnCollisionEnter 등), 센서 데이터(키넥트, 도약 모션, VR 입력 등)는 모두 이벤트 유형에 속합니다. Rx는 이벤트를 반응형 시퀀스로 표현하며, 이는 쉽게 구성할 수 있고 LINQ 쿼리 연산자를 사용하여 시간 기반 연산을 지원합니다.

Unity는 일반적으로 싱글 스레드이지만 UniRx는 조인, 취소, 게임 오브젝트 액세스 등을 위한 멀티스레딩을 지원합니다.

UniRx는 uGUI로 UI 프로그래밍을 지원합니다. 모든 UI 이벤트(클릭, 값 변경 등)를 UniRx 이벤트 스트림으로 변환할 수 있습니다. 

유니티는 C# 업그레이드를 통해 2017년부터 비동기/대기 기능을 지원하며, UniRx 제품군 프로젝트는 더욱 가볍고 강력한 비동기/대기 기능을 유니티와 통합합니다. Cysharp/UniTask](https://github.com/Cysharp/UniTask)를 참조하세요.

Introduction
---
Rx에 대한 훌륭한 소개 글입니다:
[당신이 놓치고 있던 반응형 프로그래밍 입문](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754).다음 코드는 이 글의 더블 클릭 감지 예제를 UniRx로 구현한 것입니다:

```csharp
var clickStream = Observable.EveryUpdate()
    .Where(_ => Input.GetMouseButtonDown(0));

clickStream.Buffer(clickStream.Throttle(TimeSpan.FromMilliseconds(250)))
    .Where(xs => xs.Count >= 2)
    .Subscribe(xs => Debug.Log("DoubleClick Detected! Count:" + xs.Count));
```

이 예시에서는 다음 기능을 5줄로 설명합니다:

* 이벤트 스트림으로서의 게임 루프(업데이트)
* 컴포저블 이벤트 스트림* 자체 스트림 병합
* 시간 기반 작업의 손쉬운 처리   

네트워크 작업
---
비동기 네트워크 작업에는 ObservableWWW를 사용하세요. 이 함수의 Get/Post 함수는 구독 가능한 IObservables를 반환합니다:

```csharp
ObservableWWW.Get("http://google.co.jp/")
    .Subscribe(
        x => Debug.Log(x.Substring(0, 100)), // onSuccess
        ex => Debug.LogException(ex)); // onError
```

Rx는 구성 및 취소가 가능합니다. LINQ 표현식으로 쿼리할 수도 있습니다:

```csharp
// composing asynchronous sequence with LINQ query expressions
var query = from google in ObservableWWW.Get("http://google.com/")
            from bing in ObservableWWW.Get("http://bing.com/")
            from unknown in ObservableWWW.Get(google + bing)
            select new { google, bing, unknown };

var cancel = query.Subscribe(x => Debug.Log(x));

// Call Dispose is cancel.
cancel.Dispose();
```

병렬 요청에는 Observable.WhenAll을 사용합니다:

```csharp
// Observable.WhenAll is for parallel asynchronous operation
// (It's like Observable.Zip but specialized for single async operations like Task.WhenAll)
var parallel = Observable.WhenAll(
    ObservableWWW.Get("http://google.com/"),
    ObservableWWW.Get("http://bing.com/"),
    ObservableWWW.Get("http://unity3d.com/"));

parallel.Subscribe(xs =>
{
    Debug.Log(xs[0].Substring(0, 100)); // google
    Debug.Log(xs[1].Substring(0, 100)); // bing
    Debug.Log(xs[2].Substring(0, 100)); // unity
});
```

진행률 정보를 확인할 수 있습니다:

```csharp
// notifier for progress use ScheduledNotifier or new Progress<float>(/* action */)
var progressNotifier = new ScheduledNotifier<float>();
progressNotifier.Subscribe(x => Debug.Log(x)); // write www.progress

// pass notifier to WWW.Get/Post
ObservableWWW.Get("http://google.com/", progress: progressNotifier).Subscribe();
```

오류 처리:

```csharp
// If WWW has .error, ObservableWWW throws WWWErrorException to onError pipeline.
// WWWErrorException has RawErrorMessage, HasResponse, StatusCode, ResponseHeaders
ObservableWWW.Get("http://www.google.com/404")
    .CatchIgnore((WWWErrorException ex) =>
    {
        Debug.Log(ex.RawErrorMessage);
        if (ex.HasResponse)
        {
            Debug.Log(ex.StatusCode);
        }
        foreach (var item in ex.ResponseHeaders)
        {
            Debug.Log(item.Key + ":" + item.Value);
        }
    })
    .Subscribe();
```

IEnumerator(Coroutines)와 함께 사용
---
IEnumerator(Coroutine)는 유니티의 기본 비동기 툴입니다. UniRx는 Coroutine과 IObservables를 통합합니다. Coroutine으로 비동기 코드를 작성하고 UniRx를 사용하여 오케스트레이션할 수 있습니다. 이는 비동기 흐름을 제어하는 가장 좋은 방법입니다.

```csharp
// two coroutines

IEnumerator AsyncA()
{
    Debug.Log("a start");
    yield return new WaitForSeconds(1);
    Debug.Log("a end");
}

IEnumerator AsyncB()
{
    Debug.Log("b start");
    yield return new WaitForEndOfFrame();
    Debug.Log("b end");
}

// main code
// Observable.FromCoroutine converts IEnumerator to Observable<Unit>.
// You can also use the shorthand, AsyncA().ToObservable()
        
// after AsyncA completes, run AsyncB as a continuous routine.
// UniRx expands SelectMany(IEnumerator) as SelectMany(IEnumerator.ToObservable())
var cancel = Observable.FromCoroutine(AsyncA)
    .SelectMany(AsyncB)
    .Subscribe();

// you can stop a coroutine by calling your subscription's Dispose.
cancel.Dispose();
```

Unity 5.3에서는 Coroutine에 옵저버블에 ToYieldInstruction을 사용할 수 있습니다.

```csharp
IEnumerator TestNewCustomYieldInstruction()
{
    // wait Rx Observable.
    yield return Observable.Timer(TimeSpan.FromSeconds(1)).ToYieldInstruction();

    // you can change the scheduler(this is ignore Time.scale)
    yield return Observable.Timer(TimeSpan.FromSeconds(1), Scheduler.MainThreadIgnoreTimeScale).ToYieldInstruction();

    // get return value from ObservableYieldInstruction
    var o = ObservableWWW.Get("http://unity3d.com/").ToYieldInstruction(throwOnError: false);
    yield return o;

    if (o.HasError) { Debug.Log(o.Error.ToString()); }
    if (o.HasResult) { Debug.Log(o.Result); }

    // other sample(wait until transform.position.y >= 100) 
    yield return this.transform.ObserveEveryValueChanged(x => x.position).FirstOrDefault(p => p.y >= 100).ToYieldInstruction();
}
```
일반적으로 값을 반환하기 위해 Coroutine이 필요할 때는 콜백을 사용해야 합니다. Observable.FromCoroutine은 대신 Coroutine을 취소 가능한 IObservable[T]로 변환할 수 있습니다.

```csharp
// public method
public static IObservable<string> GetWWW(string url)
{
    // convert coroutine to IObservable
    return Observable.FromCoroutine<string>((observer, cancellationToken) => GetWWWCore(url, observer, cancellationToken));
}

// IObserver is a callback publisher
// Note: IObserver's basic scheme is "OnNext* (OnError | Oncompleted)?" 
static IEnumerator GetWWWCore(string url, IObserver<string> observer, CancellationToken cancellationToken)
{
    var www = new UnityEngine.WWW(url);
    while (!www.isDone && !cancellationToken.IsCancellationRequested)
    {
        yield return null;
    }

    if (cancellationToken.IsCancellationRequested) yield break;

    if (www.error != null)
    {
        observer.OnError(new Exception(www.error));
    }
    else
    {
        observer.OnNext(www.text);
        observer.OnCompleted(); // IObserver needs OnCompleted after OnNext!
    }
}
```

다음은 몇 가지 예시입니다. 다음은 다중 OnNext 패턴입니다.

```csharp
public static IObservable<float> ToObservable(this UnityEngine.AsyncOperation asyncOperation)
{
    if (asyncOperation == null) throw new ArgumentNullException("asyncOperation");

    return Observable.FromCoroutine<float>((observer, cancellationToken) => RunAsyncOperation(asyncOperation, observer, cancellationToken));
}

static IEnumerator RunAsyncOperation(UnityEngine.AsyncOperation asyncOperation, IObserver<float> observer, CancellationToken cancellationToken)
{
    while (!asyncOperation.isDone && !cancellationToken.IsCancellationRequested)
    {
        observer.OnNext(asyncOperation.progress);
        yield return null;
    }
    if (!cancellationToken.IsCancellationRequested)
    {
        observer.OnNext(asyncOperation.progress); // push 100%
        observer.OnCompleted();
    }
}

// usecase
Application.LoadLevelAsync("testscene")
    .ToObservable()
    .Do(x => Debug.Log(x)) // output progress
    .Last() // last sequence is load completed
    .Subscribe();
```

멀티스레딩에 사용
---

```csharp
// Observable.Start is start factory methods on specified scheduler
// default is on ThreadPool
var heavyMethod = Observable.Start(() =>
{
    // heavy method...
    System.Threading.Thread.Sleep(TimeSpan.FromSeconds(1));
    return 10;
});

var heavyMethod2 = Observable.Start(() =>
{
    // heavy method...
    System.Threading.Thread.Sleep(TimeSpan.FromSeconds(3));
    return 10;
});

// Join and await two other thread values
Observable.WhenAll(heavyMethod, heavyMethod2)
    .ObserveOnMainThread() // return to main thread
    .Subscribe(xs =>
    {
        // Unity can't touch GameObject from other thread
        // but use ObserveOnMainThread, you can touch GameObject naturally.
        (GameObject.Find("myGuiText")).guiText.text = xs[0] + ":" + xs[1];
    }); 
```

DefaultScheduler
---
UniRx의 기본 시간 기반 연산(Interval, Timer, Buffer(timeSpan) 등)은 스케줄러로 `Scheduler.MainThread`를 사용합니다. 즉, 대부분의 연산자(`Observable.Start` 제외)는 단일 스레드에서 작동하므로 ObserverOn이 필요하지 않고 스레드 안전 조치를 무시할 수 있습니다. 이는 표준 RxNet 구현과는 다르지만 Unity 환경에 더 적합합니다.  

스케줄러.메인 스레드`는 시간 척도의 영향을 받아 실행됩니다. 시간 척도를 무시하려면 대신 `Scheduler.MainThreadIgnoreTimeScale`을 사용합니다.

MonoBehaviour triggers
---
UniRx는 `UniRx.Triggers`로 모노비헤이비어 이벤트를 처리할 수 있습니다:

```csharp
using UniRx;
using UniRx.Triggers; // need UniRx.Triggers namespace

public class MyComponent : MonoBehaviour
{
    void Start()
    {
        // Get the plain object
        var cube = GameObject.CreatePrimitive(PrimitiveType.Cube);

        // Add ObservableXxxTrigger for handle MonoBehaviour's event as Observable
        cube.AddComponent<ObservableUpdateTrigger>()
            .UpdateAsObservable()
            .SampleFrame(30)
            .Subscribe(x => Debug.Log("cube"), () => Debug.Log("destroy"));

        // destroy after 3 second:)
        GameObject.Destroy(cube, 3f);
    }
}
```

지원되는 트리거는 [UniRx.위키#UniRx.트리거](https://github.com/neuecc/UniRx/wiki#unirxtriggers)에 나열되어 있습니다.

또한 컴포넌트/게임 오브젝트의 확장 메서드가 반환하는 옵저버블을 직접 구독하면 더 쉽게 처리할 수 있습니다. 이 메서드들은 ObservableTrigger를 자동으로 삽입합니다(`ObservableEventTrigger` 및 `ObservableStateMachineTrigger`는 제외):

```csharp
using UniRx;
using UniRx.Triggers; // need UniRx.Triggers namespace for extend gameObejct

public class DragAndDropOnce : MonoBehaviour
{
    void Start()
    {
        // All events can subscribe by ***AsObservable
        this.OnMouseDownAsObservable()
            .SelectMany(_ => this.UpdateAsObservable())
            .TakeUntil(this.OnMouseUpAsObservable())
            .Select(_ => Input.mousePosition)
            .Subscribe(x => Debug.Log(x));
    }
}
```

> 이전 버전의 UniRx는 `ObservableMonoBehaviour`를 제공했습니다. 이 인터페이스는 더 이상 지원되지 않는 레거시 인터페이스입니다. 대신 UniRx.Triggers를 사용하세요.

사용자 지정 트리거 만들기
---
관찰 가능으로 변환하는 것이 Unity 이벤트를 처리하는 가장 좋은 방법입니다. UniRx에서 제공하는 표준 트리거가 충분하지 않은 경우 커스텀 트리거를 생성할 수 있습니다. 예를 들어, 다음은 uGUI용 LongTap 트리거입니다:

```csharp
public class ObservableLongPointerDownTrigger : ObservableTriggerBase, IPointerDownHandler, IPointerUpHandler
{
    public float IntervalSecond = 1f;

    Subject<Unit> onLongPointerDown;

    float? raiseTime;

    void Update()
    {
        if (raiseTime != null && raiseTime <= Time.realtimeSinceStartup)
        {
            if (onLongPointerDown != null) onLongPointerDown.OnNext(Unit.Default);
            raiseTime = null;
        }
    }

    void IPointerDownHandler.OnPointerDown(PointerEventData eventData)
    {
        raiseTime = Time.realtimeSinceStartup + IntervalSecond;
    }

    void IPointerUpHandler.OnPointerUp(PointerEventData eventData)
    {
        raiseTime = null;
    }

    public IObservable<Unit> OnLongPointerDownAsObservable()
    {
        return onLongPointerDown ?? (onLongPointerDown = new Subject<Unit>());
    }

    protected override void RaiseOnCompletedOnDestroy()
    {
        if (onLongPointerDown != null)
        {
            onLongPointerDown.OnCompleted();
        }
    }
}
```

표준 트리거처럼 쉽게 사용할 수 있습니다:

```csharp
var trigger = button.AddComponent<ObservableLongPointerDownTrigger>();

trigger.OnLongPointerDownAsObservable().Subscribe();
```

Observable 수명 주기 관리
---
OnCompleted는 언제 호출되나요? 구독 라이프사이클 관리는 UniRx를 사용할 때 고려해야 할 매우 중요한 사항입니다. ObservableTriggers는 연결된 게임 오브젝트가 소멸될 때 OnCompleted를 호출합니다. 다른 정적 제너레이터 메서드(`Observable.Timer`, `Observable.EveryUpdate` 등)는 자동으로 중지되지 않으므로 해당 구독을 수동으로 관리해야 합니다.

Rx는 여러 구독을 한 번에 폐기할 수 있는 `IDisposable.AddTo`와 같은 몇 가지 도우미 메서드를 제공합니다:

```csharp
// CompositeDisposable is similar with List<IDisposable>, manage multiple IDisposable
CompositeDisposable disposables = new CompositeDisposable(); // field

void Start()
{
    Observable.EveryUpdate().Subscribe(x => Debug.Log(x)).AddTo(disposables);
}

void OnTriggerEnter(Collider other)
{
    // .Clear() => Dispose is called for all inner disposables, and the list is cleared.
    // .Dispose() => Dispose is called for all inner disposables, and Dispose is called immediately after additional Adds.
    disposables.Clear();
}
```

게임 오브젝트가 소멸될 때 자동으로 폐기하려면 AddTo(게임 오브젝트/컴포넌트)를 사용하면 됩니다:

```csharp
void Start()
{
    Observable.IntervalFrame(30).Subscribe(x => Debug.Log(x)).AddTo(this);
}
```

AddTo 호출은 자동 처리를 용이하게 합니다. 그러나 파이프라인에서 특별한 OnCompleted 처리가 필요한 경우 `TakeWhile`, `TakeUntil`, `TakeUntilDestroy` 및 `TakeUntilDisable`을 대신 사용하세요:

```csharp
Observable.IntervalFrame(30).TakeUntilDisable(this)
    .Subscribe(x => Debug.Log(x), () => Debug.Log("completed!"));
```

이벤트를 처리할 때 `Repeat`는 중요하지만 위험한 메서드입니다. 무한 루프를 유발할 수 있으므로 주의해서 다뤄야 합니다:

```csharp
using UniRx;
using UniRx.Triggers;

public class DangerousDragAndDrop : MonoBehaviour
{
    void Start()
    {
        this.gameObject.OnMouseDownAsObservable()
            .SelectMany(_ => this.gameObject.UpdateAsObservable())
            .TakeUntil(this.gameObject.OnMouseUpAsObservable())
            .Select(_ => Input.mousePosition)
            .Repeat() // dangerous!!! Repeat cause infinite repeat subscribe at GameObject was destroyed.(If in UnityEditor, Editor is freezed)
            .Subscribe(x => Debug.Log(x));
    }
}
```

UniRx는 안전한 반복 메서드를 추가로 제공합니다. 'RepeatSafe': 연속된 "OnComplete"가 반복 중지를 호출하는 경우. RepeatUntilDestroy(게임 오브젝트/컴포넌트)`, `RepeatUntilDisable(게임 오브젝트/컴포넌트)`는 대상 게임 오브젝트가 파괴되었을 때 중지할 수 있습니다:

```csharp
this.gameObject.OnMouseDownAsObservable()
    .SelectMany(_ => this.gameObject.UpdateAsObservable())
    .TakeUntil(this.gameObject.OnMouseUpAsObservable())
    .Select(_ => Input.mousePosition)
    .RepeatUntilDestroy(this) // safety way
    .Subscribe(x => Debug.Log(x));            
```

UniRx는 처리되지 않은 예외 내구성을 가진 핫 옵저버블(FromEvent/Subject/ReactiveProperty/UnityUI.AsObservable..., 같은 이벤트가 있음)의 내구성을 보장합니다. 무슨 문제인가요? 구독에서 구독하면 이벤트가 분리되지 않습니다.

```csharp
button.OnClickAsObservable().Subscribe(_ =>
{
    // If throws error in inner subscribe, but doesn't detached OnClick event.
    ObservableWWW.Get("htttp://error/").Subscribe(x =>
    {
        Debug.Log(x);
    });
});
```

이 동작은 때때로 사용자 이벤트 처리와 같이 유용합니다.


모든 클래스 인스턴스는 매 프레임마다 값의 변화를 감시하는 `ObserveEveryValueChanged` 메서드를 제공합니다:

```csharp
// watch position change
this.transform.ObserveEveryValueChanged(x => x.position).Subscribe(x => Debug.Log(x));
```

매우 유용합니다. 감시 대상이 게임 오브젝트인 경우, 대상이 파괴되면 관찰을 중지하고 OnCompleted를 호출합니다. 감시 대상이 일반 C# 오브젝트인 경우 GC에서 OnCompleted가 호출됩니다.

Unity 콜백을 IObservables로 변환하기
---
Subject(또는 비동기 작업의 경우 AsyncSubject)를 사용합니다:

```csharp
public class LogCallback
{
    public string Condition;
    public string StackTrace;
    public UnityEngine.LogType LogType;
}

public static class LogHelper
{
    static Subject<LogCallback> subject;

    public static IObservable<LogCallback> LogCallbackAsObservable()
    {
        if (subject == null)
        {
            subject = new Subject<LogCallback>();

            // Publish to Subject in callback
            UnityEngine.Application.RegisterLogCallback((condition, stackTrace, type) =>
            {
                subject.OnNext(new LogCallback { Condition = condition, StackTrace = stackTrace, LogType = type });
            });
        }

        return subject.AsObservable();
    }
}

// method is separatable and composable
LogHelper.LogCallbackAsObservable()
    .Where(x => x.LogType == LogType.Warning)
    .Subscribe();

LogHelper.LogCallbackAsObservable()
    .Where(x => x.LogType == LogType.Error)
    .Subscribe();
```

Unity5에서는 `Application.RegisterLogCallback`이 `Application.logMessageReceived` 대신 제거되었으므로 이제 `Observable.FromEvent`를 간단히 사용할 수 있습니다.

```csharp
public static IObservable<LogCallback> LogCallbackAsObservable()
{
    return Observable.FromEvent<Application.LogCallback, LogCallback>(
        h => (condition, stackTrace, type) => h(new LogCallback { Condition = condition, StackTrace = stackTrace, LogType = type }),
        h => Application.logMessageReceived += h, h => Application.logMessageReceived -= h);
}
```

Stream Logger
---
```csharp
// using UniRx.Diagnostics;

// logger is threadsafe, define per class with name.
static readonly Logger logger = new Logger("Sample11");

// call once at applicationinit
public static void ApplicationInitialize()
{
    // Log as Stream, UniRx.Diagnostics.ObservableLogger.Listener is IObservable<LogEntry>
    // You can subscribe and output to any place.
    ObservableLogger.Listener.LogToUnityDebug();

    // for example, filter only Exception and upload to web.
    // (make custom sink(IObserver<EventEntry>) is better to use)
    ObservableLogger.Listener
        .Where(x => x.LogType == LogType.Exception)
        .Subscribe(x =>
        {
            // ObservableWWW.Post("", null).Subscribe();
        });
}

// Debug is write only DebugBuild.
logger.Debug("Debug Message");

// or other logging methods
logger.Log("Message");
logger.Exception(new Exception("test exception"));
```

디버깅
---
UniRx.Diagnostics` 네임스페이스의 `Debug` 연산자는 디버깅에 도움이 됩니다.

```csharp
// needs Diagnostics using
using UniRx.Diagnostics;

---

// [DebugDump, Normal]OnSubscribe
// [DebugDump, Normal]OnNext(1)
// [DebugDump, Normal]OnNext(10)
// [DebugDump, Normal]OnCompleted()
{
    var subject = new Subject<int>();

    subject.Debug("DebugDump, Normal").Subscribe();

    subject.OnNext(1);
    subject.OnNext(10);
    subject.OnCompleted();
}

// [DebugDump, Cancel]OnSubscribe
// [DebugDump, Cancel]OnNext(1)
// [DebugDump, Cancel]OnCancel
{
    var subject = new Subject<int>();

    var d = subject.Debug("DebugDump, Cancel").Subscribe();

    subject.OnNext(1);
    d.Dispose();
}

// [DebugDump, Error]OnSubscribe
// [DebugDump, Error]OnNext(1)
// [DebugDump, Error]OnError(System.Exception)
{
    var subject = new Subject<int>();

    subject.Debug("DebugDump, Error").Subscribe();

    subject.OnNext(1);
    subject.OnError(new Exception());
}
```

`OnNext`, `OnError`, `OnCompleted`, `OnCancel`, `OnSubscribe` 타이밍의 시퀀스 요소를 Debug.Log에 표시합니다. 이 옵션은 `#if DEBUG`만 활성화합니다.

유니티 전용 추가 보석
---
```csharp
// Unity's singleton UiThread Queue Scheduler
Scheduler.MainThreadScheduler 
ObserveOnMainThread()/SubscribeOnMainThread()

// Global StartCoroutine runner
MainThreadDispatcher.StartCoroutine(enumerator)

// convert Coroutine to IObservable
Observable.FromCoroutine((observer, token) => enumerator(observer, token)); 

// convert IObservable to Coroutine
yield return Observable.Range(1, 10).ToYieldInstruction(); // after Unity 5.3, before can use StartAsCoroutine()

// Lifetime hooks
Observable.EveryApplicationPause();
Observable.EveryApplicationFocus();
Observable.OnceApplicationQuit();
```

Framecount-based time operators
---
UniRx는 몇 가지 프레임 수 기반 시간 연산자를 제공합니다:

Method | 
-------|
EveryUpdate|
EveryFixedUpdate|
EveryEndOfFrame|
EveryGameObjectUpdate|
EveryLateUpdate|
ObserveOnMainThread|
NextFrame|
IntervalFrame|
TimerFrame|
DelayFrame|
SampleFrame|
ThrottleFrame|
ThrottleFirstFrame|
TimeoutFrame|
DelayFrameSubscription|
FrameInterval|
FrameTimeInterval|
BatchFrame|

예를 들어, 한 번 지연 호출합니다:

```csharp
Observable.TimerFrame(100).Subscribe(_ => Debug.Log("after 100 frame"));
```

모든* 메서드의 실행 순서는 다음과 같습니다.

```
EveryGameObjectUpdate(in MainThreadDispatcher's Execution Order) ->
EveryUpdate -> 
EveryLateUpdate -> 
EveryEndOfFrame
```

호출자가 MainThreadDisatcher.Update보다 먼저 호출된 경우 동일한 프레임에서 호출되는 EveryGameObjectUpdate(다른 것보다 먼저 호출된 MainThreadDisatcher를 권장합니다(스크립트 실행 순서가 -32000이 되게 함).      
EveryLateUpdate, EveryEndOfFrame은 같은 프레임에서 호출합니다.  
EveryUpdate, 다음 프레임에서 호출합니다.  

MicroCoroutine
---
마이크로코루틴은 메모리 효율적이고 빠른 코루틴 워커입니다. 이 구현은 [Unity blog's 10000 UPDATE() CALLS](http://blogs.unity3d.com/2015/12/23/1k-update-calls/)를 기반으로 하며, 관리-비관리 오버헤드를 피하여 10배 빠른 반복을 구현합니다. 마이크로 코루틴은 프레임 카운트 기반 시간 연산자 및 ObserveEveryValueChanged에 자동으로 사용됩니다.

표준 유니티 코루틴 대신 MicroCoroutine 을 사용하려면 `MainThreadDispatcher.StartUpdateMicroCoroutine` 또는 `Observable.FromMicroCoroutine`을 사용하세요.

```csharp
int counter;

IEnumerator Worker()
{
    while(true)
    {
        counter++;
        yield return null;
    }
}

void Start()
{
    for(var i = 0; i < 10000; i++)
    {
        // fast, memory efficient
        MainThreadDispatcher.StartUpdateMicroCoroutine(Worker());

        // slow...
        // StartCoroutine(Worker());
    }
}
```

![image](https://cloud.githubusercontent.com/assets/46207/15267997/86e9ed5c-1a0c-11e6-8371-14b61a09c72c.png)

MicroCoroutine의 한계로 인해 `yield return null`만 지원하며 업데이트 타이밍은 시작 메서드(`StartUpdateMicroCoroutine`, `StartFixedUpdateMicroCoroutine`, `StartEndOfFrameMicroCoroutine`)를 통해 결정됩니다. 

다른 IObservable과 결합하면 isDone과 같은 완료된 속성을 확인할 수 있습니다.

```csharp
IEnumerator MicroCoroutineWithToYieldInstruction()
{
    var www = ObservableWWW.Get("http://aaa").ToYieldInstruction();
    while (!www.IsDone)
    {
        yield return null;
    }

    if (www.HasResult)
    {
        UnityEngine.Debug.Log(www.Result);
    }
}
```

uGUI 통합
---
UniRx는 `UnityEvent`를 쉽게 처리할 수 있습니다. 이벤트를 구독하려면 `UnityEvent.AsObservable`을 사용하세요:

```csharp
public Button MyButton;
// ---
MyButton.onClick.AsObservable().Subscribe(_ => Debug.Log("clicked"));
```

이벤트를 옵저버블로 처리하면 선언적 UI 프로그래밍이 가능합니다. 

```csharp
public Toggle MyToggle;
public InputField MyInput;
public Text MyText;
public Slider MySlider;

// On Start, you can write reactive rules for declaretive/reactive ui programming
void Start()
{
    // Toggle, Input etc as Observable (OnValueChangedAsObservable is a helper providing isOn value on subscribe)
    // SubscribeToInteractable is an Extension Method, same as .interactable = x)
    MyToggle.OnValueChangedAsObservable().SubscribeToInteractable(MyButton);
    
    // Input is displayed after a 1 second delay
    MyInput.OnValueChangedAsObservable()
        .Where(x => x != null)
        .Delay(TimeSpan.FromSeconds(1))
        .SubscribeToText(MyText); // SubscribeToText is helper for subscribe to text
    
    // Converting for human readability
    MySlider.OnValueChangedAsObservable()
        .SubscribeToText(MyText, x => Math.Round(x, 2).ToString());
}
```

반응형 UI 프로그래밍에 대한 자세한 내용은 Sample12, Sample13 및 아래의 ReactiveProperty 섹션을 참조하세요. 

ReactiveProperty, ReactiveCollection
---
게임 데이터에는 종종 알림이 필요합니다. 프로퍼티와 이벤트(콜백)를 사용해야 할까요? 너무 복잡할 때가 많습니다. 유니알엑스는 가벼운 프로퍼티 브로커인 리액티브프로퍼티를 제공합니다.

```csharp
// Reactive Notification Model
public class Enemy
{
    public ReactiveProperty<long> CurrentHp { get; private set; }

    public ReactiveProperty<bool> IsDead { get; private set; }

    public Enemy(int initialHp)
    {
        // Declarative Property
        CurrentHp = new ReactiveProperty<long>(initialHp);
        IsDead = CurrentHp.Select(x => x <= 0).ToReactiveProperty();
    }
}

// ---
// onclick, HP decrement
MyButton.OnClickAsObservable().Subscribe(_ => enemy.CurrentHp.Value -= 99);
// subscribe from notification model.
enemy.CurrentHp.SubscribeToText(MyText);
enemy.IsDead.Where(isDead => isDead == true)
    .Subscribe(_ =>
    {
        MyButton.interactable = false;
    });
```

ReactiveProperties, ReactiveCollections 및 UnityEvent.AsObservable이 반환하는 옵저버블을 결합할 수 있습니다. 모든 UI 요소는 관찰 가능합니다.

일반 ReactiveProperties는 Unity 에디터에서 직렬화하거나 인스펙터블할 수 없지만, UniRx는 직렬화할 수 있는 ReactiveProperty의 특수 서브클래스를 제공합니다. 여기에는 Int/LongReactiveProperty, Float/DoubleReactiveProperty, StringReactiveProperty, BoolReactiveProperty 등의 클래스가 포함됩니다(여기에서 찾아보기): [InspectableReactiveProperty.cs](https://github.com/neuecc/UniRx/blob/master/Assets/Plugins/UniRx/Scripts/UnityEngineBridge/InspectableReactiveProperty.cs)). 모두 인스펙터에서 완전히 편집할 수 있습니다. 커스텀 Enum ReactiveProperty의 경우, 커스텀 검사 가능한 ReactiveProperty[T]를 쉽게 작성할 수 있습니다.

ReactiveProperty에 `[Multiline]` 또는 `[Range]` 어태치먼트가 필요한 경우 `Multiline`과 `Range` 대신 `MultilineReactivePropertyAttribute`와 `RangeReactivePropertyAttribute`를 사용할 수 있습니다.

제공된 파생된 InpsectableReactiveProperties는 인스펙터에 자연스럽게 표시되며, 인스펙터에서 값이 변경되더라도 변경된 값을 알려줍니다.

![](StoreDocument/RxPropInspector.png)

이 기능은 [InspectorDisplayDrawer](https://github.com/neuecc/UniRx/blob/master/Assets/Plugins/UniRx/Scripts/UnityEngineBridge/InspectorDisplayDrawer.cs)에서 제공합니다. 이를 상속하여 자신만의 특화된 ReactiveProperties를 제공할 수 있습니다:

```csharp
public enum Fruit
{
    Apple, Grape
}

[Serializable]
public class FruitReactiveProperty : ReactiveProperty<Fruit>
{
    public FruitReactiveProperty()
    {
    }

    public FruitReactiveProperty(Fruit initialValue)
        :base(initialValue)
    {
    }
}

[UnityEditor.CustomPropertyDrawer(typeof(FruitReactiveProperty))]
[UnityEditor.CustomPropertyDrawer(typeof(YourSpecializedReactiveProperty2))] // and others...
public class ExtendInspectorDisplayDrawer : InspectorDisplayDrawer
{
}
```

ReactiveProperty 값이 스트림 내에서만 업데이트되는 경우, `ReadOnlyReactiveProperty`를 사용하여 읽기 전용으로 설정할 수 있습니다.

```csharp
public class Person
{
    public ReactiveProperty<string> GivenName { get; private set; }
    public ReactiveProperty<string> FamilyName { get; private set; }
    public ReadOnlyReactiveProperty<string> FullName { get; private set; }

    public Person(string givenName, string familyName)
    {
        GivenName = new ReactiveProperty<string>(givenName);
        FamilyName = new ReactiveProperty<string>(familyName);
        // If change the givenName or familyName, notify with fullName!
        FullName = GivenName.CombineLatest(FamilyName, (x, y) => x + " " + y).ToReadOnlyReactiveProperty();
    }
}
```

모델 보기-(반응형) 발표자 패턴
---
UniRx를 사용하면 MVP(MVRP) 패턴을 구현할 수 있습니다.

![](StoreDocument/MVP_Pattern.png)

MVVM 대신 MVP를 사용해야 하는 이유는 무엇인가요? Unity는 UI 바인딩 메커니즘을 제공하지 않으며, 바인딩 레이어를 생성하는 것은 너무 복잡하고 손실이 발생하며 성능에 영향을 미칩니다. 그럼에도 불구하고 뷰는 업데이트가 필요합니다. 발표자는 뷰의 컴포넌트를 알고 있으며 이를 업데이트할 수 있습니다. 실제 바인딩은 없지만 Observables를 사용하면 알림을 구독할 수 있으므로 실제와 매우 유사하게 작동할 수 있습니다. 이 패턴을 반응형 발표자라고 합니다: 

```csharp
// Presenter for scene(canvas) root.
public class ReactivePresenter : MonoBehaviour
{
    // Presenter is aware of its View (binded in the inspector)
    public Button MyButton;
    public Toggle MyToggle;
    
    // State-Change-Events from Model by ReactiveProperty
    Enemy enemy = new Enemy(1000);

    void Start()
    {
        // Rx supplies user events from Views and Models in a reactive manner 
        MyButton.OnClickAsObservable().Subscribe(_ => enemy.CurrentHp.Value -= 99);
        MyToggle.OnValueChangedAsObservable().SubscribeToInteractable(MyButton);

        // Models notify Presenters via Rx, and Presenters update their views
        enemy.CurrentHp.SubscribeToText(MyText);
        enemy.IsDead.Where(isDead => isDead == true)
            .Subscribe(_ =>
            {
                MyToggle.interactable = MyButton.interactable = false;
            });
    }
}

// The Model. All property notify when their values change
public class Enemy
{
    public ReactiveProperty<long> CurrentHp { get; private set; }

    public ReactiveProperty<bool> IsDead { get; private set; }

    public Enemy(int initialHp)
    {
        // Declarative Property
        CurrentHp = new ReactiveProperty<long>(initialHp);
        IsDead = CurrentHp.Select(x => x <= 0).ToReactiveProperty();
    }
}
```

뷰는 씬, 즉 Unity 계층 구조입니다. 뷰는 초기화 시 Unity 엔진에 의해 프레젠터와 연결됩니다. XxxAsObservable 메서드를 사용하면 오버헤드 없이 이벤트 신호를 간단하게 생성할 수 있습니다. SubscribeToText와 SubscribeToInteractable은 간단한 바인딩과 유사한 헬퍼입니다. 단순한 툴이지만 매우 강력합니다. Unity 환경에서 자연스럽게 느껴지며 고성능과 깔끔한 아키텍처를 제공합니다.

![](StoreDocument/MVRP_Loop.png)

V -> RP -> M -> RP -> V가 리액티브 방식으로 완전히 연결됩니다. UniRx는 모든 어댑터 메서드와 클래스를 제공하지만, 다른 MVVM(또는 MV*) 프레임워크를 대신 사용할 수 있습니다. UniRx/ReactiveProperty는 단순한 툴킷일 뿐입니다. 

옵저버블트리거는 GUI 프로그래밍에도 유용합니다. 옵저버블트리거는 Unity 이벤트를 옵저버블로 변환하므로 이를 사용하여 MV(R)P 패턴을 구성할 수 있습니다. 예를 들어 `ObservableEventTrigger`는 uGUI 이벤트를 옵저버블로 변환합니다:

```csharp
var eventTrigger = this.gameObject.AddComponent<ObservableEventTrigger>();
eventTrigger.OnBeginDragAsObservable()
    .SelectMany(_ => eventTrigger.OnDragAsObservable(), (start, current) => UniRx.Tuple.Create(start, current))
    .TakeUntil(eventTrigger.OnEndDragAsObservable())
    .RepeatUntilDestroy(this)
    .Subscribe(x => Debug.Log(x));
```

(Obsolete)PresenterBase
---
> 참고:
> PresenterBase는 충분히 작동하지만 너무 복잡합니다.  
> 간단한 `Initialize` 메서드를 사용하고 부모에서 자식으로 호출하면 대부분의 시나리오에서 작동합니다.  
> 그래서 저는 `PresenterBase`를 사용하지 않는 것이 좋습니다.   

리액티브 커맨드, 비동기 리액티브 커맨드
----
상호작용이 가능한 부울을 사용한 버튼 명령의 ReactiveCommand 추상화.
             
```csharp
public class Player
{		
   public ReactiveProperty<int> Hp;		
   public ReactiveCommand Resurrect;		
		
   public Player()
   {		
        Hp = new ReactiveProperty<int>(1000);		
        		
        // If dead, can not execute.		
        Resurrect = Hp.Select(x => x <= 0).ToReactiveCommand();		
        // Execute when clicked		
        Resurrect.Subscribe(_ =>		
        {		
             Hp.Value = 1000;		
        }); 		
    }		
}		
		
public class Presenter : MonoBehaviour		
{		
    public Button resurrectButton;		
		
    Player player;		
		
    void Start()
    {		
      player = new Player();		
		
      // If Hp <= 0, can't press button.		
      player.Resurrect.BindTo(resurrectButton);		
    }		
}		
```		
		
비동기 실행이 완료될 때까지 `CanExecute`(대부분의 경우 버튼의 인터랙티브에 바인딩)가 거짓으로 변경되는 ReactiveCommand의 변형인 AsyncReactiveCommand입니다.
		
```csharp		
public class Presenter : MonoBehaviour		
{		
    public UnityEngine.UI.Button button;		
		
    void Start()
    {		
        var command = new AsyncReactiveCommand();		
		
        command.Subscribe(_ =>		
        {		
            // heavy, heavy, heavy method....		
            return Observable.Timer(TimeSpan.FromSeconds(3)).AsUnitObservable();		
        });		
		
        // after clicked, button shows disable for 3 seconds		
        command.BindTo(button);		
		
        // Note:shortcut extension, bind aync onclick directly		
        button.BindToOnClick(_ =>		
        {		
            return Observable.Timer(TimeSpan.FromSeconds(3)).AsUnitObservable();		
        });		
    }		
}		
```

AsyncReactiveCommand`에는 세 개의 생성자가 있습니다.

* `()` - 비동기 실행이 완료될 때까지 CanExecute가 false로 변경됩니다.
* `(IObservable<bool> canExecuteSource)` - empty와 혼합된 경우, canExecuteSource가 참으로 전송하고 실행하지 않으면 CanExecute가 참이 됩니다. 
* `(IReactiveProperty<bool> sharedCanExecute)` - 여러 AsyncReactiveCommand 간에 실행 상태를 공유하면, 하나의 AsyncReactiveCommand가 실행 중이면 비동기 실행이 완료될 때까지 다른 AsyncReactiveCommand(동일한 sharedCanExecute 속성을 가진)는 CanExecute가 거짓이 됩니다.

```csharp
public class Presenter : MonoBehaviour
{
    public UnityEngine.UI.Button button1;
    public UnityEngine.UI.Button button2;

    void Start()
    {
        // share canExecute status.
        // when clicked button1, button1 and button2 was disabled for 3 seconds.

        var sharedCanExecute = new ReactiveProperty<bool>();

        button1.BindToOnClick(sharedCanExecute, _ =>
        {
            return Observable.Timer(TimeSpan.FromSeconds(3)).AsUnitObservable();
        });

        button2.BindToOnClick(sharedCanExecute, _ =>
        {
            return Observable.Timer(TimeSpan.FromSeconds(3)).AsUnitObservable();
        });
    }
}
```

메시지 브로커, 비동기 메시지 브로커
---
MessageBroker는 유형별로 필터링된 Rx 기반 인메모리 pubsub 시스템입니다.

```csharp
public class TestArgs
{
    public int Value { get; set; }
}

---

// Subscribe message on global-scope.
MessageBroker.Default.Receive<TestArgs>().Subscribe(x => UnityEngine.Debug.Log(x));

// Publish message
MessageBroker.Default.Publish(new TestArgs { Value = 1000 });
```

AsyncMessageBroker는 메시지 브로커의 변형으로, 게시 호출을 기다릴 수 있습니다.

```csharp
AsyncMessageBroker.Default.Subscribe<TestArgs>(x =>
{
    // show after 3 seconds.
    return Observable.Timer(TimeSpan.FromSeconds(3))
        .ForEachAsync(_ =>
        {
            UnityEngine.Debug.Log(x);
        });
});

AsyncMessageBroker.Default.PublishAsync(new TestArgs { Value = 3000 })
    .Subscribe(_ =>
    {
        UnityEngine.Debug.Log("called all subscriber completed");
    });
```

UniRx.Toolkit
---
`UniRx.Toolkit`에는 서버용 Rx 도구가 포함되어 있습니다. 현재 `ObjectPool`과 `AsyncObjectPool`이 포함되어 있습니다.  `Rent`, `Return`, Rent 작업 전 풀을 채우기 위한 `PreloadAsync`가 가능합니다.

```csharp
// sample class
public class Foobar : MonoBehaviour
{
    public IObservable<Unit> ActionAsync()
    {
        // heavy, heavy, action...
        return Observable.Timer(TimeSpan.FromSeconds(3)).AsUnitObservable();
    }
}

public class FoobarPool : ObjectPool<Foobar>
{
    readonly Foobar prefab;
    readonly Transform hierarchyParent;

    public FoobarPool(Foobar prefab, Transform hierarchyParent)
    {
        this.prefab = prefab;
        this.hierarchyParent = hierarchyParent;
    }

    protected override Foobar CreateInstance()
    {
        var foobar = GameObject.Instantiate<Foobar>(prefab);
        foobar.transform.SetParent(hierarchyParent);

        return foobar;
    }

    // You can overload OnBeforeRent, OnBeforeReturn, OnClear for customize action.
    // In default, OnBeforeRent = SetActive(true), OnBeforeReturn = SetActive(false)

    // protected override void OnBeforeRent(Foobar instance)
    // protected override void OnBeforeReturn(Foobar instance)
    // protected override void OnClear(Foobar instance)
}

public class Presenter : MonoBehaviour
{
    FoobarPool pool = null;

    public Foobar prefab;
    public Button rentButton;

    void Start()
    {
        pool = new FoobarPool(prefab, this.transform);

        rentButton.OnClickAsObservable().Subscribe(_ =>
        {
            var foobar = pool.Rent();
            foobar.ActionAsync().Subscribe(__ =>
            {
                // if action completed, return to pool
                pool.Return(foobar);
            });
        });
    }
}
```

Visual Studio 분석기
---
Visual Studio 2015 사용자의 경우, 사용자 지정 분석기인 UniRxAnalyzer가 제공됩니다. 예를 들어 스트림이 구독되지 않은 경우를 감지할 수 있습니다.

![](StoreDocument/AnalyzerReference.jpg)

![](StoreDocument/VSAnalyzer.jpg)

`ObservableWWW`은 구독될 때까지 실행되지 않으므로 분석기는 잘못된 사용에 대해 경고합니다. NuGet에서 다운로드할 수 있습니다.

* Install-Package [UniRxAnalyzer](http://www.nuget.org/packages/UniRxAnalyzer)

새로운 분석기 아이디어를 깃허브 이슈에 제출해 주세요!

샘플
---
[UniRx/예제](https://github.com/neuecc/UniRx/tree/master/Assets/Plugins/UniRx/Examples)를 참조하세요. 

샘플에서는 리소스 관리 방법(Sample09_EventHandling), 메인 스레드 디스패처가 무엇인지 등을 설명합니다.

윈도우 스토어/폰 앱(NETFX_CORE)
---
UniRx.IObservable<T>` 및 `System.IObservable<T>`와 같은 일부 인터페이스는 Windows 스토어 앱에 제출할 때 충돌을 일으킵니다.
따라서 NETFX_CORE를 사용할 때는 `UniRx.IObservable<T>`와 같은 구조체 사용을 자제하고 네임스페이스를 추가하지 않고 UniRx 컴포넌트를 짧은 이름으로 참조하시기 바랍니다. 이렇게 하면 충돌이 해결됩니다.

DLL 분리
---
UniRx를 미리 빌드하고 싶다면 자체적으로 dll을 빌드할 수 있습니다. 프로젝트를 복제하고 `UniRx.sln`을 열면 `UniRx`가 표시되며, 이는 UniRx의 풀셋 분리된 프로젝트입니다. 컴파일 심볼은 `UNITY;UNITY_5_4_OR_NEWER;UNITY_5_4_0;UNITY_5_4;UNITY_5;` + `UNITY_EDITOR`, `UNITY_IPHONE` 또는 기타 플랫폼 심볼로 정의해야 합니다. 컴파일 심볼이 서로 다르기 때문에 릴리스 페이지, 에셋 스토어에 사전 빌드 바이너리를 제공할 수 없습니다.


UPM 패키지
---
Unity 2019.3.4f1, Unity 2020.1a21 이후 버전부터 git 패키지의 경로 쿼리 파라미터를 지원합니다. 패키지 관리자에 `https://github.com/neuecc/UniRx.git?path=Assets/Plugins/UniRx/Scripts`을 추가하거나


또는 `"com.neuecc.unirx": "https://github.com/neuecc/UniRx.git?path=Assets/Plugins/UniRx/Scripts"`를 `패키지/매니페스트.json`에 추가합니다.


참조
---
* [UniRx/wiki](https://github.com/neuecc/UniRx/wiki)


UniRx API 문서.


* [ReactiveX](http://reactivex.io/)


리액티브X의 홈 페이지. [소개](http://reactivex.io/intro.html), [모든 연산자](http://reactivex.io/documentation/operators.html)는 그래픽 마블 다이어그램으로 설명되어 있어 쉽게 이해할 수 있습니다. 그리고 유니알엑스는 공식 [리액티브엑스 언어](http://reactivex.io/languages.html)입니다.


* [Rx 소개](http://introtorx.com/)


훌륭한 온라인 튜토리얼 및 전자책.


* [리액티브 확장 프로그램 초보자 가이드](http://msdn.microsoft.com/en-us/data/gg577611)


Rx.NET에 대한 많은 비디오, 슬라이드 및 문서.


* [Unity 프로그래밍 기술의 미래

도움말 및 기여
---
유니티 포럼의 지원 스레드 질문하기 - [http://forum.unity3d.com/threads/248535-UniRx-Reactive-Extensions-for-Unity](http://forum.unity3d.com/threads/248535-UniRx-Reactive-Extensions-for-Unity)  


후원자 되기, 후원, 일회성 기부를 환영합니다. [오픈 컬렉티브 - UniRx](https://opencollective.com/unirx/#)


버그 리포트, 요청, 풀 리퀘스트 등 모든 기여를 환영합니다.  
GitHub 이슈에 대한 보고서나 요청을 상담하고 제출해 주세요.  
소스 코드는 `Assets/Plugins/UniRx/Scripts`에서 확인할 수 있습니다.  


작성자의 다른 Unity + LINQ 에셋
---
[LINQ to GameObject](https://github.com/neuecc/LINQ-to-GameObject-for-Unity/)는 계층구조를 가로지르며 LINQ to XML처럼 게임 오브젝트를 추가할 수 있는 Unity용 게임 오브젝트 확장 그룹입니다. GitHub에서 무료로 오픈소스로 제공됩니다.


![](https://raw.githubusercontent.com/neuecc/LINQ-to-GameObject-for-Unity/master/Images/axis.jpg)


작성자 정보
---
요시후미 카와이(일명 neuecc)는 일본의 소프트웨어 개발자입니다.  
현재 컨설팅 회사 [뉴월드 주식회사](http://new-world.co/)를 설립했습니다.  
2011년부터 Visual C# 부문 Microsoft MVP를 수상했습니다.  


블로그: https://medium.com/@neuecc (영어)  
블로그: http://neue.cc/ (일본어)   
Twitter: https://twitter.com/neuecc (일본어)


라이선스
---
이 라이브러리는 MIT 라이선스 하에 있습니다.


일부 코드는 [Rx.NET](https://github.com/dotnet/reactive/) 및 [mono/mcs](https://github.com/mono/mono)에서 차용했습니다.
