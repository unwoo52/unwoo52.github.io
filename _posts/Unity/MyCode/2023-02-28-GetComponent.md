---
title: GetComponent
author: unwoo52
date: 2023-02-28 00:00:00 +09:00
categories: [Unity, MyCode]
tags: [Unity, MyCode, Static]
---

## GetCom\<T\>

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class StaticTool : MonoBehaviour
{
    public static bool GetCom<T>(GameObject gameObject, ref T t)
    {
        t = gameObject.GetComponent<T>();
        if (t == null) return false;
        return true;
    }

}
```

>이 코드는 컴포넌트를 가져오는 함수가 static으로 선언되어 있기 때문에, 해당 코드에서는 this를 통해 오브젝트의 인스턴스를 전달받을 수 없습니다.
>
>따라서 이 코드에서 반드시 GameObject를 인자로 전달하여야 합니다.


## 사용 예

```cs
    private void Start()
    {
        if(!Initialize()) 
            Debug.LogError("ERROR! 컴포넌트를 찾지 못했습니다.");
    }

    private bool Initialize()
    {
        if (!StaticTool.GetCom<AnimMovement>(gameObject, ref _animMovement)) return false;
        if (!StaticTool.GetCom<ZombieFSM_Idle>(gameObject, ref _zombieFSM_Idle)) return false;
        if (!StaticTool.GetCom<ZombieFSM_Tracking>(gameObject, ref _zombieFSM_Tracking)) return false;

        return true;
    }
```