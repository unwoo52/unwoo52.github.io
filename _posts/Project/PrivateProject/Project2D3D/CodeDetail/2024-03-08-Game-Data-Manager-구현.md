---
title: Game Data Manager 구현
author: unwoo52
date: 2024-03-08 00:00:00 +09:00
categories: [Project, PrivateProject, Project2D3D, CodeDetail]
tags: [Unity, ScriptableObject, Project2D3D, Json, Save, Load]
---

# Game Data Manager 구현

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

