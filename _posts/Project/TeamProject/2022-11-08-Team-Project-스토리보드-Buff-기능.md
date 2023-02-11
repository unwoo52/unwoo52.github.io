---
title: Team Project 스토리보드 Buff 기능
author: unwoo52
date: 2022-11-08 00:00:00 +09:00
categories: [Unity, TeamProj]
tags: [Unity, TeamProj, Team, Buff, StoryBoard]
---

# 버프 생성 스토리보드

## 개요

[팀프로젝트](https://unwoo52.github.io/posts/Team-Project-About/)동안 구현한 Buff System을 설명하기 위해 시간의 흐름되로 어떻게 작동하는지 사진과 코드 설명을 나열한 문서이다.

BuffSystem은 버프의 종류에 따라 [일반 버프]()(지속시간이 끝나면 사라지는 버프)와 [지형 버프]()(버프 지속시간이 없는 대신 특정 조건에 의해 발동되고 사라짐)으로 나누어진다.

## 일반 버프

### 테스트 발판 - 버프 생성하기

![imagename](/assets/image/Project/TeamProject/BuffStoryBoard/003.png)

>버프를 생성하는 발판이다. testBuffPenal코드가 들어있으며, 밟으면 버프 생성의 시작인 BuffManagerScript.instance.CreateBuff()을 실행한다.


일반 버프를 임의로 생성시켜 테스트하기 위한 [테스트 발판]()을 생성하였다.

테스트 발판을 밟으면 플레이어는 이동속도(MoveSpeed)와 채광속도(MineDelay_Mining)가 증가하는 버프를 얻게 된다.


<details>
<summary>testBuffPenal 코드</summary>
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



<br>


**********

```cs
    List<string> buffTypenameList = new() { "MoveSpeed", "MineDelay_Mining" };
    List<float> buffValueList = new() { 0.5f, -0.7f };

    BuffManagerScript.instance.CreateBuff(buffTypenameList, buffValueList, 5.0f, Resources.Load("BuffImage/14_Summon", typeof(Sprite)) as Sprite);
```

플레이어가 테스트 발판을 밟으면 발판 내의 필드인 buffTypenameList와 buffValueList에는 각각 "MoveSpeed", "MineDelay_Mining"와 0.5f, -0.7f이 정의되어 있다.

테스트 발판은 플레이어와 접촉되면 이 정의된 필드를 인자로 하는 [BuffManagerScript](https://unwoo52.github.io/posts/Team-Project-%EA%B8%B0%EC%88%A0%EB%AC%B8%EC%84%9C-Buff-%EA%B8%B0%EB%8A%A5/#buffmanagerscript)의 CreateBuff()를 실행한다.

<br>

***********


### [BuffManagerScript](https://unwoo52.github.io/posts/Team-Project-%EA%B8%B0%EC%88%A0%EB%AC%B8%EC%84%9C-Buff-%EA%B8%B0%EB%8A%A5/#buffmanagerscript) - 버프 Initialize

![imagename](/assets/image/Project/TeamProject/BuffStoryBoard/004.png)

> 오른쪽 위 하얀 부분이 버프가 표시되는 버프 패널이다. BuffManagerScript스크립트가 들어있으며, 이곳에 생성된 버프가 등록되고 관리된다.


- BuffManagerScript::CreateBuff
```cs
    public void CreateBuff(List<string> buffTypename, List<float> buffValue, float buffOriginTime, Sprite bufficon)
    {
        GameObject gameObject = Instantiate(buffPrefab, transform);
        gameObject.GetComponent<BaseBuff>().Init(buffTypename, buffValue, buffOriginTime);
        gameObject.GetComponent<Image>().sprite = bufficon;
    }
```

BuffManagerScript의 buffPrefab은 버프 오브젝트 프레펩([BaseBuff](https://unwoo52.github.io/posts/Team-Project-%EA%B8%B0%EC%88%A0%EB%AC%B8%EC%84%9C-Buff-%EA%B8%B0%EB%8A%A5/#basebuff))이 저장되어 있다. 버프 오브젝트를 생성한 후, 만들어진 버프 오브젝트의 Init()을 실행하고 아이콘을 지정한다.

<br>

*************

### [BaseBuff](https://unwoo52.github.io/posts/Team-Project-%EA%B8%B0%EC%88%A0%EB%AC%B8%EC%84%9C-Buff-%EA%B8%B0%EB%8A%A5/#basebuff)

![imagename](/assets/image/Project/TeamProject/BuffStoryBoard/005.png)

> 버프 패널에 생성된 버프 오브젝트이다. 여기에 BaseBuff스크립트가 들어있다.

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

### [PlayerIBuff](https://unwoo52.github.io/posts/Team-Project-%EA%B8%B0%EC%88%A0%EB%AC%B8%EC%84%9C-Buff-%EA%B8%B0%EB%8A%A5/#playerscript-ibuff) - 플레이어 스탯에게 버프 효과 적용

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

>BuffEffectAplly()를 설명하기 전에 앞서 플레이어의 스탯 구성에 대해 설명하겠다.
>
>예를 들어 플레이어의 이동속도 스탯은 다음처럼 구성되어있다.

<details>
<summary>PlayerStat Code</summary>
<div markdown="1">

```cs
        //이동속도
        [SerializeField] float moveSpeed_Origin;
        public float MoveSpeed_Origin { get => moveSpeed_Origin; set { moveSpeed_Origin = Mathf.Clamp(value, 0.1f, 100.0f); ; } }
        [SerializeField] float moveSpeed_Multiple;
        public float MoveSpeed__Multiple { get => moveSpeed_Multiple; set { moveSpeed_Multiple = value; } }
        [SerializeField] float moveSpeed_Plus;
        public float MoveSpeed_Plus { get => moveSpeed_Plus; set { moveSpeed_Plus = value; } }
        [SerializeField] float moveSpeed_AfterBuff;
        public float MoveSpeed_AfterBuff { get => moveSpeed_AfterBuff; set { moveSpeed_AfterBuff = Mathf.Clamp(value, 0.1f, 100.0f); } }
```


</div>
</details>

<br>

>각각의 스탯을 설명하면 다음과 같다.
>
>스탯중 **_Origin**은 플레이어 스탯의 기본 값이다. 이 값은 함부로 변경되어선 안된다.
>
>**_Multiple**은 버프 증강치 중 곱연산에 해당하는 값이다. **_Plus**는 버프 증강치 중 합연산에 해당하는 값이다.
>
>버프를 통해 _Multiple과 _Plus이 증가하면 <U>Origin에 _Multiple과 _Plus을 연산하여</U> 최종적으로 **_AfterBuff**을 만든다.
>
>***!플레이어의 실제 플레이에 적용되는 스탯 값은 이 _AfterBuff이다!***

<br>

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

버프의 종류와 해당 버프의 _Origin값을 인자로 받으면 해당 종류의 버프에 대해 탐색을 실행한다.

탐색의 대상은 플레이어의 버프 리스트, 리스트 내의 모든 버프의 값들을 전부 더한 뒤 return한다.


최종적으로 return된 origin값은 

```cs
myInfo.MoveSpeed_AfterBuff = BuffEffectAplly(s, myInfo.MoveSpeed_Origin);
```

원래 플레이어 스탯(myInfo)의 버프 적용 후 값에 저장되며, 플레이어의 실제 능력치의 연산에 사용되는 값은 이 _AfterBuff값들이다.


<br>

******
### 플레이어 버프 적용


## 지형 버프
