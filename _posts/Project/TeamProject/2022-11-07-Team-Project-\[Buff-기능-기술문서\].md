---
title: Team Project \[Buff 기능 기술문서\]
author: unwoo52
date: 2022-11-07 00:00:00 +09:00
categories: [Unity, TeamProj]
tags: [Unity, TeamProj, Team, Buff, TechnicalDocument, Docs, Document]
---

# 개요

팀 프로젝트 전체 개요 문서 >> [Team Project About](https://unwoo52.github.io/posts/Team-Project-About/)

Buff  시연 >> [https://youtu.be/XYon_3MIK5E?t=39](https://youtu.be/XYon_3MIK5E?t=39)

Buff 기능 스토리 보드 >> [https://unwoo52.github.io/posts/Team-Project-%EC%8A%A4%ED%86%A0%EB%A6%AC%EB%B3%B4%EB%93%9C-Buff-%EA%B8%B0%EB%8A%A5/](https://unwoo52.github.io/posts/Team-Project-%EC%8A%A4%ED%86%A0%EB%A6%AC%EB%B3%B4%EB%93%9C-Buff-%EA%B8%B0%EB%8A%A5/)

팀프로젝트 내에서 내가 구현한 Buff에 대한 기술문서이다.

## Buff 구현 개요

팩토리 메소드 패턴을 이용하여 Buff를 구현하였다.




# 버프 관련 오브젝트들

----------


## 버프 판넬

![imagename](/assets/image/Project/TeamProject/BuffSystem/001.png)

플레이어의 버프창과 같은 오브젝트로, BuffManagerScript 스크립트로 제어되고 버프 오브젝트들이 저장되는곳. 버프들 아이콘이 표시되며, 지속시간이 있다면 지속시간도 표시된다.

----------



## 버프 오브젝트

![imagename](/assets/image/Project/TeamProject/BuffSystem/002.png)

Base Buff로 제어되는 버프 객체의 오브젝트. 버프에 대한 정보들을 담고 있으며, 팩토리인 BuffManagerScript를 통해 원하는 타입과 종류의 버프 오브젝트가 생성된다.

----------


# 코드 설명


----------



## BaseBuff 

### 전체 코드

<details>
<summary>BaseBuff</summary>
<div markdown="1">

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using Player;

/*
 * BaseBuff에 대한 설명
 * 버프에 대한 정보를 init으로 type, value, time, image을 받아 실행
 * playerScript의 buff list에 버프를 추가(BuffListAdd) 및 버프 효과 갱신(ChooseBuff)를 실행
 * 버프가 지속시간이 다 되면 자신을 파괴함과 동시에 playerScirpt의 buff list에서 다시 자신의 인스턴스를 삭제(removeBuff(this.BaseBuff)) 및 버프 효과 갱신(ChooseBuff)을 실행
 */
public class BaseBuff : MonoBehaviour //BuffPrefeb의 스크립트
{
    #region Buff Data
    public IBuff PlayerIBuff;

    //1. 버프의 효과 종류 이름(공격력 디버프, 방어력 디버프)을 받아올 변수 ex)디버프오라 "방어력"
    List<string> buffTypenameList = new();
    public List<string> BuffTypenameList { get => buffTypenameList; }

    //2-1. 얼마큼의 값을 변화시켜줄지 ex)디버프오라 방어력 "-10%"
    List<float> buffValueList = new();
    public List<float> BuffValueList { get => buffValueList; }

    //3. 버프 지속시간 ex)유니온오라 "40초"
    float buffOriginTime;
    public float BuffOriginTime { get => buffOriginTime; }

    //4. 버프 아이콘
    public Image icon; //여기에 버프 이미지 보관 ex)"디버프오라 아이콘"

    //5. 버프 현재 남은 시간. root method에서 지속적으로 deltatime을 뺌
    float currentTime;

    //6. TerrainLayerIntNumber을 갖고있거나 ex)31 = ice, 그 외의 버프 코드를 저장
    int buffCode;
    public int BuffCode { get => buffCode; }
    
    readonly WaitForSeconds BuffCheckRootSecond = new((float)(Time.deltaTime * 0.1));

    #endregion
    private void Awake()
    {
        icon = GetComponent<Image>();
    }
    /// <summary>
    /// 일반 버프
    /// </summary>
    /// <param name="buffTypenameList"></param>
    /// <param name="buffValueList"></param>
    public void Init(List<string> buffTypenameList, List<float> buffValueList) //받아온 정보를 이 prefep에 init
    {
        this.buffTypenameList = buffTypenameList;
        this.buffValueList = buffValueList;
        currentTime = this.buffOriginTime;
        icon.fillAmount = 1f;

        PlayerIBuff = PlayerScript.instance.GetComponent<IBuff>();
        Destroy(gameObject.GetComponent<Button>());
        NonCoroutineBuffActivation();
    }
    /// <summary>
    /// 일반 버프
    /// </summary>
    /// <param name="buffTypenameList"></param>
    /// <param name="buffValueList"></param>
    /// <param name="buffOriginTime"></param>
    public void Init(List<string> buffTypenameList, List<float> buffValueList, float buffOriginTime) //받아온 정보를 이 prefep에 init
    {
        this.buffTypenameList = buffTypenameList;
        this.buffValueList = buffValueList;
        this.buffOriginTime = buffOriginTime;
        currentTime = this.buffOriginTime;
        icon.fillAmount = 1f;

        PlayerIBuff = PlayerScript.instance.GetComponent<IBuff>();
        NomalBuffactivation();
    }
    /// <summary>
    /// Terrain 버프
    /// </summary>
    /// <param name="buffTypenameList"></param>
    /// <param name="buffValueList"></param>
    /// <param name="buffcode">지형 레이어 번호</param>
    public void Init(List<string> buffTypenameList, List<float> buffValueList, int buffcode)
    {
        this.buffTypenameList = buffTypenameList;
        this.buffValueList = buffValueList;
        buffCode = buffcode;
        icon.fillAmount = 1f;

        PlayerIBuff = PlayerScript.instance.GetComponent<IBuff>();
        NonCoroutineBuffActivation();
    }


    #region 버프가 생성될 때 버프효과 적용
    void NonCoroutineBuffActivation()
    {
        PlayerIBuff.BuffListAdd(this);
        PlayerIBuff.ChooseBuff(buffTypenameList);
    }
    private void NomalBuffactivation()
    {
        PlayerIBuff.BuffListAdd(this);
        PlayerIBuff.ChooseBuff(buffTypenameList);
        StartCoroutine(Activation());
    }
    #endregion

    #region 버프효과 Root
    //타이머
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

    #endregion

    #region 버프효과 파괴
    public void BuffDeActivation() // 버프 파괴 메서드
    {
        PlayerIBuff.RemovBuff(this);
        PlayerIBuff.ChooseBuff(buffTypenameList);
        Destroy(this.gameObject);
    }
    #endregion

    #region Click Method
    public void ClickBuffDeActivation()
    {
        BuffDeActivation();
    }
    #endregion
}
```


</div>
</details>

버프 객체이다. BaseBuff 버프 오브젝트가 존재함으로써 버프가 유지되고, BaseBuff.BuffDeActivation() 등으로 파괴되면 버프 효과가 제거된다.

버프 효과를 위한 효과 종류와 가중치, 지속시간과 버프 아이콘 등을 갖고 있다.

### 코드 설명

- 필드 전체

```cs
	public IBuff PlayerIBuff;
    List<string> buffTypenameList = new();
    public List<string> BuffTypenameList { get => buffTypenameList; }

    List<float> buffValueList = new();
    public List<float> BuffValueList { get => buffValueList; }

    float buffOriginTime;
    public float BuffOriginTime { get => buffOriginTime; }

    public Image icon; //여기에 버프 이미지 보관 ex)"디버프오라 아이콘"

    float currentTime;

    int buffCode;
    public int BuffCode { get => buffCode; }
    
    readonly WaitForSeconds BuffCheckRootSecond = new((float)(Time.deltaTime * 0.1));

    public float WeightTime;
```

버프에게 필요한 필드들이다. 버프 아이콘, 버프 효과 종류와 그 값, 지속시간 등이 포함되어 있다.

<br>

```cs
readonly WaitForSeconds BuffCheckRootSecond = new((float)(Time.deltaTime * 0.1));
```

나중에 버프타임 코루틴이 돌아갈 때, 매번 new로 WaitForSeconds를 생성하지 않고 한번 생성된 BuffCheckRootSecond를 yield return하기 위함.

<br>

```cs
    private void Awake()
    {
        icon = GetComponent<Image>();
    }
```

버프 생성시 인자로 받은 이미지를 아이콘으로 지정

<br>

```cs
    public void Init(List<string> buffTypenameList, List<float> buffValueList, float buffOriginTime) { ... }
    public void Init(List<string> buffTypenameList, List<float> buffValueList, int buffcode) { ... }
    public void Init(List<string> buffTypenameList, List<float> buffValueList) { ... }
    public void Init(List<string> buffTypenameList, List<float> buffValueList) //받아온 정보를 이 prefep에 init
    {
        this.buffTypenameList = buffTypenameList;
        this.buffValueList = buffValueList;
        currentTime = this.buffOriginTime;
        icon.fillAmount = 1f;

        PlayerIBuff = PlayerScript.instance.GetComponent<IBuff>();
        Destroy(gameObject.GetComponent<Button>());
        NonCoroutineBuffActivation();
    }
```

버프가 생성된 이후 실행하는 Initalize 함수. 버프의 종류별로 override되어 종류별로 함수가 존재한다.

인자로 받아온 버프의 종류들과 가중치, 버프 지속시간 등을 버프 오브젝트의 필드들에 대입한다.

<br>

```cs
    private void NomalBuffactivation()
    {
        PlayerIBuff.BuffListAdd(this);
        PlayerIBuff.ChooseBuff(buffTypenameList);
        StartCoroutine(Activation());
    }
```

지속시간이 있는 일반 버프에게 실행되는 Buff Activation 함수. Activation()을 실행시키면 버프지속시간에게서 WeightTime만큼을 감소시키는 코루틴이 실행된다.

<br>

```cs
    void NonCoroutineBuffActivation()
    {
        PlayerIBuff.BuffListAdd(this);
        PlayerIBuff.ChooseBuff(buffTypenameList);
    }
```

지속시간이 없는(WeightTime을 감소시키는 코루틴을 실행할 필요가 없는) 버프의 경우 실행되는 Buff Activation 함수.

<br>

```cs
    public void BuffDeActivation()
    {
        PlayerIBuff.RemovBuff(this);
        PlayerIBuff.ChooseBuff(buffTypenameList);
        Destroy(this.gameObject);
    }
```

버프 파괴 함수. 플레이어의 버프 리스트에서 이 버프를 제거하고, 플레이어의 스탯을 갱신한 뒤 버프 오브젝트를 파고한다.

> 플레이어의 RemovBuff에서 ChooseBuff를 실행시키게 해 코드를 한 줄 짧게 만들 수 있지만, 모든 버프가 ChooseBuff()실행이 필요하지 않을 수도 있으므로 BaseBuff 스크립트에서 실행하도록 하였다.

<br>

```cs
    public void ClickBuffDeActivation()
    {
        BuffDeActivation();
    }
```

인스펙터의 버튼과 연결되어 있는 함수.

<br>

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

while동안 버프 지속시간을 감소시키고, 버프 아이콘의 모양을 변경한다.

while문이 끝나면 버프를 파괴하는 코루틴.

<br>


----------



## BuffManagerScript 

### 전체 코드

<details>
<summary>BuffManagerScript</summary>
<div markdown="1">

```cs
public class BuffManagerScript : MonoBehaviour //UI중 Buff Panel에 인스턴스
{
    #region instance
    public static BuffManagerScript instance;
    [SerializeField] private GameObject buffPrefab;
    private void Awake()
    {
        instance = this;
        buffPrefab = Resources.Load("Prefabs/buffObject") as GameObject;
    }
    #endregion

    /// <summary>
    /// 
    /// </summary>
    /// <param name="buffTypename">버프 효과 이름</param>
    /// <param name="buffValue">버프 효과 수치</param>
    /// <param name="buffOriginTime">지속시간</param>
    /// <param name="bufficon">Sprite 버프 아이콘</param>
    /// 
    public void CreateBuff(List<string> buffTypename, List<float> buffValue, Sprite bufficon)
    {
        GameObject gameObject = Instantiate(buffPrefab, transform); //인스턴스 생성(buffBase), 위치는 this(MyBuffPanel)에.
        //gameobject = instanced baseBuff prefep
        gameObject.GetComponent<BaseBuff>().Init(buffTypename, buffValue); //인스턴스에 버프 정보 입력
        gameObject.GetComponent<Image>().sprite = bufficon; //
    }
    public void CreateBuff(List<string> buffTypename, List<float> buffValue, float buffOriginTime, Sprite bufficon)
    {
        GameObject gameObject = Instantiate(buffPrefab, transform); //인스턴스 생성(buffBase), 위치는 this(MyBuffPanel)에.
        //gameobject = instanced baseBuff prefep
        gameObject.GetComponent<BaseBuff>().Init(buffTypename, buffValue, buffOriginTime); //인스턴스에 버프 정보 입력
        gameObject.GetComponent<Image>().sprite = bufficon; //
    }
    public void CreateBuff(List<string> buffTypename, List<float> buffValue, Sprite bufficon, int buffcode)
    {
        GameObject gameObject = Instantiate(buffPrefab, transform); //인스턴스 생성(buffBase), 위치는 this(MyBuffPanel)에.
        //gameobject = instanced baseBuff prefep
        gameObject.GetComponent<BaseBuff>().Init(buffTypename, buffValue, buffcode); //인스턴스에 버프 정보 입력
        gameObject.GetComponent<Image>().sprite = bufficon; //
        if (29 <= buffcode || buffcode <= 31) Destroy(gameObject.GetComponent<Button>());
    }
}
```

</div>
</details>

### 코드 설명

```cs
	public static BuffManagerScript instance;
    [SerializeField] private GameObject buffPrefab;
    private void Awake()
    {
        instance = this;
        buffPrefab = Resources.Load("Prefabs/buffObject") as GameObject;
    }
```

싱글턴 객체 구현과 [버프 오브젝트 프레펩]()을 갖고있다.

<br>

```cs
public void CreateBuff(List<string> buffTypename, List<float> buffValue, Sprite bufficon)
    {
        GameObject gameObject = Instantiate(buffPrefab, transform);
        //gameobject = instanced baseBuff prefep
        gameObject.GetComponent<BaseBuff>().Init(buffTypename, buffValue);
        gameObject.GetComponent<Image>().sprite = bufficon; //
    }
```

여러 종류의 버프를 구현할 수 있게 다양한 타입을 가지는 버프 생성 메소드.

버프 생성을 원하는 곳에서 호출된다.

```cs
//ex)아이템 사용 스크립트에 BuffManagerScript.instance.CreateBuff(아이템 버프 효과 종류, 버프 값, 버프 아이콘)
```

----------




## interface IBuff 

### 전체 코드

<details>
<summary>interface IBuff</summary>
<div markdown="1">

```cs
public interface IBuff
{
    void BuffListAdd(BaseBuff baseBuff);
    void RemovBuff(BaseBuff baseBuff);
    void ChooseBuff(List<string> buffTypenameList);
    float BuffEffectAplly(string s, float origin);
    void BuffValueApply(string s, float value);
}
```


</div>
</details>


----------



## PlayerScript IBuff 

### 전체 코드

<details>
<summary>PlayerScript IBuff</summary>
<div markdown="1">

```cs
		public List<BaseBuff> BuffList = new();
        /// <summary>
        /// 버프스탯에 적용받는 스탯들(이동속도 등) 외의 체력과 같은 스탯에 효과를 적용
        /// </summary>
        /// <param name="s">ex) CurHP</param>
        /// <param name="f">ex) 5.0f</param>
        public void BuffValueApply(string s, float f)
        {
            switch (s)
            {
                case "CurHP":
                    myInfo.CurHP += f;
                    break;
            }
        }
        public void BuffListAdd(BaseBuff baseBuff)
        {
            BuffList.Add(baseBuff);
        }
        public void RemovBuff(BaseBuff baseBuff)
        {
            BuffList.Remove(baseBuff);
        }
        /// <summary>
        /// buffTypenameList에 있는 buffTypename들에 대해 각각의 IBuff.BuffEffectAplly()을 수행
        /// </summary>
        /// <param name="buffTypenameList"></param>
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
        /// <summary>
        /// 버프 리스트를 모두 탐색하여 buffTypeName이 일치하는 버프의 값들을 구해 origin에 더하여 반환
        /// </summary>
        /// <param name="buffTypeName">변경해야 하는 버프 타입 이름 ex)MoveSpeed_AfterBuff</param>
        /// <param name="origin">해당 타입의 오리진 ex)MoveSpeed_Origin</param>
        /// <returns>해당 버프 타입의 버프 효과가 모두 더해진 값</returns>
        public float BuffEffectAplly(string buffTypeName, float origin)//플레이어가 가진 BuffList에서 s와 일치하는 모두를 탐색한 후 해당하는 value들을 origin에 더해서 return
        {
            if (BuffList.Count > 0)
            {
                float temp = 0;
                for (int i = 0; i < BuffList.Count; i++)//버프 갯수만큼 반복, 버프 리스트를 훑기
                {
                    for (int j = 0; j < BuffList[i].BuffTypenameList.Count; j++)//리스트의 i번째 버프의 buffTypenameList를 탐색
                    {
                        if (BuffList[i].BuffTypenameList[j].Equals(buffTypeName)) //리스트의 i번째 버프의 buffTypenameList의 j번째 문자열을 s와 비교
                            temp += origin * BuffList[i].BuffValueList[j]; //일치한다면 temp에 버프의 value를 더하기
                    }
                }
                return origin + temp; //origin에 temp들을 모두 더하여 리턴
            }
            else return origin;
        }
```


</div>
</details>

### 코드 설명

```cs
		public List<BaseBuff> BuffList = new();
```

플레이어가 갖고 있는 버프들을 담고 있는 리스트

<br>

```cs
        public void BuffListAdd(BaseBuff baseBuff)
        {
            BuffList.Add(baseBuff);
        }
        public void RemovBuff(BaseBuff baseBuff)
        {
            BuffList.Remove(baseBuff);
        }
```

BuffList에서 버프를 추가하거나 더하는 함수

<br>


```cs
	public void ChooseBuff(List<string> buffTypenameList)
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

버프 리스트를 인자로 받아 리스트 내의 각각의 버프 타입들에 대해 BuffEffectAplly를 실행해 플레이어 스탯을 갱신하는 함수

<br>

```cs
		public void BuffValueApply(string s, float f)
        {
            switch (s)
            {
                case "CurHP":
                    myInfo.CurHP += f;
                    break;
                    
                ....
            }
        }
```

ChooseBuff를 통해 실행되며, 해당하는 버프 종류에 따라 플레이어의 스탯을 변경하는 함수

