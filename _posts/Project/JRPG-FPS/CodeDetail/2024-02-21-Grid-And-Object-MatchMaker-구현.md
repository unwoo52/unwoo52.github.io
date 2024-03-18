---
title: Grid And Object MatchMaker 구현
author: unwoo52
date: 2024-02-21 00:00:00 +09:00
categories: [Project, JRPG-FPS, CodeDetail]
tags: [Unity, ScriptableObject, Project2D3D, Automatize]
---

![GridAndObjectMatchMaker](https://github.com/unwoo52/unwoo52.github.io/assets/73688472/39d71682-5edc-4247-a3ff-b0ddf820215d)



# Grid And Object MatchMaker 구현

tile grid에 배치된 sprite에 매치되는 3d prefab을 서로 짝지을 수 있는 화면을 구현하고,
이 매치를 저장하고 불러올 수 있는 기능을 구현하였다.

가령 tile palette의 초록 풀 바닥 타일과, 3d prefab인 초록 풀 오브젝트를 짝지은 뒤 배치 기능을 실행하면
tile palette에 배치된 초록 풀 바닥 타일에 위치에 대응되는 3d공간에 3d prefab인 초록 풀 오브젝트가 자동으로 배치된다.


---

## 결과물

> tile의 sprite와 대칭되는 3d prefab을 배치할 수 있는 EditorWindow의 모습

TODO editor사진

> EditorWindow를 이용해 자동으로 타일을 배치한 전 후 모습

grid사진 TODO 배치후 사진

EditorWindow로 각 tile의 sprite들과 3d prefab들을 짝지어서 작업을 실행시키면 자동으로 tileGrid에 배치된 언덕과 동일하게 3d prefab들이 자동으로 배치된다.

---

## 전체 코드

<details>
<summary>전체 코드 [접기/펼치기]</summary>
<div markdown="1">

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using Tool.AutoTileDisposition;
using UnityEditor;
using UnityEngine;
using UnityEngine.Tilemaps;
using Object = System.Object;

[Serializable]
public class TileBatchWindowData
{
    public string[] prefabPathArray;
    public Vector3Int[] quaternionArray;
    public Vector3[] positionArray;
    public string[] texturePathArray;
    public string[] spriteNameArray;
}

public class GridAnd3dObjectMatchMaker : EditorWindow
{
    //좌측화면 우측화면 비율
    private Vector2Int leftRightRatio = new(3, 1);
    private Tilemap targetTilemap;

    private Transform tileObjectParent;

    private List<(GameObject prefab, Vector3Int rotation, Vector3 position)> textureList = new();
    private List<Sprite> spriteList = new();

    private Vector2 scrollPosition;

    [MenuItem("Window/AutoTileDisposition/Grid And 3d Object MatchMaker")]
    public static void ShowWindow()
    {
        GetWindow<GridAnd3dObjectMatchMaker>("Grid And 3d Object MatchMaker");
    }

    private void OnGUI()
    {
        scrollPosition = EditorGUILayout.BeginScrollView(scrollPosition);
        int highestHeight = Math.Max(textureList.Count, spriteList.Count);

        targetTilemap = (Tilemap)EditorGUILayout.ObjectField("Tilemap", targetTilemap, typeof(Tilemap), true);
        tileObjectParent =
            (Transform)EditorGUILayout.ObjectField("TileObjectParent", tileObjectParent, typeof(Transform), true);

        // Add GUI for textureList and spriteList
        EditorGUILayout.BeginHorizontal();
        EditorGUILayout.BeginVertical();
        GUILayout.Label("Texture List", EditorStyles.boldLabel);
        for (int i = 0; i < textureList.Count; i++)
        {
            Rect prefabRect = new Rect(30, 150 + (i * 100),
                (EditorGUIUtility.currentViewWidth * (0.7f)) - 60,
                100);

            GUILayout.BeginArea(prefabRect);
            EditorGUILayout.BeginHorizontal();
            GameObject prefab =
                (GameObject)EditorGUILayout.ObjectField("Prefab", textureList[i].Item1, typeof(GameObject), true);
            Vector3Int rotation = EditorGUILayout.Vector3IntField("Rotation", textureList[i].Item2);
            Vector3 positon = EditorGUILayout.Vector3Field("Position", textureList[i].Item3);
            EditorGUILayout.EndHorizontal();
            textureList[i] = (prefab, rotation, positon);
            EditorGUILayout.BeginHorizontal();
            GUI.enabled = i > 0;
            if (GUILayout.Button("▲"))
            {
                (textureList[i], textureList[i - 1]) = (textureList[i - 1], textureList[i]);
            }

            GUI.enabled = i < textureList.Count - 1;
            if (GUILayout.Button("▼"))
            {
                (textureList[i], textureList[i + 1]) = (textureList[i + 1], textureList[i]);
            }

            GUI.enabled = true;
            EditorGUILayout.EndHorizontal();

            GUIStyle greenBoxStyle = new GUIStyle(GUI.skin.box);
            greenBoxStyle.normal.background = EditorGUIUtility.whiteTexture;
            GUILayout.Box("", greenBoxStyle, GUILayout.ExpandWidth(true), GUILayout.Height(5));

            GUILayout.EndArea();

        }

        EditorGUILayout.EndVertical();

        EditorGUILayout.BeginVertical();
        GUILayout.Label("Sprite List", EditorStyles.boldLabel);
        for (int i = 0; i < spriteList.Count; i++)
        {
            Rect spriteRect = new Rect(EditorGUIUtility.currentViewWidth * 0.7f + 30, 150 + (i * 100),
                (EditorGUIUtility.currentViewWidth * 0.3f) - 60,
                100);

            GUILayout.BeginArea(spriteRect);

            spriteList[i] = (Sprite)EditorGUILayout.ObjectField(spriteList[i], typeof(Sprite), true);
            GUILayout.Label(" ");
            EditorGUILayout.BeginHorizontal();
            GUI.enabled = i > 0;
            if (GUILayout.Button("▲"))
            {
                (spriteList[i], spriteList[i - 1]) = (spriteList[i - 1], spriteList[i]);
            }

            GUI.enabled = i < spriteList.Count - 1;
            if (GUILayout.Button("▼"))
            {
                (spriteList[i], spriteList[i + 1]) = (spriteList[i + 1], spriteList[i]);
            }

            GUI.enabled = true;

            EditorGUILayout.EndHorizontal();

            GUIStyle greenBoxStyle = new GUIStyle(GUI.skin.box);

            greenBoxStyle.normal.background = EditorGUIUtility.whiteTexture;
            GUILayout.Box("", greenBoxStyle, GUILayout.ExpandWidth(true), GUILayout.Height(5));

            GUILayout.EndArea();
        }



        EditorGUILayout.EndVertical();
        EditorGUILayout.EndHorizontal();

        EditorGUILayout.BeginHorizontal();

        if (GUILayout.Button("Batch Tiles", new GUIStyle(GUI.skin.button){normal = {textColor = Color.yellow}}))
        {
            for (int i = 0; i < spriteList.Count; i++)
            {
                {
                    BatchTiles(textureList[i].Item1, textureList[i].Item2, textureList[i].Item3, spriteList[i]);
                }
            }

            string sceneName = UnityEngine.SceneManagement.SceneManager.GetActiveScene().name;
            string date = DateTime.Now.ToString("yyyy_MM_dd");
            string fileName = $"{sceneName}_{date}";
            string directoryPath = "Assets/Data/TileDispositionData/AutoSave";
            string filePath = Path.Combine(directoryPath, fileName);

            // 디렉토리가 없으면 생성
            if (!Directory.Exists(directoryPath))
            {
                Directory.CreateDirectory(directoryPath);
            }

            Save(filePath);
        }

        EditorGUILayout.EndHorizontal();
        EditorGUILayout.BeginHorizontal();

        if (GUILayout.Button("Add Texture and Sprite",new GUIStyle(GUI.skin.button){normal = {textColor = Color.green}}))
        {
            textureList.Add((null, Vector3Int.zero, Vector3.zero));
            spriteList.Add(null);
        }

        if (spriteList.Count > 0 && GUILayout.Button("Remove Texture and Sprite",new GUIStyle(GUI.skin.button){normal = {textColor = Color.red}}))
        {
            textureList.RemoveAt(textureList.Count - 1);
            spriteList.RemoveAt(spriteList.Count - 1);
        }

        if (GUILayout.Button("Save"))
        {
            Save();
        }

        if (GUILayout.Button("Load"))
        {
            Load();
        }
        EditorGUILayout.EndHorizontal();

        //rect들은 guilayout의 scroll에 포함되지 않기 때문에 스크롤뷰가 짧게 나옴. 때문에 rect들만한 높이의 박스를 넣어서 스크롤 뷰가 길어지게 해야 함.
        float scrollViewHeight = 300 + (highestHeight * 100);
        GUILayout.Box("", GUILayout.Width(0), GUILayout.Height(scrollViewHeight));

        EditorGUILayout.EndScrollView();
    }

    private void Save(string path = null)
    {
        var data = new TileBatchWindowData();
        data.prefabPathArray = new String[textureList.Count];
        data.quaternionArray = new Vector3Int[textureList.Count];
        data.positionArray = new Vector3[textureList.Count];
        data.spriteNameArray = new string[spriteList.Count];

        for (int i = 0; i < textureList.Count; i++)
        {
            if (textureList[i].Item1 != null)
                data.prefabPathArray[i] = AssetDatabase.GetAssetPath(textureList[i].Item1);
            data.quaternionArray[i] = textureList[i].Item2;
            data.positionArray[i] = textureList[i].Item3;
        }

        data.texturePathArray = new String[spriteList.Count];
        for (int i = 0; i < spriteList.Count; i++)
        {
            if (spriteList[i] != null)
            {
                data.texturePathArray[i] = AssetDatabase.GetAssetPath(spriteList[i]);
                data.spriteNameArray[i] = spriteList[i].name;
            }
        }

        string json = JsonUtility.ToJson(data);

        // 파일 경로에 json 데이터 저장
        if (path != null)
        {
            //이미 같은 이름의 파일이 있다면 이름 뒤에 숫자를 붙여서 저장
            int i = 1;
            string tempPath = path + ".json";
            while (File.Exists(tempPath))
            {
                tempPath = path + "_" + i++ + ".json";
            }

            File.WriteAllText(tempPath, json);
            AssetDatabase.Refresh();
        }
        else
        {
            path = EditorUtility.SaveFilePanel("Save Texture and Sprite Lists", "", "TileBatchWindowData", "json");
            if (path.Length != 0)
            {
                File.WriteAllText(path, json);
            }
        }
    }

    private void Load()
    {
        //load전에 초기화
        textureList = new List<(GameObject, Vector3Int, Vector3)>();
        spriteList = new List<Sprite>();

        string path = EditorUtility.OpenFilePanel("Load Texture and Sprite Lists", "", "json");
        if (path.Length != 0)
        {
            //파일을 json으로 읽어오기
            string[] jsons = File.ReadAllLines(path);
            TileBatchWindowData data = JsonUtility.FromJson<TileBatchWindowData>(jsons[0]);

            //textureList에 data의 prefabArray와 quaternionArray를 복사
            textureList = new List<(GameObject, Vector3Int, Vector3)>();
            for (int i = 0; i < data.prefabPathArray.Length; i++)
            {
                GameObject prefab = AssetDatabase.LoadAssetAtPath<GameObject>(data.prefabPathArray[i]);
                textureList.Add((prefab, data.quaternionArray[i], data.positionArray[i]));
            }

            for (var i = 0; i < data.texturePathArray.Length; i++)
            {
                var spritePath = data.texturePathArray[i];
                Texture2D texture2D = AssetDatabase.LoadAssetAtPath<Texture2D>(spritePath);
                Object[] objs = AssetDatabase.LoadAllAssetsAtPath(spritePath);
                //spriteNameArray에 저장된 이름과 같은 이름을 가진 sprite를 찾아서 spriteList에 추가
                foreach (var obj in objs)
                {
                    if (obj is Sprite sprite && sprite.name == data.spriteNameArray[i])
                    {
                        spriteList.Add(sprite);
                        break;
                    }
                }
            }
        }
    }

    private void BatchTiles(GameObject objectToInstantiate, Vector3Int rotationOffset, Vector3 positionOffset, Sprite targetSprite)
    {
        if (targetTilemap == null || objectToInstantiate == null)
        {
            Debug.LogError("Tilemap or objectToInstantiate is not set.");
            return;
        }

        if (tileObjectParent == null)
        {
            tileObjectParent = new GameObject("TileObjects").transform;
        }

        foreach (var pos in targetTilemap.cellBounds.allPositionsWithin)
        {
            TileBase tile = targetTilemap.GetTile(pos);

            if(tile == null) continue;

            if (tile is not RuleTile)
            {
                //get tile data
                TileData tileData = new TileData();
                tile.GetTileData(pos, targetTilemap, ref tileData);

                if (tileData.sprite == targetSprite)
                {
                    Vector3 worldPos = targetTilemap.CellToWorld(pos);
                    // Use y position as z position
                    worldPos.z = worldPos.y;
                    worldPos.y = 0;
                    //rotation을 quaternion으로 변환
                    Quaternion quaternion = Quaternion.Euler(rotationOffset);
                    GameObject newTileObject = Instantiate(objectToInstantiate, worldPos * 2 + positionOffset, quaternion);
                    newTileObject.transform.parent = tileObjectParent;
                }

                continue;
            }

            var sprite = RuleTileDistinction.DistinguishRuleTile(tile, targetTilemap, pos);

            if (tile != null && sprite == targetSprite)
            {
                Vector3 worldPos = targetTilemap.CellToWorld(pos);
                // Use y position as z position
                worldPos.z = worldPos.y;
                worldPos.y = 0;
                //rotation을 quaternion으로 변환
                Quaternion quaternion = Quaternion.Euler(rotationOffset);
                GameObject newTileObject = Instantiate(objectToInstantiate, worldPos * 2 + positionOffset, quaternion);
                newTileObject.transform.parent = tileObjectParent;
            }
        }
    }
}
```

</div>
</details>

---

## 기본 기능(설정 및 버튼) 설명

![image](https://github.com/unwoo52/unwoo52.github.io/assets/73688472/db777182-71dd-48e1-a6aa-da4836198698)


```csharp
targetTilemap = (Tilemap)EditorGUILayout.ObjectField("Tilemap", targetTilemap, typeof(Tilemap), true);
tileObjectParent =
    (Transform)EditorGUILayout.ObjectField("TileObjectParent", tileObjectParent, typeof(Transform), true);
```

Tilemap과 TileObjectParent를 설정할 수 있는 필드를 만든다.

---

```csharp
if (GUILayout.Button("Add Texture and Sprite",new GUIStyle(GUI.skin.button){normal = {textColor = Color.green}}))
{
    textureList.Add((null, Vector3Int.zero, Vector3.zero));
    spriteList.Add(null);
}

if (spriteList.Count > 0 && GUILayout.Button("Remove Texture and Sprite",new GUIStyle(GUI.skin.button){normal = {textColor = Color.red}}))
{
    textureList.RemoveAt(textureList.Count - 1);
    spriteList.RemoveAt(spriteList.Count - 1);
}
```

설정 화면 아래쪽에 위치한 버튼을 통해 매치할 대상을 늘리거나 줄일 수 있다.

## 타일 매치 부분 설명

```csharp
for (int i = 0; i < textureList.Count; i++)
        {
            ...


for (int i = 0; i < spriteList.Count; i++)
        {
            ...
```
각각 textureList와 spriteList에 있는 요소의 수 만큼 for문을 돌려 아래 코드를 화면에 표시한다.

```csharp
// textureList의 경우
Rect prefabRect = new Rect(30, 150 + (i * 100),
                (EditorGUIUtility.currentViewWidth * (0.7f)) - 60,
                100);

GUILayout.BeginArea(prefabRect);
EditorGUILayout.BeginHorizontal();
GameObject prefab =
    (GameObject)EditorGUILayout.ObjectField("Prefab", textureList[i].Item1, typeof(GameObject), true);
Vector3Int rotation = EditorGUILayout.Vector3IntField("Rotation", textureList[i].Item2);
Vector3 positon = EditorGUILayout.Vector3Field("Position", textureList[i].Item3);
EditorGUILayout.EndHorizontal();
textureList[i] = (prefab, rotation, positon);
EditorGUILayout.BeginHorizontal();
GUI.enabled = i > 0;
if (GUILayout.Button("▲"))
{
    (textureList[i], textureList[i - 1]) = (textureList[i - 1], textureList[i]);
}

GUI.enabled = i < textureList.Count - 1;
if (GUILayout.Button("▼"))
{
    (textureList[i], textureList[i + 1]) = (textureList[i + 1], textureList[i]);
}

GUI.enabled = true;
EditorGUILayout.EndHorizontal();

GUIStyle greenBoxStyle = new GUIStyle(GUI.skin.box);
greenBoxStyle.normal.background = EditorGUIUtility.whiteTexture;
GUILayout.Box("", greenBoxStyle, GUILayout.ExpandWidth(true), GUILayout.Height(5));

GUILayout.EndArea();
```

Button("▲")과 Button("▼")을 통해 각 요소를 위아래로 이동할 수 있도록 하였다.

3d prefab부분에는 위치와 회전을 조정할 수 있는 offset을 입력할 필드를 추가하였다. 반대로 sprite는 sprite뿐이다. sprite가 있는 tile grid를 바탕으로
3d prefab을 배치하기 때문이다.

---

## 배치 Save 및 Load 부분 설명

Json으로 저장하기 위한 팜플랫 클래스를 정의한다.

```csharp
[Serializable]
public class TileBatchWindowData
{
    public string[] prefabPathArray;
    public Vector3Int[] quaternionArray;
    public Vector3[] positionArray;
    public string[] texturePathArray;
    public string[] spriteNameArray;
}
```

배치를 할 때 자동으로 저장하는 기능도 구현하였다. filePath에 null을 만들면 자동으로 이름을 지정해준다. 아래 Save()함수 설명 부분 참고

```csharp
if (GUILayout.Button("Batch Tiles", new GUIStyle(GUI.skin.button){normal = {textColor = Color.yellow}}))
{
    ...

    Save(filePath);
}
```

* 자동 저장된 파일들. 씬이름_날짜_인덱스

![image](https://github.com/unwoo52/unwoo52.github.io/assets/73688472/b187b635-f99b-40e0-819a-b871c1ef6d45)

---

<details>
<summary>Load and Save 전체 코드 [접기/펼치기]</summary>
<div markdown="1">

```csharp
private void Save(string path = null)
    {
        var data = new TileBatchWindowData();
        data.prefabPathArray = new String[textureList.Count];
        data.quaternionArray = new Vector3Int[textureList.Count];
        data.positionArray = new Vector3[textureList.Count];
        data.spriteNameArray = new string[spriteList.Count];

        for (int i = 0; i < textureList.Count; i++)
        {
            if (textureList[i].Item1 != null)
                data.prefabPathArray[i] = AssetDatabase.GetAssetPath(textureList[i].Item1);
            data.quaternionArray[i] = textureList[i].Item2;
            data.positionArray[i] = textureList[i].Item3;
        }

        data.texturePathArray = new String[spriteList.Count];
        for (int i = 0; i < spriteList.Count; i++)
        {
            if (spriteList[i] != null)
            {
                data.texturePathArray[i] = AssetDatabase.GetAssetPath(spriteList[i]);
                data.spriteNameArray[i] = spriteList[i].name;
            }
        }

        string json = JsonUtility.ToJson(data);

        // 파일 경로에 json 데이터 저장
        if (path != null)
        {
            //이미 같은 이름의 파일이 있다면 이름 뒤에 숫자를 붙여서 저장
            int i = 1;
            string tempPath = path + ".json";
            while (File.Exists(tempPath))
            {
                tempPath = path + "_" + i++ + ".json";
            }

            File.WriteAllText(tempPath, json);
            AssetDatabase.Refresh();
        }
        else
        {
            path = EditorUtility.SaveFilePanel("Save Texture and Sprite Lists", "", "TileBatchWindowData", "json");
            if (path.Length != 0)
            {
                File.WriteAllText(path, json);
            }
        }
    }
```

```csharp
private void Load()
{
    //load전에 초기화
    textureList = new List<(GameObject, Vector3Int, Vector3)>();
    spriteList = new List<Sprite>();

    string path = EditorUtility.OpenFilePanel("Load Texture and Sprite Lists", "", "json");
    if (path.Length != 0)
    {
        //파일을 json으로 읽어오기
        string[] jsons = File.ReadAllLines(path);
        TileBatchWindowData data = JsonUtility.FromJson<TileBatchWindowData>(jsons[0]);

        //textureList에 data의 prefabArray와 quaternionArray를 복사
        textureList = new List<(GameObject, Vector3Int, Vector3)>();
        for (int i = 0; i < data.prefabPathArray.Length; i++)
        {
            GameObject prefab = AssetDatabase.LoadAssetAtPath<GameObject>(data.prefabPathArray[i]);
            textureList.Add((prefab, data.quaternionArray[i], data.positionArray[i]));
        }

        for (var i = 0; i < data.texturePathArray.Length; i++)
        {
            var spritePath = data.texturePathArray[i];
            Texture2D texture2D = AssetDatabase.LoadAssetAtPath<Texture2D>(spritePath);
            Object[] objs = AssetDatabase.LoadAllAssetsAtPath(spritePath);
            //spriteNameArray에 저장된 이름과 같은 이름을 가진 sprite를 찾아서 spriteList에 추가
            foreach (var obj in objs)
            {
                if (obj is Sprite sprite && sprite.name == data.spriteNameArray[i])
                {
                    spriteList.Add(sprite);
                    break;
                }
            }
        }
    }
}
```

</div>
</details>

처음에는 단순히 gameObject와 Sprite를 저장하려고 시도했었다. 그러나 전혀 작동하지 않았다.

Sprite는 Sprite의 경로가 아닌, 자신이 속한 texture의 경로를 저장했다.

gameObject 역시 guid를 저장했다.

때문에 코드를 아래와 같이 수정하였다.

```csharp
// sprite의 경우에는 - Save -
// sprite가 속한 texturePathArray와 sprite의 이름을 저장한다.
data.texturePathArray = new String[spriteList.Count];
        for (int i = 0; i < spriteList.Count; i++)
        {
            if (spriteList[i] != null)
            {
                data.texturePathArray[i] = AssetDatabase.GetAssetPath(spriteList[i]);
                data.spriteNameArray[i] = spriteList[i].name;
            }
        }
...

// sprite의 경우에는 - Load -
// sprite가 속한 texturePathArray와 sprite의 이름을 통해 sprite를 찾아서 spriteList에 추가한다.
for (var i = 0; i < data.texturePathArray.Length; i++)
        {
            var spritePath = data.texturePathArray[i];
            Texture2D texture2D = AssetDatabase.LoadAssetAtPath<Texture2D>(spritePath);
            Object[] objs = AssetDatabase.LoadAllAssetsAtPath(spritePath);
            //spriteNameArray에 저장된 이름과 같은 이름을 가진 sprite를 찾아서 spriteList에 추가
            foreach (var obj in objs)
            {
                if (obj is Sprite sprite && sprite.name == data.spriteNameArray[i])
                {
                    spriteList.Add(sprite);
                    break;
                }
            }
        }
```

```csharp
// gameObject의 경우에는 - Save -
// gameObject의 경로를 저장한다.
data.prefabPathArray = new String[textureList.Count];
for (int i = 0; i < textureList.Count; i++)
{
    if (textureList[i].Item1 != null)
        data.prefabPathArray[i] = AssetDatabase.GetAssetPath(textureList[i].Item1);
    data.quaternionArray[i] = textureList[i].Item2;
    data.positionArray[i] = textureList[i].Item3;
}
...

// gameObject의 경우에는 - Load -
// gameObject의 경로를 통해 gameObject를 찾아서 textureList에 추가한다.
for (int i = 0; i < data.prefabPathArray.Length; i++)
{
    GameObject prefab = AssetDatabase.LoadAssetAtPath<GameObject>(data.prefabPathArray[i]);
    textureList.Add((prefab, data.quaternionArray[i], data.positionArray[i]));
}
```

---

## Editor Window의 화면 크기 가변 기능 설명

매치하려는 타일과 prefab의 수가 많아져서 화면 밖으로 나가는 일이 잦아서 스크롤 뷰를 추가하였다.
그러나 GUILayout이 아닌 GUIRect로 만든 rect들은 스크롤 뷰에 포함되지 않기 때문에 스크롤 기능이 동작하지 않기 때문에
GUILayout.Box("", GUILayout.Width(0), GUILayout.Height(scrollViewHeight));
를 통해 스크롤 뷰의 높이를 늘려주었다.

```csharp
for (int i = 0; i < textureList.Count; i++)
        {
            Rect prefabRect = new Rect(30, 150 + (i * 100),
                (EditorGUIUtility.currentViewWidth * (0.7f)) - 60,
                100);
            ...

for (int i = 0; i < spriteList.Count; i++)
        {
            Rect spriteRect = new Rect(EditorGUIUtility.currentViewWidth * 0.7f + 30, 150 + (i * 100),
                (EditorGUIUtility.currentViewWidth * 0.3f) - 60,
                100);
            ...
```
textureList와 spriteList의 너비는 Editor Window의 너비에 따라 가변적으로 변하도록 구현하였다.

---

```csharp
private void OnGUI()
{
    scrollPosition = EditorGUILayout.BeginScrollView(scrollPosition);

    ...

    //rect들은 guilayout의 scroll에 포함되지 않기 때문에 스크롤뷰가 짧게 나옴. 때문에 rect들만한 높이의 박스를 넣어서 스크롤 뷰가 길어지게 해야 함.
    float scrollViewHeight = 300 + (highestHeight * 100);
    GUILayout.Box("", GUILayout.Width(0), GUILayout.Height(scrollViewHeight));

    EditorGUILayout.EndScrollView();
}
```


