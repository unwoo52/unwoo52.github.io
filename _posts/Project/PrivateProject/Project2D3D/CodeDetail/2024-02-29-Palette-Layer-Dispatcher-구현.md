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

공백부분FIXME

## 코드 설명

```csharp
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
```

Next Step버튼과 Previous Step은 isSelectLayers라는 bool 필드를 변경한다. 아래에서 각 레이어 선택 전 화면과 후 화면으로 나누어서 설명한다.


### 첫번째 화면 설명

```csharp
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
```

#### ShowTargetTilemaps()

```csharp
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
```

tileMap을 선택하는 필드를 늘리거나 줄일 수 있는 단순한 기능의 코드가 있다.

### 두번째 화면 실행

```csharp
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
```

#### ShowTopMenuZone()

```csharp
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
```

Column Count를 설정할 수 있고, AddRow버튼을 눌러서 새로운 Row를 추가할 수 있다.

columnCount는 한 Row에 몇개의 타일을 배치할지를 설정하는 필드이다. 이 필드는 5에서 20까지의 값을 가질 수 있다.

#### ShowRowData()

```csharp
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
```

ShowRowData에서는 각 Row의 데이터를 보여주고, 각 Row의 데이터를 설정할 수 있는 기능을 제공한다.

여기서 각 Row는 OtdEditorsRow 클래스의 인스턴스이다. 이 클래스는 각 Row의 데이터를 설정할 수 있는 기능을 제공한다.

이 Row클래스는 아래의 공개 함수들을 노출한다.

```csharp
public void ResetToggles();
public bool ShowColumnFields(int columnCount);
public void ShowDropdownList(Tilemap[] targetTilemaps);
public void ReturnRowData(out List<TileBase> tileBaseOuters, out List<Sprite> spriteOuters, out string tilemapNameOuter);
```

각 함수들은 다음의 역할을 수행한다.

* ResetToggles() : 현재 Row의 토글을 리셋한다.
* ShowColumnFields(int columnCount) : 현재 Row의 Column을 설정할 수 있는 필드를 보여준다.
* ShowDropdownList(Tilemap[] targetTilemaps) : 현재 Row의 tileMap을 설정할 수 있는 드롭다운을 보여준다.
* ReturnRowData(out List<TileBase> tileBaseOuters, out List<Sprite> spriteOuters, out string tilemapNameOuter) : 현재 Row의 데이터를 반환한다.

이 OtdEditorsRow 클래스는 다음 포스트에서 자세히 설명한다. [TODO링크 추가]

<br>

---

```csharp
//if some row data select toggles, reset other row data's toggles
if (selectedRowData != -1) //선택된 Row가 있을 때
{
    for (int i = 0; i < otdEditorsRows.Length; i++)
    {
        if (i != selectedRowData)
        {
            otdEditorsRows[i].ResetToggles();
        }
    }
}
```

OtdEditorsRow관련 코드 외에 다른 부분은 선택 초기화 기능 관련 코드이다.

여러 Row중 하나를 선택하면, 다른 Row의 선택을 초기화하는 기능을 제공한다. 특히, 같은 column에 있는 Row들 뿐 아니라 다른 column에 있는 Row들도 초기화해야 했기 때문에 코드가 복잡해질 뻔했다.

이를 위해 ResetToggles()를 구현하여 사용하였다.

otdEditorsRows코드의 일부를 미리 보자면 아래와 같다.

* otdEditorsRows.cs
```csharp
public void ResetToggles()
{
    //set false all boolean array elements
    for (int i = 0; i < toggles.Length; i++)
    {
        toggles[i] = false;
    }

    SceneView.duringSceneGui -= SelectTile; //이 코드의 용도는 otdEditorsRows 포스팅의 SelectTile()설명 참조
}
```

#### ShowBottomMenuZone()

[TODO 바텀메뉴존사진]

```csharp
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
```

Save버튼을 누르면 각 Row의 데이터를 수집하여, 이 데이터를 ObjectTileDispositionData 클래스의 인스턴스로 만들어서 json으로 변환하여 저장한다.

