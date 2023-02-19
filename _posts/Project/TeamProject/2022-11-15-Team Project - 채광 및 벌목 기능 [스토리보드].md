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


## 나무 보상 아이템 드롭

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


