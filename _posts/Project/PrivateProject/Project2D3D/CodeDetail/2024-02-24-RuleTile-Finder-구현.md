---
title: RuleTile Finder 구현
author: unwoo52
date: 2024-02-24 00:00:00 +09:00
categories: [Project, PrivateProject, Project2D3D, CodeDetail]
tags: [Unity, ScriptableObject, Project2D3D, Palette, Grid, RuleTile]
---

TODO - 링크 하나 적어야 함

# RuleTile Finder 구현

[링크 적기]Grid And Object MatchMaker을 구현하던 도중 문제가 생겼다.

tile grid에 그려진 TileBase에서는 ruletile에 대한 정보밖에 얻을 수 없었고, 어떤 형태의 sprite인지 알 수 없었다.

TileBase의 모든 함수를 찾아보기도 하고, TileBase를 Tile로 캐스팅 하는 방법 등 여러 방법을 시도하였지만,
TileBase는 어떤 sprite인지 디버그창에서만 보여줄 수 있을 뿐 함수나 프로퍼티로는 전혀 공개되어 노출 되어있지 않았다.

때문에 각 8방향별로 다르게 배치되어야 하는 언덕 prefab을 배치하는 것이 불가능했다.

<br>

그래서 TileBase의 Rule과, Rule별 배치되는 Sprite에 접근해서 어떤 유형의 RuleTile인지를 판별하는 Tool을 구현하였다.

<br>
<br>

## 코드

```csharp
using UnityEngine;
using UnityEngine.Tilemaps;

namespace Tool.AutoTileDisposition
{
    public static class RuleTileDistinction
    {
        public static Sprite DistinguishRuleTile(TileBase tile, Tilemap tilemap, Vector3 pos)
        {
            if (tile is not RuleTile ruleTile) return null;
            Vector3Int cellPosition = tilemap.WorldToCell(pos);

            // Evaluate each rule
            foreach (var rule in ruleTile.m_TilingRules)

            {
                bool ruleMatches = true;

                for (int i = 0; i < rule.m_Neighbors.Count; i++)
                {
                    Vector3Int neighborPosition = cellPosition + rule.m_NeighborPositions[i];
                    TileBase neighborTile = tilemap.GetTile(neighborPosition);

                    if(rule.m_Neighbors[i] == 0) continue;

                    if ((neighborTile == null && rule.m_Neighbors[i] == RuleTile.TilingRule.Neighbor.This)||
                        (neighborTile != null && rule.m_Neighbors[i] == RuleTile.TilingRule.Neighbor.NotThis))
                    {
                        ruleMatches = false;
                        break;
                    }
                }

                if (ruleMatches)
                {
                    return rule.m_Sprites[0];
                }
            }

            return null;
        }
    }
}
```


<br>

## 설명

8방향에 tile이 존재하는지 검사한 후,

<br>

## 개선 가능성

유니티에 기본으로 제공되는 룰타일은 단지 방향에 타일이 1.있는가 2.없는가 3.상관없다 만 구분할 수 있다.

때문에 더 복잡한 룰타일을 구현하고 싶다면, 룰타일을 커스텀하여 구현할 수 있다.

```csharp
public class MyTile : RuleTile {
    public string tileId;
    public bool isWater;
}
```

유니티는 RuleTile를 상속받는 클래스를 만들어 룰타일을 추가하고, 직접 규칙을 구현할 수 있도록 지원한다.

[커스텀 룰타일](https://docs.unity3d.com/Packages/com.unity.2d.tilemap.extras@1.6/manual/CustomRulesForRuleTile.html)
