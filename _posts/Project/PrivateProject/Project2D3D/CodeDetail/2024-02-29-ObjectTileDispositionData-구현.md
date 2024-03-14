---
title: ObjectTileDispositionData 구현
author: unwoo52
date: 2024-02-29 00:00:00 +09:00
categories: [Project, PrivateProject, Project2D3D, CodeDetail]
tags: [Unity, ScriptableObject, Project2D3D, Palette, Grid, Automatize]
---

# ObjectTileDispositionData 구현

이전 포스트 [TODO RuleTile Finder 구현 링크]에서 RuleTile에 대한 정보를 얻는 Tool을 구현하기 위해 사용되는 클래스이다.

[TODO otd 데이터 한줄 사진]

위 사진의 한 줄마다 하나의 ObjectTileDispositionData(이하 otdData) 인스턴스가 존재한다.

## 전체 코드

<details>
<summary> otdData 전체 코드 [접기/펼치기]</summary>
<div markdown="1">

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using UnityEditor;
using UnityEngine;
using UnityEngine.Tilemaps;

namespace Tool.AutoTileDisposition
{
    public class OtdEditorsRow
    {
        //생성자
        public OtdEditorsRow(Tilemap[] tilemaps)
        {
            this.tilemaps = tilemaps;
        }

        ~OtdEditorsRow()
        {
            SceneView.duringSceneGui -= SelectTile;
        }

        private readonly Tilemap[] tilemaps;
        private TileBase[] tileBases = new TileBase[5];
        private Sprite[] sprites = new Sprite[5];
        private bool[] toggles = new bool[5];
        private int currentSelectTileBase;
        private string[] layerNames;
        private float[] columns;

        public string CurrentTileMapName;

        public void ResetToggles()
        {
            //set false all boolean array elements
            for (int i = 0; i < toggles.Length; i++)
            {
                toggles[i] = false;
            }

            SceneView.duringSceneGui -= SelectTile;
        }

        public bool ShowColumnFields(int columnCount)
        {
            bool result = false;

            //만약 tileBases, sprites, toggles의 길이가 columnCount와 다르다면 다시 생성
            if (tileBases.Length != columnCount || sprites.Length != columnCount || toggles.Length != columnCount)
            {
                tileBases = new TileBase[columnCount];
                sprites = new Sprite[columnCount];
                toggles = new bool[columnCount];
            }

            //toggles 길이의 새 array 생성해서 toggles의 내용을 복사
            bool[] tempToggles = new bool[columnCount];
            Array.Copy(toggles, tempToggles, columnCount);

            //toggles을 사용하는 foreach문으로 toggle 구현
            for (int i = 0; i < toggles.Length; i++)
            {
                if (sprites[i] != null)
                {
                    var rect = DrawOnGUISprite(sprites[i]); //, horizontal: 20 * i
                    //rect에 버튼
                    if (GUI.Button(rect, "", new GUIStyle()))
                    {
                        sprites[i] = null;
                        tileBases[i] = null;
                    }

                    continue;
                }

                var style = new GUIStyle(GUI.skin.button)
                {
                    normal = {textColor = toggles[i] ? Color.magenta: Color.white},
                };

                toggles[i] = EditorGUILayout.Toggle(toggles[i], style,GUILayout.Width(20));
            }

            //tempToggles과 toggles을 비교
            bool isSelectToggle = false;
            int index = -1;
            for (int j = 0; j < tempToggles.Length; j++)
            {
                if(tempToggles[j] != toggles[j])
                {
                    isSelectToggle = true;
                    index = j;
                    break;
                }
            }

            //만약 toggle 중 하나가 선택되었다면
            if(isSelectToggle)
            {
                //선택한 toggle을 제외한 toggles를 모두 false로
                for (int j = 0; j < toggles.Length; j++)
                {
                    if (j != index)
                    {
                        toggles[j] = false;
                    }
                }

                result = true;
                currentSelectTileBase = index;
                SceneView.duringSceneGui += SelectTile;
            }

            return result;
        }

        private void OnDropdownSelected(string layerName)
        {
            CurrentTileMapName = layerName;
        }

        private void SelectTile(SceneView scene)
        {
            if (Event.current.type == EventType.MouseDown && Event.current.button == 0)
            {
                Ray ray = HandleUtility.GUIPointToWorldRay(Event.current.mousePosition);

                //tilemaps의 이름들을 array로 만들기
                string[] tilemapNames = new string[tilemaps.Length];
                for (int i = 0; i < tilemaps.Length; i++)
                {
                    tilemapNames[i] = tilemaps[i].name;
                }

                //현재 선택된 타일맵 찾기
                Tilemap currentSelectTilemap = tilemaps[Array.IndexOf(tilemapNames, CurrentTileMapName)];

                //get cellPosition
                Vector3Int cellPosition = currentSelectTilemap.WorldToCell(ray.origin);

                //get tile
                TileBase tile = currentSelectTilemap.GetTile(cellPosition);

                //get tile sprite
                TileData tileData = new TileData();

                //defensive coding
                if(tile == null || tile == tileBases[currentSelectTileBase]) return;

                tile.GetTileData(cellPosition, currentSelectTilemap, ref tileData);

                tileBases[currentSelectTileBase] = tile;
                sprites[currentSelectTileBase] = tileData.sprite;
                toggles[currentSelectTileBase] = false;

                SceneView.duringSceneGui -= SelectTile;
            }
        }

        private Rect DrawOnGUISprite(Sprite sprite, float horizontal = 0, float vertical = 0)
        {
            Rect spriteRect = sprite.rect;

            Rect rect = GUILayoutUtility.GetRect(spriteRect.width + horizontal, spriteRect.height + vertical);
            if (Event.current.type == EventType.Repaint)
            {
                var tex = sprite.texture;
                spriteRect.xMin /= tex.width;
                spriteRect.xMax /= tex.width;
                spriteRect.yMin /= tex.height;
                spriteRect.yMax /= tex.height;
                GUI.DrawTextureWithTexCoords(rect, tex, spriteRect);
            }

            return rect;
        }

        public void ShowDropdownList(Tilemap[] targetTilemaps)
        {
            var rect = GUILayoutUtility.GetRect(new GUIContent(),
                EditorStyles.toolbarButton, GUILayout.Width(200));
            if (GUI.Button(rect, new GUIContent(CurrentTileMapName + "  ▼"), EditorStyles.toolbarButton))
            {
                //targetTilemaps의 이름들을 string[]으로 변환. null인 요소도 있을 수 있으니까, list로 변환해서 null인 요소는 제외하고 array로 변환함
                List<string> layerNamesList = new List<string>();
                for (int j = 0; j < targetTilemaps.Length; j++)
                {
                    if (targetTilemaps[j] != null)
                    {
                        layerNamesList.Add(targetTilemaps[j].name);
                    }
                }

                string[] layerNames = layerNamesList.ToArray();
                ShowMenu(layerNames);
            }
        }

        private void ShowMenu(string[] layerNames)
        {
            GenericMenu menu = new GenericMenu();
            for (int i = 0; i < layerNames.Length; i++)
            {
                int index = i;

                menu.AddItem(new GUIContent(layerNames[i]), false, () => OnDropdownSelected(layerNames[index]));
            }

            menu.ShowAsContext();
        }

        public void ReturnRowData(out List<TileBase> tileBases, out List<Sprite> sprites, out string tilemapName)
        {
            tileBases = this.tileBases.Where(t => t != null).ToList();
            sprites = this.sprites.Where(s => s != null).ToList();
            tilemapName = CurrentTileMapName;
            if(tileBases.Count != sprites.Count)
            {
                Debug.LogError("tileBases.Count != sprites.Count");
            }
        }
    }
}
```

</div>
</details>

## 코드 설명

#### ShowColumnFields

<details>
<summary> ShowColumnFields() 코드 [접기/펼치기]</summary>
<div markdown="1">

```csharp
public bool ShowColumnFields(int columnCount)
{
    bool result = false;

    //만약 tileBases, sprites, toggles의 길이가 columnCount와 다르다면 다시 생성
    if (tileBases.Length != columnCount || sprites.Length != columnCount || toggles.Length != columnCount)
    {
        tileBases = new TileBase[columnCount];
        sprites = new Sprite[columnCount];
        toggles = new bool[columnCount];
    }

    //toggles 길이의 새 array 생성해서 toggles의 내용을 복사
    bool[] tempToggles = new bool[columnCount];
    Array.Copy(toggles, tempToggles, columnCount);

    //toggles을 사용하는 foreach문으로 toggle 구현
    for (int i = 0; i < toggles.Length; i++)
    {
        if (sprites[i] != null)
        {
            var rect = DrawOnGUISprite(sprites[i]); //, horizontal: 20 * i
            //rect에 버튼
            if (GUI.Button(rect, "", new GUIStyle()))
            {
                sprites[i] = null;
                tileBases[i] = null;
            }

            continue;
        }

        var style = new GUIStyle(GUI.skin.button)
        {
            normal = {textColor = toggles[i] ? Color.magenta: Color.white},
        };

        toggles[i] = EditorGUILayout.Toggle(toggles[i], style,GUILayout.Width(20));
    }

    //tempToggles과 toggles을 비교
    bool isSelectToggle = false;
    int index = -1;
    for (int j = 0; j < tempToggles.Length; j++)
    {
        if(tempToggles[j] != toggles[j])
        {
            isSelectToggle = true;
            index = j;
            break;
        }
    }

    //만약 toggle 중 하나가 선택되었다면
    if(isSelectToggle)
    {
        //선택한 toggle을 제외한 toggles를 모두 false로
        for (int j = 0; j < toggles.Length; j++)
        {
            if (j != index)
            {
                toggles[j] = false;
            }
        }

        result = true;
        currentSelectTileBase = index;
        SceneView.duringSceneGui += SelectTile;
    }

    return result;
}
```

</div>
</details>

```csharp
//만약 tileBases, sprites, toggles의 길이가 columnCount와 다르다면 다시 생성
if (tileBases.Length != columnCount || sprites.Length != columnCount || toggles.Length != columnCount)
{
    tileBases = new TileBase[columnCount];
    sprites = new Sprite[columnCount];
    toggles = new bool[columnCount];
}
```

사용자가 columnCount를 변경했다면, 이에 맞게 tileBases, sprites, toggles의 길이를 변경한다.

```csharp
//toggles을 사용하는 foreach문으로 toggle 구현
for (int i = 0; i < toggles.Length; i++)
{
  if (sprites[i] != null)
  {
      var rect = DrawOnGUISprite(sprites[i]); //, horizontal: 20 * i
      //rect에 버튼
      if (GUI.Button(rect, "", new GUIStyle()))
      {
          sprites[i] = null;
          tileBases[i] = null;
      }

      continue;
  }

  var style = new GUIStyle(GUI.skin.button)
  {
      normal = {textColor = toggles[i] ? Color.magenta: Color.white},
  };

  toggles[i] = EditorGUILayout.Toggle(toggles[i], style,GUILayout.Width(20));
}
```

[TODO 한 토글에 sprite를 추가하고, 또 이를 다시 클릭해서 없애는 움짤]

만약 이 toggle이 sprite를 갖고 있다면 sprite를 그린다. 다시 버튼을 누르면 sprite와 tileBase를 null로 만들어서, 새 sprite를 입력할 수 있게 했다.

---

##### DrawOnGUISprite

```csharp
private Rect DrawOnGUISprite(Sprite sprite, float horizontal = 0, float vertical = 0)
{
    Rect spriteRect = sprite.rect;

    Rect rect = GUILayoutUtility.GetRect(spriteRect.width + horizontal, spriteRect.height + vertical);
    if (Event.current.type == EventType.Repaint)
    {
        var tex = sprite.texture;
        spriteRect.xMin /= tex.width;
        spriteRect.xMax /= tex.width;
        spriteRect.yMin /= tex.height;
        spriteRect.yMax /= tex.height;
        GUI.DrawTextureWithTexCoords(rect, tex, spriteRect);
    }

    return rect;
}
```

GUI에서는 texture만 그릴 수 있고, sprite는 그릴 수 없다. 그러나 tileBase는 palette의 아틀라스 안에 있는 sprite이기 때문에
이 함수를 구현해서 GUI에서도 sprite를 그릴 수 있게 했다.

---

```csharp
//toggles 길이의 새 array 생성해서 toggles의 내용을 복사
bool[] tempToggles = new bool[columnCount];
Array.Copy(toggles, tempToggles, columnCount);

...

//tempToggles과 toggles을 비교
bool isSelectToggle = false;
int index = -1;
for (int j = 0; j < tempToggles.Length; j++)
{
    if(tempToggles[j] != toggles[j])
    {
        isSelectToggle = true;
        index = j;
        break;
    }
}

//만약 toggle 중 하나가 선택되었다면
if(isSelectToggle)
{
    //선택한 toggle을 제외한 toggles를 모두 false로
    for (int j = 0; j < toggles.Length; j++)
    {
        if (j != index)
        {
            toggles[j] = false;
        }
    }

    result = true;
    currentSelectTileBase = index;
    SceneView.duringSceneGui += SelectTile;
}

return result;

```

이 함수는 최종적으로, 마우스로 클릭된 toggle을 반환해야 한다.toggles 길이의 새 array 생성해서 toggles의 내용을 미리 복사한 후,
이복사본을 toggles와 대조해서 어떤 toggle이 선택되었는지 확인한다. 선택된 toggle을 제외한 나머지 toggles를 모두 false로 만들고,
선택된 toggle의 index를 반환한다.

또 SceneView.duringSceneGui += SelectTile;줄을 통해 마우스로 클릭된 tile을 선택할 수 있게 한다. 아래 이어서 SelectTile 함수를 설명한다.

#### SelectTile

<details>
<summary> SelectTile() 전체 코드 [접기/펼치기]</summary>
<div markdown="1">

```csharp
private void SelectTile(SceneView scene)
{
    if (Event.current.type == EventType.MouseDown && Event.current.button == 0)
    {
    Ray ray = HandleUtility.GUIPointToWorldRay(Event.current.mousePosition);

    //tilemaps의 이름들을 array로 만들기
    string[] tilemapNames = new string[tilemaps.Length];
    for (int i = 0; i < tilemaps.Length; i++)
    {
        tilemapNames[i] = tilemaps[i].name;
    }

    //현재 선택된 타일맵 찾기
    Tilemap currentSelectTilemap = tilemaps[Array.IndexOf(tilemapNames, CurrentTileMapName)];

    //get cellPosition
    Vector3Int cellPosition = currentSelectTilemap.WorldToCell(ray.origin);

    //get tile
    TileBase tile = currentSelectTilemap.GetTile(cellPosition);

    //get tile sprite
    TileData tileData = new TileData();

    //defensive coding
    if(tile == null || tile == tileBases[currentSelectTileBase]) return;

    tile.GetTileData(cellPosition, currentSelectTilemap, ref tileData);

    tileBases[currentSelectTileBase] = tile;
    sprites[currentSelectTileBase] = tileData.sprite;
    toggles[currentSelectTileBase] = false;

    SceneView.duringSceneGui -= SelectTile;
    }
}
```

</div>
</details>

SelectTile()는 SceneView.duringSceneGui에 등록되어 사용되는 함수이다.

```csharp
private void SelectTile(SceneView scene)
{
    if (Event.current.type == EventType.MouseDown && Event.current.button == 0)
    {
        ...
```

버튼을 누르는 것을 감지하면 이하 코드를 실행한다.

```csharp
Ray ray = HandleUtility.GUIPointToWorldRay(Event.current.mousePosition);

//tilemaps의 이름들을 array로 만들기
string[] tilemapNames = new string[tilemaps.Length];
for (int i = 0; i < tilemaps.Length; i++)
{
    tilemapNames[i] = tilemaps[i].name;
}

//현재 선택된 타일맵 찾기
Tilemap currentSelectTilemap = tilemaps[Array.IndexOf(tilemapNames, CurrentTileMapName)];

//get cellPosition
Vector3Int cellPosition = currentSelectTilemap.WorldToCell(ray.origin);

//get tile
TileBase tile = currentSelectTilemap.GetTile(cellPosition);

//get tile sprite
TileData tileData = new TileData();

//defensive coding
if(tile == null || tile == tileBases[currentSelectTileBase]) return;

tile.GetTileData(cellPosition, currentSelectTilemap, ref tileData);

tileBases[currentSelectTileBase] = tile;
sprites[currentSelectTileBase] = tileData.sprite;
toggles[currentSelectTileBase] = false;

SceneView.duringSceneGui -= SelectTile;
```

위 toggle에서 설정한 int currentSelectTileBase을 바탕으로 각 tileBase[], sprite[], toggle[]의 해당 인덱스에
마우스를 통해 선택된 tileBase와 sprite를 저장한 뒤 toggles boolean값을 false로 변경한다.

#### ShowDropdownList

```csharp
public void ShowDropdownList(Tilemap[] targetTilemaps)
{
    var rect = GUILayoutUtility.GetRect(new GUIContent(),
        EditorStyles.toolbarButton, GUILayout.Width(200));
    if (GUI.Button(rect, new GUIContent(CurrentTileMapName + "  ▼"), EditorStyles.toolbarButton))
    {
        //targetTilemaps의 이름들을 string[]으로 변환. null인 요소도 있을 수 있으니까, list로 변환해서 null인 요소는 제외하고 array로 변환함
        List<string> layerNamesList = new List<string>();
        for (int j = 0; j < targetTilemaps.Length; j++)
        {
            if (targetTilemaps[j] != null)
            {
                layerNamesList.Add(targetTilemaps[j].name);
            }
        }

        string[] layerNames = layerNamesList.ToArray();
        ShowMenu(layerNames);
    }
}
```

[TODO 열 하나짜리 사진,, ground ↓ , ㅁㅁㅁㅁ, Remove 이 부분 중 DropDown이 열린 사진]

tileMap을 선택할 수 있는 DropDown을 보여주는 함수이다. 이를 통해 마우스로 선택할 대상 tileMap을 선택할 수 있다.

만약 tileMap이 설정되지 않은 채 마우스 클릭으로 tile을 선택하려고 하면 곂쳐져 있는 여러 tileMap의 tile 중 어떤 tile을 선택할지 모호해진다.

때문에 이 드롭다운으로 대상 tileMap을 선택할 수 있게 해야 한다.

<br>

```csharp
private void ShowMenu(string[] layerNames)
{
    GenericMenu menu = new GenericMenu();
    for (int i = 0; i < layerNames.Length; i++)
    {
        int index = i;

        menu.AddItem(new GUIContent(layerNames[i]), false, () => OnDropdownSelected(layerNames[index]));
    }

    menu.ShowAsContext();
}
```

ShowDropdownList에서 호출되는 함수로, 드롭다운을 보여주는 함수이다.

#### ReturnRowData

* ObjectTileDispositionDataEditor.cs의 ShowBottomMenuZone()함수의 ReturnRowData()함수 호출 부분
```csharp
private void ShowBottomMenuZone()
    {
        //path input field
        path = EditorGUILayout.TextField("file name : ", path);

        //save button
        if (GUILayout.Button("Save"))

        ...

        otdEditorsRow.ReturnRowData(out List<TileBase> tileBaseOuters, out List<Sprite> spriteOuters, out string tilemapNameOuter);

        ...
```

위 코드에서 호출되는 함수이다.

```csharp
public void ReturnRowData(out List<TileBase> tileBases, out List<Sprite> sprites, out string tilemapName)
{
    tileBases = this.tileBases.Where(t => t != null).ToList();
    sprites = this.sprites.Where(s => s != null).ToList();
    tilemapName = CurrentTileMapName;
    if(tileBases.Count != sprites.Count)
    {
        Debug.LogError("tileBases.Count != sprites.Count");
    }
}
```

현재 선택된 tileMap의 tileBase와 sprite를 반환하는 함수이다.

#### ResetToggles와 소멸자

```csharp
~OtdEditorsRow()
        {
            SceneView.duringSceneGui -= SelectTile;
}

public void ResetToggles()
{
    //set false all boolean array elements
    for (int i = 0; i < toggles.Length; i++)
    {
        toggles[i] = false;
    }

    SceneView.duringSceneGui -= SelectTile;
}
```

마우스로 tile을 선택하려고 넣어두었던 콜백을 제거하는 함수이다.

이 중 ResetToggles는 사용자가 새로운 tile을 선택할 때 이전에 클릭해뒀던 박스의 체크를 해제할 때 호출된다.
