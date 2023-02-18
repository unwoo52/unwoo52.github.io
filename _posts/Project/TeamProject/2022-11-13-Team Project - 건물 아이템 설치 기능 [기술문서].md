---
title: Team Project - 건물 아이템 설치 기능 [기술문서]
author: unwoo52
date: 2022-11-13 00:00:00 +09:00
categories: [Unity, TeamProj]
tags: [Unity, TeamProj, Team, Building, TechnicalDocument, Docs, Document]
---

# 기술 문서 개요

### 프로젝트, 기술문서, 스토리보드 링크

> [팀프로젝트 문서](https://unwoo52.github.io/posts/Team-Project-About/)는 프로젝트의 전체 내용과 개요 등에 대해 적혀있습니다.
>
> [youtube 건물 아이템 시연 영상](https://youtu.be/XYon_3MIK5E?t=72)에서 버프 작동을 유튜브로 볼 수 있습니다.
> 
> <span style="color:#ffffff">기술 문서는 전체 코드와 코드들에 대한 설명이 적혀있습니다.</span>
> 
> [스토리보드]()에는 해당 기능이 어떻게 게임에서 실행되는지 시간순서대로 읽기 쉽게 스토리로 작성하였습니다.
## 건물 아이템 설치 기능 개요






## 건물 관련 오브젝트들과 스크립터블 데이터

### 건물 오브젝트 프레펩

![imagename](/assets/image/Project/TeamProject/BuildingObjectSystem/003.png)

- 텐트 오브젝트.

![imagename](/assets/image/Project/TeamProject/BuildingObjectSystem/001.png)

- 모닥불 오브젝트.

![imagename](/assets/image/Project/TeamProject/BuildingObjectSystem/002.png)

- 모닥불 오브젝트는 일정 범위 내에 플레이어에게 hp를 지속적으로 회복시켜준다.


<details>
<summary>모닥불 회복 버프 스크립트</summary>
<div markdown="1">

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Player;
public class BO_CampFireScript : MonoBehaviour
{
    readonly WaitForSeconds HealDelayTime = new(5.0f);
    [SerializeField]
    private List<PlayerScript> playerList;
    private Coroutine healCoroutine;
    #region Method
    private void Start()
    {
        healCoroutine = StartCoroutine(HealPlayer());
    }
    IEnumerator HealPlayer()
    {
        while (this.gameObject != null)
        {
            foreach (PlayerScript pl in playerList)
            {
                pl.myInfo.CurHP += 5f;
            }
            yield return HealDelayTime;
        }
    }
    #endregion
    #region OnTrigger
    private void OnTriggerEnter(Collider other)
    {
        if (other.gameObject.layer == 6)
        {
            //playerList.Add(other.GetComponent<PlayerScript>());
            if (other.TryGetComponent(out PlayerScript playerscript)) playerList.Add(playerscript);
        }
    }
    private void OnTriggerExit(Collider other)
    {
        if (other.gameObject.layer == 6)
        {
            //playerList.Remove(other.GetComponent<PlayerScript>());
            if (other.TryGetComponent(out PlayerScript playerscript)) playerList.Remove(playerscript);
        }
    }
    #endregion
}
```

> OnTriggerEnter로 범위 내 플레이어들을 List<PlayerScript> playerList에 저장하여 일정 시간마다 hp를 회복시킴.
</div>
</details>

  
### 건물 아이템 스크립터블 데이터

![imagename](/assets/image/Project/TeamProject/BuildingObjectSystem/004.png)

![imagename](/assets/image/Project/TeamProject/BuildingObjectSystem/005.png)

### 인게임 아이템 오브젝트

![imagename](/assets/image/Project/TeamProject/BuildingObjectSystem/006.png)

![imagename](/assets/image/Project/TeamProject/BuildingObjectSystem/007.png)

## 스크립트

### 마우스 입력 스크립트

<details>
<summary>MouseInput</summary>
<div markdown="1">

```cs
using Definitions;
using Player;
using System.Collections;
using System.Collections.Generic;
using Unity.Burst.CompilerServices;
using UnityEngine;
using static UnityEditor.Timeline.Actions.MenuPriority;

public class MouseInput
{
    /* fields */
    #region
    [SerializeField] private MouseInPut_Build _mouseInPut_Build = new();
    [SerializeField] private GatherItem gatherItem = new();
    [SerializeField] private Mining mining = new();
    public Mining Mining { get { return mining; } }
    public MouseInPut_Build MouseInPut_Build { get { return _mouseInPut_Build; } }
    public MouseInPut_Build MouseInPut_ { get { return _mouseInPut_Build; } }
    #endregion
    public void MouseInPutProcess()
    {
        //일반 상태일 때
        if (IsNomalState())
        {
            //좌클릭
            if (Input.GetKeyDown(PlayerScript.PlayerInstance.Key.LeftClick))
                LeftMouseInPut_NomalMethod();
            //우클릭
            if (Input.GetKeyDown(PlayerScript.PlayerInstance.Key.RightClick))
                RightMouseInPut_NomalMethod();
        }

        //건물 설치 상태일 때
        if (IsBuildState())
            MouseInPut_BuildingMethod();
    }
    /* codes */
    #region .
    void MouseInPut_BuildingMethod()
    {
        _mouseInPut_Build.BuildingProcess(RayfromTransform());
    }

    void LeftMouseInPut_NomalMethod()
    {
        Ray ray = RayfromTransform();
        if (Physics.Raycast(ray, out RaycastHit hit, 1000.0f, PlayerScript.instance.plMask.MaskMouseTrigger))
        {
            if (IsResource(hit))
                mining.MiningProcess(ray, hit);
            if (IsItem(hit))
                gatherItem.Gathering(hit);
        }
        //OnAttack();
    }
    /// <summary>
    /// 일반 상태일 때 실행되는 마우스 우클릭 이벤트 매소드
    /// </summary>
    void RightMouseInPut_NomalMethod()
    {
        ...
    }
    /* codes */
    private bool IsBuildState()
    {
        return _mouseInPut_Build.IsBuildingStateActive;
    }
    private bool IsNomalState()
    {
        if (_mouseInPut_Build.IsBuildingStateActive) return false;
        return true;
    }
    /// <summary>캐릭터의 회전을 마우스 클릭 방향으로 정렬 후 크로스헤어 위치로 Ray를 발사</summary>///
    private Ray RayfromTransform()
    {
        if (!PlayerScript.PlayerInstance.State.isCurrentFp) PlayerScript.PlayerInstance.ApplyTpcamValueToFpcam();
        if (PlayerScript.PlayerInstance.State.isCurrentFp) PlayerScript.PlayerInstance.ApplyFpcamValueToTpcam();
        Ray ray = PlayerScript.PlayerInstance.Com.fpCamera.ScreenPointToRay(
            new Vector3(PlayerScript.PlayerInstance.Com.fpCamera.pixelWidth / 2, PlayerScript.PlayerInstance.Com.fpCamera.pixelHeight / 2));
        return ray;
    }

    private bool IsResource(RaycastHit hit)
    {
        return (1 << hit.transform.gameObject.layer == PlayerScript.instance.plMask.MaskResource);
    }
    private bool IsItem(RaycastHit hit)
    {
        return (1 << hit.transform.gameObject.layer == PlayerScript.instance.plMask.MaskItem);
    }
    #endregion
}
```

</div>
</details>



--------------

### 마우스 입력 - 건축 스크립트

<details>
<summary>MouseInPut_Build</summary>
<div markdown="1">

```cs
using Definitions;
using Player;
using System;
using System.Collections;
using System.Collections.Generic;
using System.Resources;
using Unity.Burst.CompilerServices;
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.UIElements;
using static UnityEngine.RuleTile.TilingRuleOutput;

public class MouseInPut_Build// : MonoBehaviour
{
    /* fields */
    #region .
    private bool isBuildingStateActive = false;
    public bool IsBuildingStateActive { get { return isBuildingStateActive; } }
    private float distanceBuilding = 10.0f;
    private GameObject HandBuilding; //건축 상태일 때 건물 오브젝트를 보관하는 공간
    private BuildingObjectScript HandBuildingScript;


    #endregion
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

    /* codes */
    #region .
    private void SwapRenderer()
    {
        Material material = Resources.Load("Prefabs/imsi") as Material;
        HandBuildingScript.ChangeMaterials(material);
        if (HandBuildingScript.GetMaterial() == null) 
        { 
            Debug.LogError("임시 Metarial로 바꾸는 데에 실패했습니다.");
            HandBuildingScript.SetOrigibMaterail();
        }

        HandBuildingScript.GetMaterial().color = new Color(1,1,1,1);
    }
    private void RenderRed()
    {
        HandBuildingScript.GetMaterial().color = new Color(1,0,0,0.5f);
        return;
    }
    private void RenderGreen()
    {
        HandBuildingScript.GetMaterial().color = new Color(0, 1, 0, 0.5f);
        return;
    }
    private void RenderCyan()
    {
        HandBuildingScript.GetMaterial().color = new Color(0.2f, 0.2f, 0.7f, 0.5f);
        return;
    }
    private void RotateBuilding_to_RayEnd(Ray ray)
    {
        HandBuilding.transform.position = ray.GetPoint(distanceBuilding);
    }
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
    private bool IsCollOverlap()
    {
        if(HandBuildingScript.isBoxOverlap)
        {
            RenderRed();
            return true;
        }
        return false;
    }
    private void RotateBuilding_to_HitPosition(Vector3 vector3)
    {
        HandBuilding.transform.position = vector3;
    }
    private void CancelBuilding()
    {
        RollbackMaterial();
        HandBuildingScript.ChangeState(BuildingObjectScript.BuildingObjectState.Destroy);
        HandBuilding = null;
        isBuildingStateActive = false;
    }
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

    private void RollbackMaterial()
    {
        
        if(!(HandBuildingScript.ChangeMaterials(HandBuildingScript._OriginMaterial)))
        {
            Debug.LogError("Material을 Origin Material로 돌려놓는데에 실패했습니다.");
        }
    }
    #endregion
}
```

</div>
</details>


