---
title: Palette Layer Dispatcher 구현
author: unwoo52
date: 2024-02-29 00:00:00 +09:00
categories: [Project, PrivateProject, Project2D3D, CodeDetail]
tags: [Unity, ScriptableObject, Project2D3D, Palette, Grid, Automatize]
---

# Palette Layer Dispatcher 구현

[TODO사진 추가 레이어별 타일 그리드 하이라키]

게임 내에는 다양한 크기의 구조물 오브젝트가 타일맵에 그려져있다. 그 중 2x3 나무나 3x2의 커다란 바위같은 경우도 있는데,
나무의 같은 경우에는 밑둥에만 충돌 처리가 있고, 윗부분은 충돌처리가 없어야 한다.

때문에 여러개의 타일맵을 만들고, 하나의 타일맵에만 충돌처리를 하게 하며 여러 레이어로 나눠서 구현하였다.

그러나 이 나무를 그리기 위해서 충돌 레이어에 밑둥을 한번, 윗부분을 비충돌 레이어에 한번 더 그리는 복잡한 작업을 수행해야 한다.

[TODO일일히 그림을 그리는 복잡한 움짤 구현]

이런 작업을 자동화하기 위해, 레이어별로 타일맵을 그리는 작업을 자동화하는 툴을 만들었다.

## 전체 코드

<details>
<summary>전체 코드 [접기/펼치기]</summary>
<div markdown="1">

> 저장을 위한 Serializable 클래스

```csharp
[Serializable]
public class ObjectTileDispositionData
{
    //생성자
    public ObjectTileDispositionData(TileBase[] tileBases, Sprite[] sprites, Vector2Int[] tilePositions, string[] tilemapNames)
    {
        this.tileBases = tileBases;
        this.sprites = sprites;
        this.tilePositions = tilePositions;
        this.tilemapNames = tilemapNames;
    }

    public TileBase[] tileBases;
    public Sprite[] sprites;
    public Vector2Int[] tilePositions;
    public string[] tilemapNames;

    //json으로 변환
    public string ToJson()
    {
        return JsonUtility.ToJson(this);
    }

    //json을 ObjectTileDispositionData로 변환
    public static ObjectTileDispositionData FromJson(string json)
    {
        return JsonUtility.FromJson<ObjectTileDispositionData>(json);
    }
}
```

> 배치 데이터를 만들기 위한 EditorWindow

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using Tool.AutoTileDisposition;
using UnityEditor;
using UnityEngine;
using UnityEngine.Tilemaps;

public class ObjectTileDispositionDataEditor : EditorWindow
{
    private bool isSelectLayers;
    private int columnCount;
    public Tilemap[] targetTilemaps = new Tilemap[0];
    private OtdEditorsRow[] otdEditorsRows = new OtdEditorsRow[0];

    //윈도우 열기
    [MenuItem("Window/AutoTileDisposition/Object Tile Disposition Data Editor")]
    public static void ShowWindow()
    {
        GetWindow<ObjectTileDispositionDataEditor>("Object Tile Disposition Data Editor");
    }

    //윈도우 닫힐 때
    private void OnDestroy()
    {
        otdEditorsRows = null;
    }

    private void OnGUI()
    {
        //레이어를 선택하기 전 화면
        if (!isSelectLayers)
        {
            ShowTargetTilemaps();

            //isSelectLayers를 true로 변경하는 버튼
            if (GUILayout.Button("Next Step") && targetTilemaps.Length > 0)
            {
                if (targetTilemaps.All(tilemap => tilemap != null))
                    isSelectLayers = true;
                else
                    EditorUtility.DisplayDialog("Error", "There is a null in targetTilemaps", "OK");
            }
        }
        //레이어를 선택한 후 화면
        else
        {
            ShowTopMenuZone();

            ShowRowData();

            ShowBottomMenuZone();

            //isSelectLayers를 false로 변경하는 버튼
            if (GUILayout.Button("Previous Step"))
            {
                isSelectLayers = false;
            }
        }
    }

    private string path = "ObjectTileDispositionData";
    private void ShowBottomMenuZone()
    {
        //path input field
        path = EditorGUILayout.TextField("file name : ", path);

        //save button
        if (GUILayout.Button("Save"))
        {
            List<TileBase> tileBases = new();
            List<Sprite> sprites = new();
            List<Vector2Int> tilePositions = new();
            List<string> tilemapNames = new();
            for (var i = 0; i < otdEditorsRows.Length; i++)
            {
                var otdEditorsRow = otdEditorsRows[i];

                otdEditorsRow.ReturnRowData(out List<TileBase> tileBaseOuters, out List<Sprite> spriteOuters, out string tilemapNameOuter);

                for (int j = 0; j < tileBaseOuters.Count; j++)
                {
                    tileBases.Add(tileBaseOuters[j]);
                    sprites.Add(spriteOuters[j]);
                    tilemapNames.Add(tilemapNameOuter);
                    //y는 반대 순서로 저장. editor window에서 보이는건 위에서 아래이지만, tile이 배치되는 좌표는 아래서 위이기 때문
                    tilePositions.Add(new Vector2Int(j,otdEditorsRows.Length - i - 1));
                }
            }

            var objectTileDispositionData =
                new ObjectTileDispositionData(tileBases.ToArray(), sprites.ToArray(), tilePositions.ToArray(), tilemapNames.ToArray());
            var json = objectTileDispositionData.ToJson();
            var path = EditorUtility.SaveFilePanel("Save", Application.dataPath, "ObjectTileDispositionData",
                "json");
            if (path.Length != 0)
            {
                System.IO.File.WriteAllText(path, json);
            }
        }


        EditorGUILayout.LabelField("", GUI.skin.horizontalSlider);
    }

    private void ShowTopMenuZone()
    {
        //rect title "Object Tile Disposition Data Editor"
        EditorGUILayout.LabelField("Object Tile Disposition Data Editor", EditorStyles.boldLabel);

        EditorGUILayout.BeginHorizontal();

        columnCount = EditorGUILayout.IntField("Column Count", columnCount);

        if (GUILayout.Button("+") && columnCount < 20)
        {
            columnCount++;
        }

        if (GUILayout.Button("-") && columnCount > 5)
        {
            columnCount--;
        }

        columnCount = Mathf.Clamp(columnCount, 5, 20);

        EditorGUILayout.EndHorizontal();

        EditorGUILayout.LabelField("", GUI.skin.horizontalSlider);
    }

    private void ShowRowData()
    {
        //rect title "Set Tile Pallet"
        EditorGUILayout.LabelField("Set Tile Pallet", EditorStyles.boldLabel);


        //Show Row Data, and select row's currentSelectTileBase
        int selectedRowData = -1;
        for (var i = 0; i < otdEditorsRows.Length; i++)
        {
            EditorGUILayout.BeginHorizontal();

            var row = otdEditorsRows[i];

            //Show Dropdown button
            row.ShowDropdownList(targetTilemaps);

            //show Column Fields
            if (row.ShowColumnFields(columnCount))
            {
                selectedRowData = i;
            }

            //오른쪽 끝에 Remove 버튼
            GUILayout.FlexibleSpace();
            if (GUILayout.Button("Remove", GUILayout.Width(150)))
            {
                ArrayUtility.RemoveAt(ref otdEditorsRows, i);
            }

            EditorGUILayout.EndHorizontal();
        }

        //if some row data select toggles, reset other row data's toggles
        if (selectedRowData != -1)
        {
            for (int i = 0; i < otdEditorsRows.Length; i++)
            {
                if (i != selectedRowData)
                {
                    otdEditorsRows[i].ResetToggles();
                }
            }
        }

        //Add Button
        if (GUILayout.Button("Add Row"))
        {
            var newRowData = new OtdEditorsRow(targetTilemaps);
            ArrayUtility.Add(ref otdEditorsRows, newRowData);
            newRowData.CurrentTileMapName = targetTilemaps[0].name;
        }

        EditorGUILayout.LabelField("", GUI.skin.horizontalSlider);
    }

    private void ShowTargetTilemaps()
    {
        //rect title "Set Target Tilemaps"
        EditorGUILayout.LabelField("Set Target Tilemaps", EditorStyles.boldLabel);

        for (var i = 0; i < targetTilemaps.Length; i++)
        {
            var targetTilemap = targetTilemaps[i];
            EditorGUILayout.BeginHorizontal();

            targetTilemaps[i] =
                (Tilemap)EditorGUILayout.ObjectField("Tilemap", targetTilemap, typeof(Tilemap), true);
            if (GUILayout.Button("Remove"))
            {
                ArrayUtility.RemoveAt(ref targetTilemaps, i);
            }

            EditorGUILayout.EndHorizontal();
        }

        //Add Button
        if (GUILayout.Button("Add"))
        {
            ArrayUtility.Add(ref targetTilemaps, null);
        }

        EditorGUILayout.LabelField("", GUI.skin.horizontalSlider);
    }
}
```

</div>
</details>
