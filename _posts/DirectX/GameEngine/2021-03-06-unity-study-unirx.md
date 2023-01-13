---
title: 유니티 - UniRx (Reactive Extensions for Unity)
author: Rito15
date: 2021-03-06 20:14:00 +09:00
categories: [Unity, Unity Study]
tags: [unity, csharp, unirx]
math: true
mermaid: true
---

# 개요
---

<details>
<summary>상세 내용 확인</summary>
<div markdown="1">

div 에 markdown attribute 를 1 로 
하는 이유는 div 안에서
markdown 을 사용하기 위해서 입니다.


</div>
</details>


# Observable 스트림의 생성
---


<details>
<summary>상세내용확인</summary>
<div markdown="1">

- 미리 만들어져 있는 기능들을 이용해 빠르게 스트림을 생성할 수 있다.

- [참고 : Wiki](https://github.com/neuecc/UniRx/wiki/UniRx#observable)

```cs
// Empty : OnCompleted()를 즉시 전달
Observable.Empty<Unit>()
    .Subscribe(x => Debug.Log("Next"), () => Debug.Log("Completed"));
// Return : 한 개의 메시지만 전달
Observable.Return(2.5f)
    .Subscribe(x => Debug.Log("value : " + x));
// Range(a, b) : a부터 (a + b - 1)까지 b번 OnNext()
// 5부터 14까지 10번 OnNext()
Observable.Range(5, 10)
    .Subscribe(x => Debug.Log($"Range : {x}"));
// Interval : 지정한 시간 간격마다 OnNext()
Observable.Interval(TimeSpan.FromSeconds(1))
    .Subscribe(_ => Debug.Log("Interval"));
// Timer : 지정한 시간 이후에 OnNext()
Observable.Timer(TimeSpan.FromSeconds(2))
    .Subscribe(_ => Debug.Log("Timer"));
// EveryUpdate : 매 프레임마다 OnNext()
Observable.EveryUpdate()
    .Subscribe(_ => Debug.Log("Every Update"));
// Start : 무거운 작업을 병렬로 처리할 때 사용된다.
//         멀티스레딩으로 동작한다.
Debug.Log($"Frame : {Time.frameCount}");
Observable.Start(() =>
{
    Thread.Sleep(TimeSpan.FromMilliseconds(2000));
    MainThreadDispatcher.Post(_ => Debug.Log($"Frame : {Time.frameCount}"), new object());
    return Thread.CurrentThread.ManagedThreadId;
})
    .Subscribe(
        id => Debug.Log($"Finished : {id}"),
        err => Debug.Log(err)
    );
```


</div>
</details>



## 1. Observable 팩토리 메소드(생성 연산자)

<details>
<summary markdown="span"> 

</summary>

- 미리 만들어져 있는 기능들을 이용해 빠르게 스트림을 생성할 수 있다.

- [참고 : Wiki](https://github.com/neuecc/UniRx/wiki/UniRx#observable)

```cs
// Empty : OnCompleted()를 즉시 전달
Observable.Empty<Unit>()
    .Subscribe(x => Debug.Log("Next"), () => Debug.Log("Completed"));
// Return : 한 개의 메시지만 전달
Observable.Return(2.5f)
    .Subscribe(x => Debug.Log("value : " + x));
// Range(a, b) : a부터 (a + b - 1)까지 b번 OnNext()
// 5부터 14까지 10번 OnNext()
Observable.Range(5, 10)
    .Subscribe(x => Debug.Log($"Range : {x}"));
// Interval : 지정한 시간 간격마다 OnNext()
Observable.Interval(TimeSpan.FromSeconds(1))
    .Subscribe(_ => Debug.Log("Interval"));
// Timer : 지정한 시간 이후에 OnNext()
Observable.Timer(TimeSpan.FromSeconds(2))
    .Subscribe(_ => Debug.Log("Timer"));
// EveryUpdate : 매 프레임마다 OnNext()
Observable.EveryUpdate()
    .Subscribe(_ => Debug.Log("Every Update"));
// Start : 무거운 작업을 병렬로 처리할 때 사용된다.
//         멀티스레딩으로 동작한다.
Debug.Log($"Frame : {Time.frameCount}");
Observable.Start(() =>
{
    Thread.Sleep(TimeSpan.FromMilliseconds(2000));
    MainThreadDispatcher.Post(_ => Debug.Log($"Frame : {Time.frameCount}"), new object());
    return Thread.CurrentThread.ManagedThreadId;
})
    .Subscribe(
        id => Debug.Log($"Finished : {id}"),
        err => Debug.Log(err)
    );
```

</details>

<br>

## 2. UniRx.Triggers

<details>
<summary markdown="span"> 

</summary>

- `using UniRx.Triggers;` 필요

- 유니티의 모노비헤이비어 콜백 메소드들을 스트림으로 빠르게 변환하여 사용할 수 있다.

- 이를 활용하여 콜백 메소드를 완전히 대체할 수 있다.

- [참고 : UniRx.Triggers 위키](https://github.com/neuecc/UniRx/wiki/UniRx.Triggers)

```cs
// 필드 값을 매 프레임 조건 없이 출력
this.UpdateAsObservable()
    .Select(_ => this._intValue)
    .Subscribe(x => Debug.Log(x));
```

</details>

<br>

