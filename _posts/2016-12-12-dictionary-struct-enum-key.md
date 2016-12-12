---
layout: post
title: "유니티 C#에서 struct와 enum을 key로 사용하는 Dictionary의 최적화"
description: "struct와 enum을 key로 사용하는 Dictionary에서 발생할 수 있는 가비지들과, 최적화의 방법으로 IEqualityComparer를 통한 방법에 대해 알아본다."
date: 2016-12-12
tags: [unity, c#]
comments: true
share: true
---

# Garbage
일반적인 상황에서 enum과 struct는 Value-Type이기 때문에 힙 할당을 하지 않으므로 가비지가 생길 일이 존재하지 않는다.
다만 아래 2가지 경우에선 가비지가 발생하게 되는 점은 잘 알려지지 않았다.

* enum-key를 사용하는 Dictionary
* struct-key를 사용하는 Dictionary

## 왜 가비지가 발생하는가?
struct를 선언할 때 `Equals()`, `GetHashCode()` 메서드들을 구현하지 않았다면, Dictionary의 Contains 메서드를 호출할 경우 내부적으로 object로 박싱하여 `object.Equals`를 통해 비교 구문을 실행하게 된다.
이는 some_dictionary[key] 처럼 접근할 때에도 key의 비교 구문을 실행하기 때문에 내부적으로 박싱이 이뤄지게 된다.
안타깝게도 struct에는 Vector3같은 유니티의 built-in struct들도 박싱이 이뤄진다.

## 가비지가 생기지 않게 하려면?
가장 편한 방법은 `System.Collections.Generic` 네임스페이스 내부에 존재하는 IEqualityComparer<T> 인터페이스를 상속받는 클래스를 선언해서 비교하게 하는 방법이다.
Dictionary 인스턴스를 새로 생성할 때 생성자에 인스턴스를 넣어주면 된다.

```csharp
// Struct.
public struct SomeStruct
{
    public int a, b;
}

class SomeComparer : IEqualityComparer<SomeStruct>
{
    bool IEqualityComparer<SomeStruct>.Equals (SomeStruct x, SomeStruct y)
    {
        return x.a == y.a && x.b == y.b;
    }

    bool IEqualityComparer<SomeStruct>.GetHashCode (SomeStruct obj)
    {
        return obj.a ^ obj.b;
    }
}

// Usage.
Dictionary<SomeStruct, int> dic = new Dictionary<SomeStruct, int>(new SomeComparer());
```

struct 외에도 enum 역시 동일하게 적용할 수 있다.

```csharp
// Enum.
public enum SomeEnum
{
    StateA = 1,
    StateB = 2
}

class SomeComparer : IEqualityComparer<SomeEnum>
{
    bool IEqualityComparer<SomeEnum>.Equals (SomeEnum x, SomeEnum y)
    {
        return (int)x == (int)y;
    }

    bool IEqualityComparer<SomeEnum>.GetHashCode (SomeEnum obj)
    {
        return ((int)obj).GetHashCode();
    }
}

// Usage.
Dictionary<SomeEnum, int> dic = new Dictionary<SomeEnum, int>(new SomeComparer());
```
