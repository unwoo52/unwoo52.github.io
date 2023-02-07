---
title: Survival-Project-Improve
author: unwoo52
date: 2022-12-19 00:00:00 +09:00
categories: [Unity, TeamProj_ImprovementProject]
tags: [Unity, TeamProj_ImprovementProject]
---

## 개요

![image](/assets/image/Unity/TeamProj_ImprovementProject/TeamProj_ImprovementProject_post_001/000.png)

Survival-Project-Improve의 포스트들은 두달전에 끝난 Unity개발 팀 프로젝트를 다시 살펴보고 문제점이나 개선점을 발견하고 수정해보는 활동에 대한 포스팅이다.

이번에 작업할 부분은 팀원 중 문준범님이 개발을 담당한 부분은 Scene로드와 게임 데이터 저장 및 불러오기 등 기능이다.

<br>

<br>

## ChangeMenu Method 개선

### 기존 코드

![image](/assets/image/Unity/TeamProj_ImprovementProject/TeamProj_ImprovementProject_post_001/001.png)

```cs
    public enum MenuState
    { 
    Main, Save, Settings, KeySettings
    }

    public MenuState myMenu = MenuState.Main;
 
 ```
 ```cs

    void ChangeMenu(MenuState state)
    {
        if (myMenu == state) return;
        myMenu = state;
        switch (myMenu)
        {
            case MenuState.Main:
                foreach(GameObject n in myPause)
                {
                    n.SetActive(false);
                }
                myPause[(int)MenuState.Main].SetActive(true);
                break;
            case MenuState.Save:
                foreach (GameObject n in myPause)
                {
                    n.SetActive(false);
                }
                myPause[(int)MenuState.Save].SetActive(true);
                break;
            case MenuState.Settings:
                foreach (GameObject n in myPause)
                {
                    n.SetActive(false);
                }
                myPause[(int)MenuState.Settings].SetActive(true);
                break;
            case MenuState.KeySettings:
                foreach (GameObject n in myPause)
                {
                    n.SetActive(false);
                }
                myPause[(int)MenuState.KeySettings].SetActive(true);
                break;
        }
    }
```
#### 기능

> enum을 통해 활성화된 메뉴의 하위 오브젝트들만 SetActive하는 기능을 가진 함수.

<br>

#### 문제점들

>메뉴가 바뀔 때 마다 모든 메뉴를 foreach로 비활성화 시키고 있어서 낭비가 발생한다.

- 지금 활성화된 메뉴만을 비활성화로 바꾸게 만들 수 있다. 


```cs
foreach (GameObject n in myPause)
{
	n.SetActive(false);
}

>>>>>

myMain[(int)myMenu].SetActive(false);
```

<br>

>enum을 잘 활용하지 못하여서 불필요한 switch문이 사용되 코드가 길어졌다. 

- switch문을 제거할 수 있다.

```cs
switch (myMenu)
{
    case MenuState.Main:
        foreach(GameObject n in myPause)
        {
        n.SetActive(false);
        }
        myPause[(int)MenuState.Main].SetActive(true);
    break;
    
>>>>>

myMain[(int)myMenu].SetActive(false);
myMenu = state;
myMain[(int)state].SetActive(true);
```

<br>

### 개선 코드

```cs
    private void ChangeMenu(MenuState state)
    {
        if (myMenu == state) return;

        myMain[(int)myMenu].SetActive(false);
        myMenu = state;
        myMain[(int)state].SetActive(true);
    }
```


<br>
<br>

## Menu 부모 클래스 만들기

### 기존 구조

```cs
public class Title : MonoBehaviour
{
	private void ChangeMenu(MenuState state) ...
}
```

```cs
public class Pause : MonoBehaviour
{
	private void ChangeMenu(MenuState state) ...
}
```


#### 문제점

>메뉴 클래스마다 ChangeMenu 함수를 만들어 두어서 같은 코드가 중복되어 존재함.

- 메뉴 부모 클래스를 만들고, title화면의 메뉴와 인게임 메뉴 클래스가 상속하도록 변경해 코드 중복을 줄이고, 앞으로 개발 편의성도 개선하도록 함.

### Menu Class 생성

<br>
<br>


## 

### 기존 코드

```cs
public class Pause : MonoBehaviour
{
	...
    
    void Update()
    {
        if (Input.GetKeyDown(KeyCode.F1))
        {
            PlayerScript.instance.State.isCursorActive = !PlayerScript.instance.State.isCursorActive;
            IsActive = !IsActive;
            myCanvas.gameObject.SetActive(IsActive);
        }
    }
    
	...
```

### 기능

> F1버튼을 누르면 메뉴 오브젝트(myCanvas)를 활성화하고, 마우스 커서를 활성화 시킴.
>
> isActive는 왜 있는지 모르겠다... private값이라 외부에서 쓰이고 있지도 않고, 스크립트 안에서도 위 기능을 제외하면 어디에서도 쓰이지 않음. 제거해도 되는 값인 것 같다.

### 문제점

> Key입력이 각 기능의 각 클래스와 함수에 분산되어 있으면 찾거나 관리하기 힘들 것 같다.

- Key입력은 한 장소로 모아 관리가 용이하게 한다.

> 코드를 읽었을 때 무슨 기능을 하는지 직관적으로 알기 어렵다.

- 함수로 분리, 함수 명을 통해 어떤 기능을 하는 코드인지 유추할 수 있게 함

```cs	
    void Update()
    {
        if (Input.GetKeyDown(KeyCode.F1))
        {
            ActivateMenu();
        }
    }

    private void ActivateMenu()
    {
        PlayerScript.instance.State.isCursorActive = !PlayerScript.instance.State.isCursorActive;
        IsActive = !IsActive;
        myCanvas.gameObject.SetActive(IsActive);
    }
```

