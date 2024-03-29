---
title: Team Project - 건물 아이템 설치 기능 [기술 문서]
author: unwoo52
date: 2022-11-14 00:00:00 +09:00
categories: [Unity, TeamProj]
tags: [Unity, TeamProj, Team, Build, StoryBoard]
---

# 기술 문서 개요

### 프로젝트, 오브젝트와 전체코드 설명, 기술 문서 링크

> [팀프로젝트 문서](https://unwoo52.github.io/posts/Team-Project-About/)는 프로젝트의 전체 내용과 개요 등에 대해 적혀있습니다.
>
> [youtube 건물 아이템 시연 영상](https://youtu.be/XYon_3MIK5E?t=72)에서 버프 작동을 유튜브로 볼 수 있습니다.
> 
> [오브젝트와 전체코드 설명]()는 전체 코드와 코드들에 대한 설명이 적혀있습니다.
> 
> <span style="color:#ffffff">기술 문서에는 해당 기능이 어떻게 게임에서 실행되는지 시간순서대로 읽기 쉽게 작성하였습니다.</span>


## 건물 아이템 설치 기능 개요

마우스로 아이템 클릭

start build 실행

건물 오브젝트 생성

건물 오브젝트 위치 타겟팅






### 마우스로 아이템 클릭 후 건물 아이템 사용 시작

![imagename](/assets/image/Project/TeamProject/BuildingObjectStoryBoard/007.gif)

> 플레이어가 건축 상태로 돌입한다. 건축 상태에 돌입하면서 다음과 같은 코드와 행동을 실행한다.

- MouseInPut_Build::StartBuilding(GameObject gameObject)

```cs
public void StartBuilding(GameObject gameObject)
{
    HandBuilding = gameObject;
    HandBuilding.transform.GetChild(0).TryGetComponent(out HandBuildingScript);
    if (HandBuilding is null)
    {
        throw new ArgumentNullException(nameof(gameObject));
    }
    if (HandBuildingScript is null)
    {
        throw new ArgumentNullException(nameof(gameObject));
    }

    PlayerScript.PlayerInstance.ActiveOnCursor();
    isBuildingStateActive = true;

    HandBuildingScript.ChangeState(BuildingObjectScript.BuildingObjectState.Making);
    HandBuilding.transform.SetParent(PlayerScript.PlayerInstance.transform);
    gameObject.transform.GetChild(0).TryGetComponent(out HandBuildingScript);
    SwapRenderer();
}
```

- MouseInPut_Build::SwapRenderer()

```cs
private void SwapRenderer()
{
    Material material = Resources.Load("Prefabs/imsi") as Material;
    HandBuildingScript.ChangeMaterials(material);
    if (HandBuildingScript.GetMaterial() == null) 
    { 
        Debug.LogError("임시 Metarial로 바꾸는 데에 실패했습니다.");
        HandBuildingScript.SetOrigibMaterail();
    }
}
```

건물 오브젝트가 건축중에는 반투명하게 이펙트 형태로 보여지게 하기 위해 메테리얼과 셰이더를 임시 메테리얼로 교체한다. 


param으로 전달받은 GameObject와 GameObject의 스크립트를 MouseInPut_Build인스턴스 필드에 저장한다.

bool 필드인 isBuildingStateActive를 활성화 한다.

아이템 커서가 활성화 된 상태에서 인벤토리 클릭을 통해 건축을 시작했으니, 다시 커서를 비활성화 시킨다.

생성한 오브젝트의 하이라키 위치를 플레이어 밑으로 설정한다.

<br>

------------

### 건물 오브젝트 위치 바꾸기


> 생성된 오브젝트가 플레이어의 마우스 끝에 위치하게 한다.

- MouseInPut_Build:: RotateBuilding_to_HitPosition(Vector3 vector3)


```cs
private void RotateBuilding_to_HitPosition(Vector3 vector3)
{
    HandBuilding.transform.position = vector3;
}
```



<br>

-------------


### 건축 중 건물 오브젝트 내의 엑티브 오브젝트 비활성화

건물 오브젝트는 건축 전에는 외형과 건축에 필요한 오브젝트만 활성화 시켰다가, 건축이 되면 효과 오브젝트를 활성화 시킨다.

건축 위치 선정을 위해 띄운 오브젝트도 버프 효과를 제공하면 곤란할 것이다.

<br>


![imagename](/assets/image/Project/TeamProject/BuildingObjectStoryBoard/002.gif)

> 사진의 모닥불은 모닥불 버프 기능과 광원 기능이 있는 오브젝트가 비활성화 상태이다가, 건축이 되자 다시 활성화 되는 것을 볼 수 있다.



- BuildingObjectScript::ChangeState

상태머신을 통해 건물이 running 상태가 되면 효과 오브젝트를 활성화 시킨다.

```cs
public void ChangeState(BuildingObjectState state)
{
    if (campFireState == state) return;
    switch (state)
    {
        case BuildingObjectState.Create:
            break;
        case BuildingObjectState.Item:
            break;
        case BuildingObjectState.Making:
            OverlapObject.SetActive(true);
            break;
        case BuildingObjectState.Runing:
            EffectObject.SetActive(true);
            break;
        case BuildingObjectState.Destroy:
            Destroy(transform.parent.gameObject);
            Destroy(this);
            break;
    }
}   
```
<br>

<details>
<summary>*오브젝트 하이라키 셋팅은 오브젝트가 생성될 때, 이 코드를 통해 설정된다.* </summary>
<div markdown="1">

```cs
private void HierarchySetting()
{
    TryGetComponent(out buildingObject);
    Root = buildingObject.parent.gameObject;
    if (Root.transform.GetChild(1).gameObject)
    {
        EffectObject = Root.transform.GetChild(1).gameObject;
        EffectObject.SetActive(false);
    }
    else Debug.LogError("건물에서 effectObject가 없습니다");

    if(this.transform.GetChild(0).TryGetComponent(out CheckBoxScript _))
    {
        OverlapObject = this.transform.GetChild(0).gameObject;
        OverlapObject.SetActive(false);
    }
    else Debug.LogError("건물에 OverlapObject가 없습니다");

    if (Root.TryGetComponent(out MeshCollider RootObjectMeshcollider)) RootObjectMeshcollider.enabled = false;
    else Debug.LogError("건물에 MeshCollider가 없습니다");
}
```

</div>
</details>

<br>

-------------


### 건물 오브젝트 내 '건축중 활성화되는 콜라이더' 설명

![imagename](/assets/image/Project/TeamProject/BuildingObjectStoryBoard/003.gif)

> 장애물이나 너무 경사진 장소에는 건축할 수 없도록, 건축 불가능 감지 콜라이더를 사용한다.

<br>

![imagename](/assets/image/Project/TeamProject/BuildingObjectStoryBoard/006.png)

> 어느정도 울퉁불퉁한 바닥에도 건축할 수 있게 지면으로 부터 약간 띄어져 있도록 콜라이더를 배치하였다. 만약 바닥에 딱 붙어있으면, 완전하게 평평한 바닥이 아니면 건축할 수 없을 것이다.


<details>
<summary>체크박스 스크립트</summary>
<div markdown="1">

```cs
public class CheckBoxScript : MonoBehaviour
{
    private LayerMask TerrainMask;
    [SerializeField]private List<GameObject> collList = new();
    private void Start()
    {
        TerrainMask = PlayerScript.instance.plMask.MaskTerrain;
    }
    private void Update()
    {
        GetComponentInParent<BuildingObjectScript>().isBoxOverlap = (collList.Count != 0); //checkBoxCount가 0이 아니면 isBoxOverlap를 트루로
    }
    private void OnTriggerEnter(Collider other)
    {
        if (((1 << other.gameObject.layer) & TerrainMask) != 0)
        {
            collList.Add(other.gameObject);
        }
    }
    private void OnTriggerExit(Collider other)
    {
        if (((1 << other.gameObject.layer) & TerrainMask) != 0)
        {
            collList.Remove(other.gameObject);
        }
    }
}

```

</div>
</details>

<br>

---------------


### 건축 위치가 공중일 때 이펙트 출력

![imagename](/assets/image/Project/TeamProject/BuildingObjectStoryBoard/004.gif)

> 건축할 위치가 하늘이라면 회색 그래픽으로 표시된다.

```cs
if (Physics.Raycast(ray, out RaycastHit hit, distanceBuilding, PlayerScript.instance.plMask.MaskTerrain))
        {
			...
        }
        else
        {
            RotateBuilding_to_RayEnd(ray);
            RenderCyan();
        }
```

### 좌클릭으로 건축 실행

![imagename](/assets/image/Project/TeamProject/BuildingObjectStoryBoard/005.gif)


> 좌클릭으로 건축을 실행할 수 있고, 우클릭으로 취소할 수 있다.

- MouseInPut_Build::BuildingProcess(Ray ray)

```cs
public void BuildingProcess(Ray ray)
{
    if (Input.GetMouseButtonDown(1))
    {
        CancelBuilding();
        return;
    }

    //레이가 땅에 닿는다면
    if (Physics.Raycast(ray, out RaycastHit hit, distanceBuilding, PlayerScript.instance.plMask.MaskTerrain))
    {
        RotateBuilding_to_HitPosition(hit.point);

        if (IsCollOverlap())    return;
        RenderGreen();

        if (Input.GetMouseButton(0))
            BuildProcess(hit);
    }
    else
    {
        RotateBuilding_to_RayEnd(ray);
        RenderCyan();
    }
}
```

건축 가능 상태(그린)일때만 좌클릭으로 건축을 실행할 수 있다. 만약 건축을 취소하고 싶다면 우클릭을 실행한다.

- MouseInPut_Build::BuildProcess()

```cs
public void BuildProcess(RaycastHit hit)
{
    RollbackMaterial();
    isBuildingStateActive = false;
    HandBuilding.GetComponent<Collider>().isTrigger = false;
    HandBuilding.transform.position = hit.point;
    HandBuilding.transform.parent = null;
    HandBuilding.GetComponentInChildren<Renderer>().sharedMaterial.color = Color.white;
    HandBuildingScript.ChangeState(BuildingObjectScript.BuildingObjectState.Runing);
    HandBuilding = null;
}
```

좌클릭을 통해 건축을 실행하면 다음과 같은 코드와 기능들이 실행된다.

건축이 시작되었을 때 교체해뒀던 메테리얼과 셰이더를 원래 건물의 것으로 복구시킨다.

Build 인스턴스의 bool값인 isBuildingStateActive을 다시 false로 돌려놓는다.

건물의 잡다한 설정이 건축중 상태였던 것을 건축 완료 이후 상태에 맞게 재설정한다.

<br>

- MouseInPut_Build::CancelBuilding()

```cs
private void CancelBuilding()
{
    RollbackMaterial();
    HandBuildingScript.ChangeState(BuildingObjectScript.BuildingObjectState.Destroy);
    HandBuilding = null;
    isBuildingStateActive = false;
}
```

우클릭을 통해 건축을 취소하면 다음과 같은 코드와 기능들이 실행된다.

건축 프리펩의 메테리얼과 셰이더를 원상복구 시킨다.

건물 오브젝트를 파괴시킨다.

건축을 위해 설정한 필드들을 다시 비우고, bool값인 isBuildingStateActive을 다시 false로 돌려놓는다.

