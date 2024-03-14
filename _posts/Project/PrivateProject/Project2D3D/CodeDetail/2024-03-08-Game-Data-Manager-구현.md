---
title: Game Data Manager 구현
author: unwoo52
date: 2024-03-08 00:00:00 +09:00
categories: [Project, PrivateProject, Project2D3D, CodeDetail]
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

        public void DebugData()
        {
            foreach (var data in data)
            {
                Debug.Log(data);
            }
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

