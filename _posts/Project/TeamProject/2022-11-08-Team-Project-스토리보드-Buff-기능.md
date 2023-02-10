---
title: Team Project 스토리보드 Buff 기능
author: unwoo52
date: 2022-11-08 00:00:00 +09:00
categories: [Unity, TeamProj]
tags: [Unity, TeamProj, Team, Buff, StoryBoard]
---

# 버프 생성 스토리보드

## 일반 버프

### 버프 생성하기




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

<br>


**********

- [BuffManagerScript](https://unwoo52.github.io/posts/Team-Project-%EA%B8%B0%EC%88%A0%EB%AC%B8%EC%84%9C-Buff-%EA%B8%B0%EB%8A%A5/#buffmanagerscript)::OnBuff
```cs
    List<string> buffTypenameList = new() { "MoveSpeed", "MineDelay_Mining" };
    List<float> buffValueList = new() { 0.5f, -0.7f };

    BuffManagerScript.instance.CreateBuff(buffTypenameList, buffValueList, 5.0f, Resources.Load("BuffImage/14_Summon", typeof(Sprite)) as Sprite);
```

원하는 버프를 만들기 위해 버프 종류 이름(이동속도, 채광속도)을 갖는 List<string> buffTypenameList에 원하는 버프 이름을 넣는다.

> List<string> buffTypenameList의 string을 enum 혹은 단순 int값으로 바꾸어 최적화를 할 수 있음. 

정의한 buffTypenameList와 buffValueList을 토대로 [BuffManager](https://unwoo52.github.io/posts/Team-Project-%EA%B8%B0%EC%88%A0%EB%AC%B8%EC%84%9C-Buff-%EA%B8%B0%EB%8A%A5/#buffmanagerscript)의 CreateBuff()을 실행하여 버프를 생성한다.

<br>

***********


### 버프 Initialize - 1.BuffManagerScript

- BuffManagerScript::CreateBuff
```cs
    public void CreateBuff(List<string> buffTypename, List<float> buffValue, float buffOriginTime, Sprite bufficon)
    {
        GameObject gameObject = Instantiate(buffPrefab, transform);
        gameObject.GetComponent<BaseBuff>().Init(buffTypename, buffValue, buffOriginTime);
        gameObject.GetComponent<Image>().sprite = bufficon;
    }
```

BuffManagerScript의 buffPrefab은 버프 오브젝트 프레펩[BaseBuff](https://unwoo52.github.io/posts/Team-Project-%EA%B8%B0%EC%88%A0%EB%AC%B8%EC%84%9C-Buff-%EA%B8%B0%EB%8A%A5/#basebuff)이 저장되어 있다. 버프 오브젝트를 생성한 후, 만들어진 버프 오브젝트의 Init()을 실행하고 아이콘을 지정한다.

<br>

*************

### 버프 Initialize - 2.BaseBuff

- [BaseBuff](https://unwoo52.github.io/posts/Team-Project-%EA%B8%B0%EC%88%A0%EB%AC%B8%EC%84%9C-Buff-%EA%B8%B0%EB%8A%A5/#basebuff)::Init

```cs
    public void Init(List<string> buffTypenameList, List<float> buffValueList, float buffOriginTime)
    {
        this.buffTypenameList = buffTypenameList;
        this.buffValueList = buffValueList;
        this.buffOriginTime = buffOriginTime;
        currentTime = this.buffOriginTime;
        icon.fillAmount = 1f;

        PlayerIBuff = PlayerScript.instance.GetComponent<IBuff>();
        NomalBuffactivation();
    }
```

인자로 받은 값들을 버프 오브젝트에 대입한 후, 버프 효과 적용 함수 NomalBuffactivation()을 실행한다.

<br>

- [BaseBuff](https://unwoo52.github.io/posts/Team-Project-%EA%B8%B0%EC%88%A0%EB%AC%B8%EC%84%9C-Buff-%EA%B8%B0%EB%8A%A5/#basebuff)::NomalBuffactivation

```cs
    private void NomalBuffactivation()
    {
        PlayerIBuff.BuffListAdd(this);
        PlayerIBuff.ChooseBuff(buffTypenameList);
        StartCoroutine(Activation());
    }
```


NomalBuffactivation에서는<br> **1.플레이어의 IBuff 함수들을 실행해 플레이어의 스탯을 수정하고** <br> **2.코루틴을 실행해 버프 지속시간을 감소시키고 버프 아이콘의 fillAmount을 수정한다**

<details>
<summary>코루틴 코드 내용</summary>
<div markdown="1">

```cs
    IEnumerator Activation()
    {
        while (currentTime > 0)
        {
            icon.fillAmount = currentTime / buffOriginTime;
            //Buff Root
            currentTime -= 0.1f;
            yield return BuffCheckRootSecond;
        }
        icon.fillAmount = 0f;
        currentTime = 0f;
        BuffDeActivation();
    }
```

</div>
</details>

<br>

*************

### 플레이어 스탯에게 버프 효과 적용

```cs
      PlayerIBuff.BuffListAdd(this);
      PlayerIBuff.ChooseBuff(buffTypenameList);
```

NomalBuffactivation 함수의 코드에서 위의 코드를 실행하였다.

<br>

```cs
      public void BuffListAdd(BaseBuff baseBuff)
      {
          BuffList.Add(baseBuff);
      }
```

PlayerIBuff.BuffListAdd(this) 코드는 플레이어의 버프 리스트에 버프 오브젝트를 추가하는 코드이다.

<br>

- [PlayerScriptIBuff](https://unwoo52.github.io/posts/Team-Project-%EA%B8%B0%EC%88%A0%EB%AC%B8%EC%84%9C-Buff-%EA%B8%B0%EB%8A%A5/#playerscript-ibuff)::ChooseBuff

```cs
        public void ChooseBuff(List<string> buffTypenameList)//리스트[1]에 0.miningDelay와 1.movespeed를 받아왔다면 miningDelay와 movespeed에 대한 BuffEffectApply를 실행
        {
            foreach (string s in buffTypenameList)
            {
                switch (s)
                {
                    case "MineDelay_Mining":
                        myInfo.MineDelay_Mining_AfterBuff = BuffEffectAplly(s, myInfo.MineDelay_Mining_Origin);
                        break;
                    case "MineDelay_Picking":
                        myInfo.MineDelay_Picking_AfterBuff = BuffEffectAplly(s, myInfo.MineDelay_Picking_Origin);
                        break;
                    case "MoveSpeed":
                        myInfo.MoveSpeed_AfterBuff = BuffEffectAplly(s, myInfo.MoveSpeed_Origin);
                        break;
                }
            }
        }
```

PlayerIBuff.ChooseBuff(buffTypenameList)는 BuffEffectAplly에게 입력받은 버프 유형 타입을 입력해 실행하는 코드이다.

테스트 발판이 갖고 있는 버프 타입 이름은 **"MoveSpeed", "MineDelay_Mining"** 두가지이다. 그러므로 이 ChooseBuff()는 BuffEffectAplly(s, myInfo.**MineDelay_Mining_Origin**);과 BuffEffectAplly(s, myInfo.**MoveSpeed_Origin**);를 실행한다.

BuffEffectAplly는 플레이어의 스탯을 수정하는 함수이다.

<br>

******

- [PlayerScriptIBuff](https://unwoo52.github.io/posts/Team-Project-%EA%B8%B0%EC%88%A0%EB%AC%B8%EC%84%9C-Buff-%EA%B8%B0%EB%8A%A5/#playerscript-ibuff)::BuffEffectAplly

```cs
        public float BuffEffectAplly(string buffTypeName, float origin)
        {
            if (BuffList.Count > 0)
            {
                float temp = 0;
                for (int i = 0; i < BuffList.Count; i++)
                {
                    for (int j = 0; j < BuffList[i].BuffTypenameList.Count; j++)
                    {
                        if (BuffList[i].BuffTypenameList[j].Equals(buffTypeName)) 
                            temp += origin * BuffList[i].BuffValueList[j];
                    }
                }
                return origin + temp;
            }
            else return origin;
        }
```

### 플레이어 버프 적용


## 지형 버프

----------
