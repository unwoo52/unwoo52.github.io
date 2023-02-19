---
title: Team Project - 채광 및 벌목 기능 [스토리보드]
author: unwoo52
date: 2022-11-15 00:00:00 +09:00
categories: [Unity, TeamProj]
tags: [Unity, TeamProj, Team, Mining, Logging, Resource, StoryBoard]
---

# 스토리보드 개요

### 프로젝트, 기술문서, 스토리보드 링크

> [팀프로젝트 문서](https://unwoo52.github.io/posts/Team-Project-About/)는 프로젝트의 전체 내용과 개요 등에 대해 적혀있습니다.
>
> [youtube 채광 및 벌목 시연 영상](https://youtu.be/XYon_3MIK5E?t=92)에서 버프 작동을 유튜브로 볼 수 있습니다.
>
> [기술 문서]()는 전체 코드와 코드들에 대한 설명이 적혀있습니다.
> 
> <span style="color:ffffff">스토리보드에는 해당 기능이 어떻게 게임에서 실행되는지 시간순서대로 읽기 쉽게 스토리로 작성하였습니다.</span>


<br>

---------


## 나무 벌목

### 플레이어 벌목 애니메이션 이벤트

- PlayerScript::AnimDamageToTree()

```cs
    public void AnimDamageToTree()
    {
    	Mining mining = _mouseInput.Mining;
    	if (mining.TreeScript != null)
    	{
    		mining.TreeScript.DamtoTree(50f);
    	}
    }
```

```cs
    public void MiningTreeEvent()
    {
        TreeGet?.Invoke();
    }
```

<br>

-----------------



### 나무 체력 감소

- AnimEvent::DamtoTree()

```cs
    public void DamtoTree(float f)
    {
        curHp -= f;
        if (curHp == 0) TreeDestroy();
    }
```


### 나무 파괴 스크립트 실행

- TreeScript::TreeDestroy()

```cs
    void TreeDestroy()
    {
        leafList = new GameObject[(numberBush + numberLeaf) * numberTreeEffect];
        if (TryGetComponent<CapsuleCollider>(out var cc)) Destroy(cc);
        if (TryGetComponent<MeshRenderer>(out var mr)) Destroy(mr);
        if (TryGetComponent<MeshFilter>(out var mf)) Destroy(mf);
        TreeDestroyEffect();
    }
````

나무를 파괴하는 스크립트이다. 후술할 스크립트가 실행되어야 하기 때문에 나무의 스크립트는 파괴되면 안되므로 충돌체와 렌더링을 지워 스크립트만 남은 오브젝트로 만들었다.

동시에 잎과 잔가지를 담아둘 GameObject배열인 LeafList의 크기를 지정한다. 여기에 저장된 잎과 잔가지는 코루틴을 통해 일정 시간이 지나면 파괴되도록 한다.

이후 아이템(통나무와 잎, 잔가지)가 드롭되는 TreeDestroyEffect();을 실행한다.


<br>


------------




### 나무 벌목 후 보상 오브젝트 생성

통나무는 필드로 잎의 갯수, 잔가지의 갯수, 그리고 실린더 갯수를 갖고 있다.

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingStoryBoard/002.png)

> 나무 오브젝트의 인스펙터. 나무에서 드롭되는 오브젝트의 양을 쉽게 조절할 수 있는 필드가 있다.

통나무는 파괴되면서 지정된 통나무 갯수(위 이미지의 빨간 줄 그어진 필드)만큼의 'Tree Effect'를 실행한다.

한번의 Tree Effect에는 통나무(Cylinder)와 지정된 갯수의 잎과 잔가지 수(위 이미지의 number bush, leaf가 그것이다)가 생성되어 떨어진다.


나무의 키가 크다면 통나무 갯수를 늘리고, 통나무간 간격 필드(distanceGapofCylinder) 값을 높히면, 생성되는 통나무와 잎사귀간 거리가 멀어진다.

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingStoryBoard/003.png)



<br>

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingStoryBoard/001.png)

> 4개의 통나무를 벌목한 사진

- TreeScript::TreeDestroyEffect()

```cs
    void TreeDestroyEffect()
    {
        Instantiate(TrunkBottom, transform.position + positionTrunkBottom, transform.rotation);
        for (int i = 0; i < numberTreeEffect; i++)
        {
            InstanteTreeEffect(transform.position + new Vector3(0, i * distanceGapofCylinder, 0) , i);
        }
        StartCoroutine(CleaningBushandLeaf());
    }
```

나무가 벌목되면 생성되는 오브젝트는 밑에 이미지의 4종류이다.

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingSystem/002-1.png)

> 나무가 있던 자리에 고정되어 있는 스텀프가 1개

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingSystem/002-2.png)

> 나무의 몸통을 차지하던 통나무가 1개 이상

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingSystem/002-3.png)

> 나무의 잎사귀가 1개 이상

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingSystem/002-4.png)

> 나무의 잔가지가 1개 이상

이중, 스텀프는 한 코드로 실행되고, 나머지 3종류 오브젝트는 밑의 InstanteTreeEffect()에서 생성되어 추척되고 밑의 코루틴 코드를 통해 일정시간이 지나면 사라진다.

<details>
<summary>시간제 오브젝트를 삭제하는 코루틴 코드</summary>
<div markdown="1">

- TreeScript::CleaningBushandLeaf()

```cs
    IEnumerator CleaningBushandLeaf()
    {
        yield return new WaitForSeconds(5);
        foreach(GameObject obj in leafList)
        {
            Destroy(obj.gameObject);
            //or obj.FadeLeaf();
        }
    }
```

</div>
</details>


<br>


------------



### 나무 보상 아이템 드롭

- TreeScript::InstanteTreeEffect(Vector3 position, int i)


```cs
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
````

통나무와 잎사귀, 잔가지를 지정된 갯수만큼 생성한다. 생성된 오브젝트들은 leafList라는 GameObject[] 배열 필드에 저장되었다가 일정 시간이 지나면 삭제된다.

굳이 리스트에 담아서 오브젝트들을 추적하고 삭제하는 이유는, 더 적은 비용으로 오브젝트들의 사라질 시간을 계산하기 위해서이다.

각 오브젝트들이 별도로 시간을 재고 삭제 되는 것은 오브젝트의 수(위 예시에선 18개(통나무와 잎과 잔가지) * 4(통나무 수) = **72개**)의 코루틴이 독립적으로 실행되어서 wait for second를 실행하게 된다.

때문에 하나의 오브젝트에서 일관된 시간을 측정하고 삭제하는 것이 최적화 면에서 유리하다.


<br>


------------




## 바위 채광

### 플레이어 채광 애니메이션 이벤트

- AnimEvent::MiningAnimEvent()

```cs
    public void MiningAnimEvent()
    {
        Mining?.Invoke();
    }
```

<br>

------------

### 바위 체력 종류

바위 체력에는 3종류가 있다. 전체 체력, 컷팅 체력, 크랙 체력.

전체 체력은 단순한 최대 체력이다. 모두 소진되면 바위 오브젝트가 많은 돌조각을 드롭하며 파괴된다.

컷팅 체력은 바위의 크기가 감소하는 체력이다. 이 체력이 모두 감소되면, 바위의 크기가 줄어든다.

크랙 체력은 돌맹이 아이템을 얻을 수 있는 체력이다. 곡갱이의 힘이 크랙보다 약하면 여러번 찍어야 아이템을 드롭하고, 반대로 곡갱이의 힘이 더 크다면 한번의 곡갱이질로 여러개의 아이템을 얻을 수 있다.

<br>

------------


### 바위 체력 감소

- RockScript::DamToRock(Ray ray, Vector3 hit, float Dmg)

```cs
    public void DamToRock(Ray ray, Vector3 hit, float Dmg)
    {
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
```

- RockScript::CalculateTempHp()

```cs
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
```



<br>

------------


### 바위 아이템 드롭

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingSystem/006.gif)

- RockScript::instantiateDropItem(Ray ray, Vector3 hit)

```cs
    private void instantiateDropItem(Ray ray, Vector3 hit)
    {
        GameObject obj = Instantiate(DropItem, hit, new Quaternion(Random.Range(0, 360), Random.Range(0, 360), Random.Range(0, 360), 0));
        if (obj.TryGetComponent<Rigidbody>(out var rigid))
        {
            rigid.AddForce(-ray.direction * testPower, ForceMode.VelocityChange);
        }
        obj.transform.localScale = new Vector3(0.2f, 0.2f, 0.2f);
    }
```

<br>

------------

### 바위 크기 감소

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingSystem/004.gif)

> 바위 채굴 중 컷팅 체력에 도달하면 바위의 크기가 줄어든다.

- RockScript::ChangeStoneSize(int i)

```cs
    [SerializeField] private Mesh[] MeshList;
    [SerializeField] private Vector3[] ScaleList;
    
    ...
   
    void ChangeStoneSize(int i)
    {
        if (TryGetComponent<MeshFilter>(out var rockMeshFilter))
            rockMeshFilter.mesh = MeshList[i];
        if(TryGetComponent<MeshCollider>(out var rockMeshCollider))
            rockMeshCollider.sharedMesh = MeshList[i - 1];
        if(TryGetComponent<Transform>(out var rockTransform))
            rockTransform.localScale = ScaleList[i - 1];
    }
```

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingSystem/005.gif)

> Mesh[] MeshList와 Vector3[] ScaleList에 다음과 같이 Mesh와 Scale값이 저장되어 있다.

바위의 사이즈가 바뀌면 저장된 필드의 값을 토대로 바위의 크기와 모양이 바뀐다.


<br>

------------

### 바위 아이템 

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingSystem/000.gif)

> 바위의 체력이 모두 소진되면 numofDestroyDropItem필드에 저장된 값 만큼의 돌맹이 아이템이 드롭된다.

- RockScript::RockDestroyMethod()


```cs
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
        
        ...
        
        //2분 뒤 돌맹이들이 삭제되는 코루틴
    }
```
