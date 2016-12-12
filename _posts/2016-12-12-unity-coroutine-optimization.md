---
layout: post
title: "유니티 코루틴 최적화"
description: "YieldInstruction을 캐싱해서 코루틴에서 사용하는 yield 구문을 최적화하자"
date: 2016-12-12
tags: [unity, c#]
comments: true
share: true
---

Unity 스크립팅을 해봤다면 코루틴을 모르는 사람이 없다고 생각될 정도로 매우 중요한 기능 중 하나이다.
C# 6.0의 async, await와 비슷한 느낌으로 루틴을 단계적으로 실행시킬 수 있으며, 게임처럼 시간의 흐름이 중요한 로직을 작성할 때 코루틴은 비록 비동기는 아니지만 매우 강력하면서도 심플하게 코딩할 수 있도록 도와준다.

# Garbage

그러나 코루틴은 생각보다 쓰레기 메모리(가비지)를 자주 생성하며, 이는 가비지 컬렉터의 좋은 먹잇감이 된다.
코루틴을 자주 사용하는 로직에서 이를 최적화 하지 않는다면 GC의 collect에서 프레임이 상당히 떨어질 수 있을 것이다.

코루틴에서 가비지가 생성되는 주요한 부분으로는 두가지가 존재한다.

* StartCoroutine
* YieldInstruction

## StartCoroutine?
StartCoroutine 메서드를 호출하는 순간 유니티는 해당 코루틴을 관리하기 위해 엔진 내부에서 인스턴스가 생성되며, 이는 가비지 컬렉터의 먹이가 된다.
때문에 StartCoroutine을 최소한으로 사용해야 그나마 적은 가비지를 생성하게 될 것이다.

StartCoroutine 자체는 유니티 엔진 내부의 코드이기 때문에 이를 최적화 하는 것은 불가능하며, 그나마 최적화를 한다면 직접 코루틴 기능을 제작하는 방법 외엔 없다.
비슷한 기능을 제공하는 에셋으로는 [More Effective Coroutine](https://www.assetstore.unity3d.com/en/#!/content/68480) 라는 것이 존재한다.

## YieldInstruction
YieldInstruction은 코루틴 내부에서 yield 구문에 사용되는 값이다.
크게 3가지를 사용한다.

* WaitForEndOfFrame
* WaitForFixedUpdate
* WaitForSeconds

YieldInstruction은 코루틴에서 아래와 같이 사용한다.

```csharp
yield return new WaitForEndOfFrame();
```

yield 구문 자체는 가비지를 생성하지 않지만, YieldInstruction을 new를 통해 인스턴스를 생성해서 사용해야 하므로 이 때 가비지를 생성하게 된다.
때문에 YieldInstruction을 캐싱만 하더라도 가비지가 생기지 않을 것이다.

```csharp
internal static class YieldInstructionCache
{
    public static readonly WaitForEndOfFrame WaitForEndOfFrame = new WaitForEndOfFrame();
    public static readonly WaitForFixedUpdate WaitForFixedUpdate = new WaitForFixedUpdate();
}

// Usage.
yield return YieldInstructionCache.WaitForEndOfFrame;
yield return YieldInstructionCache.WaitForFixedUpdate;
```

### WaitForSeconds
다만 WaitForSeconds의 경우 Seconds라는 float값에 따라 인스턴스가 달라지기 때문에 이를 캐싱하기 위해선 좀 더 복잡한 구조가 필요하다.
가장 간단하게 캐싱하는 방법으로는 아래와 같다.

```csharp
var wait = new WaitForSeconds(0.1f);
for (int i = 0; i < 100; i++)
{
    // Some Routine.
    yield return wait;
}
```

다만 이 방법은 고정적인 시간을 yield하는 것을 알고 있을 때 가능한 것이며, 게임을 제작하다 보면 그렇지 않은 상황이 자주 생길 것이다.
때문에 좀 더 스마트한 방법으로 캐싱하기 위해선 `System.Collection.Generic` 네임스페이스에 존재하는 Dictionary와 IEqualityComparer를 통해 다음과 같이 구현해야 할 것이다.

```csharp
internal static class YieldInstructionCache
{
    class FloatComparer : IEqualityComparer<float>
    {
        bool IEqualityComparer<float>.Equals (float x, float y)
        {
            return x == y;
        }
        int IEqualityComparer<float>.GetHashCode (float obj)
        {
            return obj.GetHashCode();
        }
    }

    public static readonly WaitForEndOfFrame WaitForEndOfFrame = new WaitForEndOfFrame();
    public static readonly WaitForFixedUpdate WaitForFixedUpdate = new WaitForFixedUpdate();

    private static readonly Dictionary<float, WaitForSeconds> _timeInterval = new Dictionary<float, WaitForSeconds>(new FloatComparer());

    public static WaitForSeconds WaitForSeconds(float seconds)
    {
        WaitForSeconds wfs;
        if (!_timeInterval.TryGetValue(seconds, out wfs))
            _timeInterval.Add(seconds, wfs = new WaitForSeconds(seconds));
        return wfs;
    }
}
```

```csharp
// Usage.
yield return YieldInstructionCache.WaitForSeconds(0.1f);
yield return YieldInstructionCache.WaitForSeconds(seconds);
```

Seconds 값마다 WaitForSeconds 인스턴스를 Dictionary에 캐싱하는 방법이기 때문에 가비지가 아예 발생하지 않는 것은 아니지만, `yield return new WaitForSeconds(seconds)`를 직접 하는 것보단 훨씬 적은 가비지가 생성될 것이며 훨씬 빠를 것이다.
