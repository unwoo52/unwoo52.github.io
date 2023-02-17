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
> [youtube 버프 시연 영상](https://youtu.be/XYon_3MIK5E?t=72)에서 버프 작동을 유튜브로 볼 수 있습니다.
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

### 인게임 건물 오브젝트




