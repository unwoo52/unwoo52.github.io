---
title: Random Tile Disposition 구현
author: unwoo52
date: 2024-02-29 00:00:00 +09:00
categories: [Project, JRPG-FPS, CodeDetail]
tags: [Unity, ScriptableObject, Project2D3D, Palette, Grid, Automatize]
---

# Random Tile Disposition 구현

![image](https://github.com/unwoo52/unwoo52.github.io/assets/73688472/eb6a2b98-5526-446c-a588-95c7a409f158)

![image](https://github.com/unwoo52/unwoo52.github.io/assets/73688472/89a07aba-3b09-41a7-8d31-548e97e4d45e)

한칸짜리 꽃이나 바위를 배치하는데, 이정도의 자동화는 아주 빠르게 만들 수 있다는 생각이 들어 즉석에서 만들어서 사용하였다.

* 자동화된 모습

![autoRandomTile](https://github.com/unwoo52/unwoo52.github.io/assets/73688472/5ccce8c1-04a9-4067-8970-430e8187dbe5)

<details>
<summary> RandomTileDisposition.cs 전체 코드 [접기/펼치기]</summary>
<div markdown="1">

```csharp
public class RandomTileDisposition : EditorWindow
{
    private List<TileBase> tileBases = new();
    private Tilemap targetTilemap;

    //윈도우 열기
    [MenuItem("Window/AutoTileDisposition/Random Tile Disposition")]
    public static void ShowWindow()
    {
        GetWindow<RandomTileDisposition>("Random Tile Disposition");
    }

    private void OnGUI()
    {
        //targetTilemap 입력 필드
        targetTilemap = (Tilemap)EditorGUILayout.ObjectField("Tilemap", targetTilemap, typeof(Tilemap), true);
        //경계선
        EditorGUILayout.LabelField("", GUI.skin.horizontalSlider);


        for (int i = 0; i < tileBases.Count; i++)
        {
            //horizontal start
            EditorGUILayout.BeginHorizontal();
            tileBases[i] = (TileBase)EditorGUILayout.ObjectField("TileBase", tileBases[i], typeof(TileBase), true);
            if (GUILayout.Button("Remove"))
            {
                tileBases.RemoveAt(i);
            }
            //horizontal end
            EditorGUILayout.EndHorizontal();
        }
        if (GUILayout.Button("Add"))
        {
            tileBases.Add(null);
        }


        //경계선================================
        EditorGUILayout.LabelField("", GUI.skin.horizontalSlider);


        if (GUILayout.Button("Start"))
        {
            SceneView.duringSceneGui += RandomTile;
        }

        if (GUILayout.Button("Stop"))
        {
            SceneView.duringSceneGui -= RandomTile;
        }
    }

    private void RandomTile(SceneView obj)
    {
        if (Event.current.type == EventType.MouseDown && Event.current.button == 0)
        {
            Ray ray = HandleUtility.GUIPointToWorldRay(Event.current.mousePosition);
            Vector3Int cellPosition = targetTilemap.WorldToCell(ray.origin);
            TileBase tile = tileBases[Random.Range(0, tileBases.Count)];
            targetTilemap.SetTile(cellPosition, tile);
            //history 기록
            Undo.RecordObject(targetTilemap, "Paint Random Tile");

            // 이벤트 소비
            Event.current.Use();

            //tilemap의 dirty 상태를 갱신
            EditorUtility.SetDirty(targetTilemap);
        }
    }
}
```

</div>
</details>

#### OnGUI

```csharp
private void OnGUI()
{
    //targetTilemap 입력 필드
    targetTilemap = (Tilemap)EditorGUILayout.ObjectField("Tilemap", targetTilemap, typeof(Tilemap), true);
    //경계선
    EditorGUILayout.LabelField("", GUI.skin.horizontalSlider);


    for (int i = 0; i < tileBases.Count; i++)
    {
        //horizontal start
        EditorGUILayout.BeginHorizontal();
        tileBases[i] = (TileBase)EditorGUILayout.ObjectField("TileBase", tileBases[i], typeof(TileBase), true);
        if (GUILayout.Button("Remove"))
        {
            tileBases.RemoveAt(i);
        }
        //horizontal end
        EditorGUILayout.EndHorizontal();
    }
    if (GUILayout.Button("Add"))
    {
        tileBases.Add(null);
    }


    //경계선================================
    EditorGUILayout.LabelField("", GUI.skin.horizontalSlider);


    if (GUILayout.Button("Start"))
    {
        SceneView.duringSceneGui += RandomTile;
    }

    if (GUILayout.Button("Stop"))
    {
        SceneView.duringSceneGui -= RandomTile;
    }
}
```

tile을 추가하고 줄일 수 있는 가벼운 화면이 있다.

Start와 Stop 버튼을 누르면 SceneView.duringSceneGui에 RandomTile을 추가하거나 제거함으로써 마우스를 이용해 타일을 랜덤으로 배치할 수 있게 한다.

#### RandomTile

```csharp
private void RandomTile(SceneView obj)
{
    if (Event.current.type == EventType.MouseDown && Event.current.button == 0)
    {
        Ray ray = HandleUtility.GUIPointToWorldRay(Event.current.mousePosition);
        Vector3Int cellPosition = targetTilemap.WorldToCell(ray.origin);
        TileBase tile = tileBases[Random.Range(0, tileBases.Count)];
        targetTilemap.SetTile(cellPosition, tile);
        //history 기록
        Undo.RecordObject(targetTilemap, "Paint Random Tile");

        // 이벤트 소비
        Event.current.Use();

        //tilemap의 dirty 상태를 갱신
        EditorUtility.SetDirty(targetTilemap);
    }
}
```

tileBases[]에 있는 타일 중 랜덤한 대상을 선택해 그림을 그리게 한다.

<br>

```csharp
//history 기록
Undo.RecordObject(targetTilemap, "Paint Random Tile");
```

위 코드를 이용하면 그림을 그린 히스토리를 기억해 **Ctrl + Z**로 되돌릴 수 있다.

<br>

```csharp
// 이벤트 소비
Event.current.Use();
```

위 코드를 이용하면 이벤트를 소비해 다른 이벤트가 발생하지 않도록 한다.

이 코드가 없으면 그림을 그린 뒤, 마우스 클릭 이벤트가 그대로 실행된다.
즉, 씬에 보이는 대상이 클릭되거나 하는 이벤트가 일어나기 때문에 그림을 그린 뒤, 마우스 이벤트를 종료시켜야 한다.

<br>

```csharp
//tilemap의 dirty 상태를 갱신
EditorUtility.SetDirty(targetTilemap);
```

![image](https://github.com/unwoo52/unwoo52.github.io/assets/73688472/2f501c9c-e093-4610-ba2c-f314b4a4f7f3)

이 코드를 실행하지 않으면 Scene이 변경점을 감지하지 못해서 저장과 같은 행동을 할 수 없고, 씬을 벗어날 때 저장할지를 뭍지 않는다.

dirty 상태를 갱신함으로써 Scene이 변경점을 감지하고 저장할지를 물어볼 수 있게 한다.
