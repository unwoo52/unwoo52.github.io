---
title: Team Project 스토리보드 Buff 기능
author: unwoo52
date: 2022-11-08 00:00:00 +09:00
categories: [Unity, TeamProj]
tags: [Unity, TeamProj, Team, Buff, StoryBoard]
---

# 버프 생성 스토리보드

## 일반 버프

![imagename](/assets/image/Project/TeamProject/BuffStoryBoard/001.png)


<details>
<summary>testBuffPenal</summary>
<div markdown="1">


```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class testBuffPenal : MonoBehaviour
{
    [SerializeField] LayerMask MaskPlayer;
    private void OnCollisionEnter(Collision collision)
    {
        if (((1 << collision.gameObject.layer) & MaskPlayer) != 0)
        {
            OnBuff();
        }
    }

    void OnBuff()
    {
        List<string> buffTypenameList = new() { "MoveSpeed", "MineDelay_Mining" };
        List<float> buffValueList = new() { 0.5f, -0.7f };
        BuffManagerScript.instance.CreateBuff(buffTypenameList, buffValueList, 5.0f, Resources.Load("BuffImage/14_Summon", typeof(Sprite)) as Sprite);
    }
}

```

</div>
</details>

버프 테스트를 위해 밟으면 버프가 생기는 테스트 발판과 발판의 스크립트를 구현하였다.

- testBuffPenal::OnBuff
```cs
List<string> buffTypenameList = new() { "MoveSpeed", "MineDelay_Mining" };
List<float> buffValueList = new() { 0.5f, -0.7f };
BuffManagerScript.instance.CreateBuff(buffTypenameList, buffValueList, 5.0f, Resources.Load("BuffImage/14_Summon", typeof(Sprite)) as Sprite);
```

원하는 버프를 만들기 위해 버프 종류 이름(이동속도, 채광속도)을 갖는 List<string> buffTypenameList에 원하는 버프 이름을 넣는다.

> List<string> buffTypenameList의 string을 추후에 enum 혹은 단순 int값으로 바꾸어 최적화를 할 수 있음. 
<br>







## 지형 버프

----------
