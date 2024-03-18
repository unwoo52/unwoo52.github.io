---
title: Game Data Manager 구현
author: unwoo52
date: 2024-03-08 00:00:00 +09:00
categories: [Project, JRPG-FPS, CodeDetail]
tags: [Unity, ScriptableObject, Project2D3D, Json, Save, Load]
---

# Game Data Manager

게임 데이터를 저장하고 불러오는 기능을 담당하는 클래스이다.

GameTotalData라는 인스턴스를 소유하며, 이 인스턴스에 게임 데이터를 저장하고 불러온다.

게임 시작 화면에서 Load 화면을 통해 기존에 저장한 GameTotalData를 불러오거나, 새로운 GameTotalData를 생성해서 게임을 시작할 수 있다.

<details>
<summary> GameDataManager.cs 전체 코드 [접기/펼치기]</summary>
<div markdown="1">

```csharp
using System;
using System.IO;
using System.Linq;
using Game_Data;
using Game_Data.Player;
using Player;
using UnityEditor;
using UnityEngine;

namespace Manager
{
    public class GameDataManager : MonoBehaviour
    {
        public static GameDataManager Instance { get; private set; }

        private TotalGameData totalGameData;

        public TotalGameData TotalGameData
        {
            get
            {
                // If the gameData is null, create a new GameData
                if (totalGameData == null)
                {
                    totalGameData = new TotalGameData();
                    totalGameData.OnDataLoaded.AddListener(() => Debug.Log("Data Loaded"));
                    totalGameData.OnDataSaved.AddListener(Save);
                }

                return totalGameData;
            }
            set => totalGameData = value;
        }

        private void Awake()
        {
            if (Instance == null)
            {
                Instance = this;
                DontDestroyOnLoad(gameObject);
            }
            else
            {
                Destroy(gameObject);
            }
        }

        public void Save()
        {
            Save(GameManager.Instance.GameName);
        }

        public void Save(string fileName)
        {
            try
            {
                FileDataHandler<TotalGameData> fileDataHandler =
                    new FileDataHandler<TotalGameData>(Application.persistentDataPath, fileName + ".save");
                fileDataHandler.Save(TotalGameData);
            }
            catch (Exception ex)
            {
                Debug.LogError($"An error occurred while saving: {ex.Message}");
            }
        }

        private void OnApplicationQuit()
        {
            TotalGameData.InvokeOnDataLoaded();
        }

        public void Load(string gameName)
        {
            try
            {
                FileDataHandler<TotalGameData> fileDataHandler =
                    new FileDataHandler<TotalGameData>(Application.persistentDataPath, gameName + ".save");
                TotalGameData data = fileDataHandler.Load();

                if (data != null)
                {
                    TotalGameData = data;
                    TotalGameData.OnDataSaved.AddListener(Save); // Auto Save
                }
            }
            catch (Exception ex)
            {
                Debug.LogError($"An error occurred while loading: {ex.Message}");
            }
        }
    }
}
```

</div>
</details>

# GameTotalData

게임 데이터를 담는 클래스이며, GameDataManager에 의해 생성되고 관리된다.

게임이 켜져있을 때 항상 존재하는게 아니며, 처음 화면에서 메뉴를 통해 게임을 불러오거나 시작할 때 생성되거나 로드되어 존재한다.

<details>
<summary> GameTotalData.cs 전체 코드 [접기/펼치기]</summary>
<div markdown="1">

```csharp
using System;
using System.Linq;
using UnityEngine;
using UnityEngine.Events;

namespace Game_Data
{
    [Serializable]
    public class TotalGameData
    {
        [SerializeField] private string[] data = new string[0];

        public UnityEvent OnDataLoaded = new();
        public UnityEvent OnDataSaved = new();

        public void SaveData<T>(T data) where T : class
        {
            Array.Resize(ref this.data, this.data.Length + 1);

            // typeName을 json 앞에 붙여서 저장. 다시 데이터를 찾을 때 Type을 구분할 수 있기 위해서
            string json = typeof(T) + "-";

            json += JsonUtility.ToJson(data, true);

            this.data[this.data.Length - 1] = json;
            //Invoke onDataSaved
            OnDataSaved?.Invoke();
        }

        public T[] LoadData<T>() where T : class
        {
            OnDataLoaded?.Invoke();
            string typeName = typeof(T).ToString();

            // data에서 -를 만나기 전까지만 읽고 typeName과 일치하는 요소들을 찾기
            string[] jsons = data.Where(x => x.Substring(0, x.IndexOf("-")) == typeName).ToArray();

            // typeName을 제외한 나머지 부분을 읽어서 T로 변환
            return jsons.Select(x => JsonUtility.FromJson<T>(x.Substring(x.IndexOf("-") + 1))).ToArray();
        }

        public void InvokeOnDataLoaded()
        {
            OnDataLoaded?.Invoke();
        }
    }
}
```

</div>
</details>

## 코드 설명

#### Save

```csharp
public void SaveData<T>(T data) where T : class
{
    Array.Resize(ref this.data, this.data.Length + 1);

    // typeName을 json 앞에 붙여서 저장. 다시 데이터를 찾을 때 Type을 구분할 수 있기 위해서
    string json = typeof(T) + "-";

    json += JsonUtility.ToJson(data, true);

    this.data[this.data.Length - 1] = json;
    //Invoke onDataSaved
    OnDataSaved?.Invoke();
}
```

받은 data를 json으로 변한한 뒤, json 앞에 getType으로 얻은 타입명을 string으로 붙혀둔다.

> 이 경우에 같은 이름의 클래스명 때문에 다시 UnBoxing(복호화)할 때 문제가 발생하는지를 검토해보았다.
> 다행히도 namespace까지 포함한 이름으로 'namespaceName.className'으로 저장되기에 같은 이름의 클래스라도 구분이 가능하다.

때문에 저장되는 형태는 아래와 같다.

> Scenario.SceneScenarioEvent-{"name":"Scene1","event":[{"name":"Event1","event":[{"name":"Event1-1","event":[]}]}]}

풀어 읽어보면 ' type명 - json열' 형태로 -로 구분되어 저장되어 있다.

#### Load

```csharp
public T[] LoadData<T>() where T : class
{
    OnDataLoaded?.Invoke();
    string typeName = typeof(T).ToString();

    // data에서 -를 만나기 전까지만 읽고 typeName과 일치하는 요소들을 찾기
    string[] jsons = data.Where(x => x.Substring(0, x.IndexOf("-")) == typeName).ToArray();

    // typeName을 제외한 나머지 부분을 읽어서 T로 변환
    return jsons.Select(x => JsonUtility.FromJson<T>(x.Substring(x.IndexOf("-") + 1))).ToArray();
}
```

저장된 데이터를 불러올 때는 typeName을 이용하여 해당 타입의 데이터만을 불러온다.

불러온 데이터는 json으로 변환된 데이터를 다시 T로 변환하여 반환한다.

복호화 할 때 string의 앞 부분만 읽게 하는 등 약간의 기능개선을 시도하였지만, 이 부분은 더 개선할 여지가 있다.

다만, 이 코드는 빠르개 개발해서 여러 게임에 투입될 수 있도록 범용성을 최대로 높히는 추상화 단계로의 구현이 목표였기에
근본적으로 더 빠르고 안전한 SaveLoad를 구현하기 위해선 아예 다른 코드를 사용해야 한다.

## 코드에 담긴 의도

이 GameTotalData는 속도나 안정성 면에서 비효율적이다.

하지만, 이 코드는 여러 게임에 적용될 수 있는 범용성을 높히기 위해 구현되었다.

GameTotalData는 저장되어야 하는 대상 data들의 타입을 모르게 할 수 있도록 구현되어 있다.

<br>

처음엔 아래와 같은 형태로 구현을 생각할 수 있었다.

```csharp
public class TotalGameData
{
    //scenario data
    [SerializeField] private ScenarioData[] data;
    //player data
    [SerializeField] private PlayerData[] data;
    //item data
    [SerializeField] private ItemData[] data;
    //enemy data
    [SerializeField] private EnemyData[] data;

    ...

}
```

하지만 이러한 방법은 새로운 Save Load 대상 타입이 추가될 때마다 TotalGameData 클래스를 수정해야 하며, 이는 또 개방폐쇄 원칙에 어긋나는 구조이다.

때문에 새로운 기능을 추가하려 해도 기존의 코드에 변경점이 없어도 되도록 만들기 위해 아래와 같은 방법을 선택하였다.

```csharp
public class TotalGameData
{
    [SerializeField] private string[] data = new string[0];

    ...

}
```

그리고 이 string 배열에 각 저장 데이터 type명과 데이터 json을 저장하게 구현하였다.

Save Load 함수도 제네릭으로 구현하여, 어떤 타입의 데이터라도 저장하고 불러올 수 있도록 하였다.

## 개선 가능성

* 데이터 저장 예시

> Scenario.SceneScenarioEvent-{"name":"Scene1","event":[{"name":"Event1","event":[{"name":"Event1-1","event":[]}]}]}

각 데이터의 type은 string으로 변환되고, 또 단순히 -구분자를 두고 json 앞에 붙여두기 때문에, 이 부분을 더 안전하게 구현할 수 있도록 개선할 수 있을 것이다.

가장 좋은 것은 이를 dictionary타입으로 저장하고, GameTotalData 인스턴스가 생성되거나 저장되는 때에 dictionary를 json으로 변환하거나 푸는 함수를 구현하여서 사용하는 것 이다.

예를들어 GameTotalData에서 ScenarioData를 불러올 때에는 기존의 방법은 모든 string들의 "-" 앞 부분을 추출해서 type이 무엇인지 읽어야 해서
높은 비용이 들었다.

하지만 dictionary(Type, data)로 해두면 빠르게 해당 데이터들 collection만 모아서 Load함수로 반환할 수 있다.
