---
layout: post
title: "Visual Studio Code 개인 설정"
description: "현재 사용중인 Visual Studio Code의 간단한 설정들"
date: 2016-11-21
tags: [vscode]
comments: true
share: true
---

Visual Studio Code는 현재 내가 가장 좋아하는 IDE이다.
[Electron](http://electron.atom.io) 기반이기 때문에 거의 대부분의 플랫폼에서 거의 비슷한 환경을 제공하고 있으며, 설정 또한 모든 플랫폼이 동일하다.

나같은 경우 회사에서 Unity3D 코딩을 VSCode로 사용중이며(사실 Visual Studio를 사용하고 싶지만 맥 유저라 ㅜ.ㅠ) 때문에 대부분 C#이나 Javascript를 VSCode로 코딩하고 있다.
VSCode에는 다양한 설정법들이 있지만, 일단 난 기본 옵션에도 충분히 만족하고 있으므로 내가 사용중인 옵션은 3개밖에 없다.

# 사용중인 설정
```json
{
    "editor.fontSize": 14,
    "editor.renderWhitespace": "boundary",
    "editor.referenceInfos": false
}
```

![](https://ejonghyuck.github.io/blog/assets/images/vscode-settings.png)

## editor.fontSize (int)
폰트 사이즈, 설명이 필요 없으므로 생략.

## editor.renderWhitespace (string)
Indent 등의 Whitespace를 표시해준다.

요게 중요하다고 생각하는 이유는 indent 같은 경우 tab일 수 있고, space인 경우도 있다.
무엇이 좋느냐는 개인의 취향 문제라 생각하지만, 팀 작업을 할 때에는 가급적이면 한 파일에 indent가 섞이면 안되기 때문에 현재 indent가 무엇인지는 눈으로 직접 볼 필요가 있다고 생각한다.

editor.renderWhitespace의 옵션은 string으로, 'none', 'boundary', 'all' 요렇게 3가지가 있다.
* none : whitespace 표시 안함 (default)
* boundary : indent만 표시함
* all : 모든 whitespace를 전부 다 표시함

주석에도 whitespace가 보여지면 가독성이 살짝 떨어진다고 생각하기 때문에, 나는 boundary로 설정해서 쓰고 있다.

## editor.referenceInfos (bool)
(다른 언어는 모르겠지만) C#에서 필드나 메서드, 클래스의 선언부 위에 `n references` 라고 표시해준다.

기본적으론 요게 설정되어 있는데, 난 이게 너무 싫다... 왼쪽에 라인 뜨는것도 얘 때문에 한 칸 밑으로 내려가는데다가, 굳이 이걸 코딩하면서 계속 쳐다봐야 하나 싶음.
그래서 false로 해놓고 사용하고 있다.
