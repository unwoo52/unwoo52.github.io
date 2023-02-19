---
title: Team Project - 채광 및 벌목 기능 [기술문서]
author: unwoo52
date: 2022-11-15 00:00:00 +09:00
categories: [Unity, TeamProj]
tags: [Unity, TeamProj, Team, Mining, Logging, Resource, TechnicalDocument, Docs, Document]
---

# 기술 문서 개요

### 프로젝트, 기술문서, 스토리보드 링크

> [팀프로젝트 문서](https://unwoo52.github.io/posts/Team-Project-About/)는 프로젝트의 전체 내용과 개요 등에 대해 적혀있습니다.
>
> [youtube 채광 및 벌목 시연 영상](https://youtu.be/XYon_3MIK5E?t=92)에서 버프 작동을 유튜브로 볼 수 있습니다.
>
> <span style="color:ffffff">기술 문서는 전체 코드와 코드들에 대한 설명이 적혀있습니다.</span>
> 
> [스토리보드]()에는 해당 기능이 어떻게 게임에서 실행되는지 시간순서대로 읽기 쉽게 스토리로 작성하였습니다.

## 채광 및 벌목 구현 개요

게임 내의 자원인 나무와 광석을 파밍할 수 있는 시스템을 구현하였다.

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingSystem/000.gif)

나무를 벌목하면 통나무와 잎사귀 오브젝트 아이템들이 생성되어 떨어진다.

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingSystem/000-1.gif)

바위를 채굴하면 돌맹이 아이템이 생성되어 떨어지고, 일정 체력 이하로 떨어질 때 마다 바위의 크기가 작아진다.


<br>

----------


## 채광 및 벌목 관련 오브젝트들

### 채광 관련 오브젝트들

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingSystem/001.png)

> 바위 오브젝트. 채굴하면 바위 아이템을 떨어트리며, 채굴하면 할수록 작아진다.

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingSystem/001-1.png)

> 바위 오브젝트에서 드롭되는 아이템.

<br>

-------------

### 벌목 관련 오브젝트들

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingSystem/002.png)

> 나무 오브젝트. 체력을 모두 소모시키면 쓰러지면서 나무 통과 잎사귀 아이템들을 일정량 드롭한다.

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingSystem/002-1.png)

> 나무 오브젝트를 벌목하면 바닥에 남는 오브젝트.

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingSystem/002-2.png)

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingSystem/002-3.png)

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingSystem/002-4.png)



<br>

----------



## 나무 스크립트

<details>
<summary>TreeScript 전체 코드</summary>
<div markdown="1">

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class TreeScript : MonoBehaviour
{
    [SerializeField] GameObject TrunkBottom;
    [SerializeField] GameObject TrunkCylinder;
    [SerializeField] GameObject Bush1;
    [SerializeField] GameObject Leaf;

    [SerializeField] int numberLeaf;
    [SerializeField] int numberBush;

    [SerializeField] int numberTreeEffect;
    [SerializeField] float distanceGapofCylinder;

    [SerializeField] private float maxHp;
    [SerializeField] private float curHp;

    public Vector3 positionLeaf;
    public Vector3 positionTrunkBottom;
    public Vector3 positionTrunkCylinder;

    //TestField 
    public float RotationValue;

    private GameObject[] leafList;
    // Start is called before the first frame update
    void Start()
    {
        curHp = maxHp;
    }

    public void DamtoTree(float f)
    {
        curHp -= f;
        if (curHp == 0) TreeDestroy();
    }

    void TreeDestroy()
    {
        leafList = new GameObject[(numberBush + numberLeaf) * numberTreeEffect];
        if (TryGetComponent<CapsuleCollider>(out var cc)) Destroy(cc);
        if (TryGetComponent<MeshRenderer>(out var mr)) Destroy(mr);
        if (TryGetComponent<MeshFilter>(out var mf)) Destroy(mf);
        TreeDestroyEffect();
    }

    void TreeDestroyEffect()
    {
        Instantiate(TrunkBottom, transform.position + positionTrunkBottom, transform.rotation);
        for (int i = 0; i < numberTreeEffect; i++)
        {
            InstanteTreeEffect(transform.position + new Vector3(0, i * distanceGapofCylinder, 0) , i);
        }
        StartCoroutine(CleaningBushandLeaf());
    }

    void InstanteTreeEffect(Vector3 position, int i)
    {
        Instantiate(TrunkCylinder, position + positionTrunkCylinder, TrunkCylinder.transform.rotation);
        int tempNumberBush = numberBush;
        int tempNumberLeaf = numberLeaf;

        while (tempNumberBush > 0)
        {
            GameObject obj = Instantiate(Bush1,
                position + positionLeaf + new Vector3(Random.Range(-3, 3), Random.Range(0, 3), Random.Range(-3, 3)),
                new Quaternion(Random.Range(0, RotationValue), Random.Range(0, RotationValue), Random.Range(0, RotationValue), 0));
            leafList[(numberBush + numberLeaf) * i + numberLeaf + tempNumberBush - 1] = obj;
            tempNumberBush--;
        }
        while (tempNumberLeaf > 0)
        {
            GameObject obj = Instantiate(Leaf,
                position + positionLeaf + new Vector3(Random.Range(-3, 3), Random.Range(0, 3), Random.Range(-3, 3)),
                new Quaternion(Random.Range(0, RotationValue), Random.Range(0, RotationValue), Random.Range(0, RotationValue), 0));
            leafList[(numberBush + numberLeaf) * i + tempNumberLeaf - 1] = obj;
            tempNumberLeaf--;
        }
    }
    // 땅에 떨어진 잎과 풀을 일정 시간 이후에 모두 제거
    IEnumerator CleaningBushandLeaf()
    {
        yield return new WaitForSeconds(5);
        foreach(GameObject obj in leafList)
        {
            Destroy(obj.gameObject);
            //or obj.FadeLeaf();
        }
    }
}
```

</div>
</details>

<br>

----------




## 바위 스크립트

<details>
<summary>RockScript 전체 코드</summary>
<div markdown="1">

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class RockScript : MonoBehaviour
{
    [SerializeField] private Mesh[] MeshList;
    [SerializeField] private Vector3[] ScaleList;
    [SerializeField] private GameObject DropItem;
    [SerializeField] private int curStoneSizeLevel;

    [SerializeField] private int stoneMaxSize;
    [SerializeField] private int numofDestroyDropItem;

    [SerializeField] private float maxHp;
    [SerializeField] private float curHp;
    [SerializeField] private float maxCuttingHp;
    [SerializeField] private float cuttingHp;
    [SerializeField] private float maxCrackHp;
    [SerializeField] private float crackHp;
    //TestField
    public float testDmg;
    public float testPower;//튕겨져나가는 힘
    
    private void Start()
    {
        curHp = maxHp;
        curStoneSizeLevel = stoneMaxSize;
        cuttingHp = maxCuttingHp;
        crackHp = maxCrackHp;
    }
    void ChangeStoneSize(int i)
    {
        if (TryGetComponent<MeshFilter>(out var rockMeshFilter))
            rockMeshFilter.mesh = MeshList[i];
        if(TryGetComponent<MeshCollider>(out var rockMeshCollider))
            rockMeshCollider.sharedMesh = MeshList[i - 1];
        if(TryGetComponent<Transform>(out var rockTransform))
            rockTransform.localScale = ScaleList[i - 1];
    }
    public void DamToRock(Ray ray, Vector3 hit, float Dmg)
    {
        Dmg = testDmg; //TESTCODE===============================
        if (Dmg <= 0) Dmg = 0;
        cuttingHp -= Dmg;
        crackHp -= Dmg;
        if (crackHp <= 0)
        {
            int temp = CalculateTempHp(ref crackHp, ref maxCrackHp);
            while(temp > 0 && curHp > 0)
            {
                Dmg -= maxCrackHp;
                curHp -= maxCrackHp;
                instantiateDropItem(ray, hit);
                temp--;
            }
            curHp -= Dmg;
        }
        if(cuttingHp <= 0 && curHp > 0)
        {
            int temp = CalculateTempHp(ref cuttingHp, ref maxCuttingHp);
            curStoneSizeLevel -= temp;
            ChangeStoneSize(curStoneSizeLevel);
        }
        if (curHp <= 0) RockDestroyMethod();
    }
    private void instantiateDropItem(Ray ray, Vector3 hit)
    {
        GameObject obj = Instantiate(DropItem, hit, new Quaternion(Random.Range(0, 360), Random.Range(0, 360), Random.Range(0, 360), 0));
        if (obj.TryGetComponent<Rigidbody>(out var rigid))
        {
            rigid.AddForce(-ray.direction * testPower, ForceMode.VelocityChange);
        }
        obj.transform.localScale = new Vector3(0.2f, 0.2f, 0.2f);
    }
    /// <summary>
    /// curTempHp보다 높은 데미지를 받아 curTempHp가 음수가 되었을 때
    /// maxTempHp의 몇배에 해당하는 데미지인지를 계산해 리턴
    /// </summary>
    /// <param name="curTempHp"></param>
    /// <param name="maxTempHp"></param>
    /// <returns>recover가 반복된 횟수</returns>
    private int CalculateTempHp(ref float curTempHp, ref float maxTempHp)
    {
        if (curTempHp > 0) return 0;
        int temp = 0;
        while(curTempHp <= 0)
        {
            curTempHp += maxTempHp;
            temp++;
        }
        return temp;
    }

    private void RockDestroyMethod()
    {
        if (TryGetComponent<MeshCollider>(out var mc)) Destroy(mc);
        if (TryGetComponent<MeshRenderer>(out var mr)) Destroy(mr);
        if (TryGetComponent<MeshFilter>(out var mf)) Destroy(mf);
        for (int i = 0; i < numofDestroyDropItem; i++)
        {
            Ray ray = new Ray(Vector3.zero, new Vector3(Random.Range(0, 1), Random.Range(0, 1), Random.Range(0, 1)));
            Vector3 hit = transform.position;
            instantiateDropItem(ray, hit);
        }
        //드롭템이 사라졌는지 1초마다 체크,
        //모두 사라졌다면 자기 자신 파괴
    }
}
```

</div>
</details>

<br>

----------


## 플레이어 애니메이션 이벤트 스크립트

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingSystem/003.png)

> Unity Event를 이용해 플레이어의 애니메이션에 함수가 발동하도록 하였다.

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingSystem/003.png)

> 플레이어 채광 애니메이션 중 곡갱이를 찍는 부분에 애니메이션 이벤트가 삽입되어 있는 사진


<details>
<summary>상세내용확인</summary>
<div markdown="1">

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Events;

public class AnimEvent : MonoBehaviour
{
    public UnityEvent Mining = null;
    public UnityEvent TreeGet = null;

	...

    public void MiningAnimEvent()
    {
        Mining?.Invoke();
    }
    public void MiningTreeEvent()
    {
        TreeGet?.Invoke();
    }
	...
}

```

</div>
</details>


<details>
<summary>플레이어 스크립트 내의 애니메이션 스크립트</summary>
<div markdown="1">

```cs
        public void AnimDamageTpRock()
        {
            Mining mining = _mouseInput.Mining;
            if (mining.RockScript != null)
            {
                mining.RockScript.DamToRock(mining.RockHitRay, mining.RockHitVector, PlayerScript.PlayerInstance.myInfo.PowerMining_Origin);
            }
        }
        public void AnimDamageToTree()
        {
            Mining mining = _mouseInput.Mining;
            if (mining.TreeScript != null)
            {
                mining.TreeScript.DamtoTree(50f);
            }
        }
```

</div>
</details>
