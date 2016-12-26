---
layout: post
title: "Unity5의 GUI 클래스에 추가된 Scope에 대해서"
description: "GUI를 그리는 Begin/End 를 using 문을 사용하여 IDisposable 개체로 표현할 수 있게 됐다."
date: 2016-12-26
tags: [unity, c#]
comments: true
share: true
---

# Scope?
Unity에 내장된 GUI를 이용하는 코드(주로 에디터 관련)를 작성한 적 있다면 GUILayout 클래스의 [BeginHorizontal()](https://docs.unity3d.com/ScriptReference/GUILayout.BeginHorizontal.html), [EndHorizontal()](https://docs.unity3d.com/ScriptReference/GUILayout.EndHorizontal.html)의 늪을 꼭 한 번쯤 경험했을 것이라 생각한다.
Horizontal을 시작하고 끝내는 것, 그래 좋다.
엔진에선 Horizontal이나 Vertical 같은 것이 언제 시작하고 언제 끝나는지 알아야 할 필요가 있다고 생각한다.

그런데 왜 Begin을 하고 End로 꼭 끝을 내줘야 할까?
가끔 Begin을 하고 End를 하지 않아서 콘솔창에 경고가 가득 뜨는 경험을 한 번 해봤다면, 정말 불편하기 짝이 없는 구조라 느낄 것이다.

그래서 Unity5에는 GUI 클래스에 Scope라는 기능이 추가되었다.

## 사용법
![](https://docs.unity3d.com/StaticFiles/ScriptRefImages/BeginEndHorizontalExample.png)

지금까지는 위와 같은 GUI를 표시하기 위해선 아래와 같은 코드를 작성해야 했다.

```csharp
void OnGUI()
{
    EditorGUILayout.BeginHorizontal("Button"))

    if (GUI.Button (h.rect, GUIContent.none))
        Debug.Log ("Go here");
    GUILayout.Label ("I'm inside the button");
    GUILayout.Label ("So am I");

    EditorGUILayout.EndHorizontal();
}
```

정말 불편한 사용법이다.
복잡한 GUI 구조라면 언제 Begin되고 언제 End가 되는지 알기 위해선 코드를 한참 들여다 보아야 하며,
Begin을 해놓고 End를 하지 않게 되는 실수도 발생할 수 있다.

그러나 Unity5부터 아래와 같이 사용할 수 있게 됐다.

```csharp
void OnGUI()
{
    using (var h = new EditorGUILayout.HorizontalScope("Button"))
    {
        if (GUI.Button (h.rect, GUIContent.none))
            Debug.Log ("Go here");
        GUILayout.Label ("I'm inside the button");
        GUILayout.Label ("So am I");
    }
}
```

아주 심플하다!

using 구문 안에서 자동으로 Begin Horizontal이 시작되며, 구문이 끝나면 자동으로 End Horizontal으로 끝나게 된다.
가독성도 상승할 뿐더러, EndHorizontal()을 까먹고 호출하지 않게 되는 실수도 줄일 수 있다.

## 좀 더 자세히 알아보기

