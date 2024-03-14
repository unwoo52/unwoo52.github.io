---
title: Game Data 및 Data사용자 구현
author: unwoo52
date: 2024-03-09 00:00:00 +09:00
categories: [Project, PrivateProject, Project2D3D, CodeDetail]
tags: [Unity, ScriptableObject, Project2D3D, Json, Save, Load]
---

# Game Data 및 Data사용자 구현

게임 내에 존재하는 여러 유형의 데이터 중 일부를 설명한다.

각 데이터마다 형태도 다르고 특성이나 불러오는 방식이 상이하다.

이렇게 상이한 내용들을 담당하는 다음 부분
* LoadData로 특정 Type의 데이터들을 받아온 뒤, 어떤 데이터가 필요한건지 검색하는 부분
* 데이터를 json으로 변환될 필드들([serializeble] 필드들)의 정의
을 GameDataManager가 아닌 데이터를 사용하는 각 클래스에서 구현하도록 위임하였다.

## PlayerData 및 PlayerData 사용자

### 플레이어 데이터
```csharp
[Serializable]
public class PlayerData
{
    public PlayerData(float positionX = 0f, float positionY = 0f, string locatedSceneName = "StartVillage House")
    {
        this.positionX = positionX;
        this.positionY = positionY;
        this.locatedSceneName = locatedSceneName;
    }

    [SerializeField] public float positionX;
    [SerializeField] public float positionY;
    [SerializeField] public string locatedSceneName;
}
```

### 플레이어 데이터 사용자

```csharp

## ScenarioData 및 ScenarioData 사용자

### 시나리오 데이터

```csharp
[Serializable]
public class FieldScenarioData : ScenarioData
{
    public FieldScenarioData(string eventName, bool isRead = false) : base(eventName, isRead)
    {
    }

    [SerializeField] private string upperTestData = "upperTest";
    public bool IsConditionSatisfied { get; private set; } = true;
}
```

### 시나리오 데이터 사용자

<details>
<summary> SceneScenarioEvent.cs 전체 코드 [접기/펼치기]</summary>
<div markdown="1">

```csharp
public class SceneScenarioEvent : MonoBehaviour
{
    public TextAsset scenarioAsset;

    private void Start()
    {
        // Get ScenarioData Array from SaveData
        FieldScenarioData fieldScenarioData = LoadGameData(scenarioAsset.name);

        if (fieldScenarioData.IsConditionSatisfied)
        {
            void OnDialogueEnd()
            {
                fieldScenarioData.isRead = true;
                SaveGameData(fieldScenarioData);
            }

            DialogueManager.Instance.StartDialogue(scenarioAsset, OnDialogueEnd).Forget();
        }
    }

    private FieldScenarioData LoadGameData(string eventName)
    {
        TotalGameData totalGameData = GameDataManager.Instance.TotalGameData;
        FieldScenarioData[] loadData = totalGameData.LoadData<FieldScenarioData>();

        FieldScenarioData fieldScenarioData = loadData.FirstOrDefault(x => x.eventName == eventName);

        // If the ScenarioData does not exist, create a new ScenarioData
        if (fieldScenarioData == null)
        {
            fieldScenarioData = new FieldScenarioData(eventName);
        }

        return fieldScenarioData;
    }

    private void SaveGameData(FieldScenarioData fieldScenarioData)
    {
        // Get T Data Array from TotalGameData
        TotalGameData totalGameData = GameDataManager.Instance.TotalGameData;
        FieldScenarioData[] loadDataArray = totalGameData.LoadData<FieldScenarioData>();

        FieldScenarioData loadData = loadDataArray.FirstOrDefault(x => x.eventName == fieldScenarioData.eventName);

        // Check if the ScenarioData is already in the GameData
        // If the ScenarioData is already in the GameData, update the data
        if (loadData != null)
        {
            loadData.isRead = true;
        }
        else // If the ScenarioData does not exist in the GameData, add the data
        {
            totalGameData.SaveData(fieldScenarioData);
        }
    }
}
```

</div>
</details>

![image](https://github.com/unwoo52/unwoo52.github.io/assets/73688472/7968ed3e-5784-4388-9e5e-e44717af8a6a)

이 오브젝트는 씬이 시작될 때 ScenarioData를 불러와서 시나리오가 읽혔는지 아닌지를 판단해서 대화를 시작한다.

사진과 같이 씬에 있는 오브젝트에 컴포넌트 추가로 생성한 뒤, 재생할 시나리오 TextAsset을 바인딩해서 사용한다.

만약 한번 읽은 시나리오라면 재생하지 않는다.

```csharp
private void Start()
{
    // Get ScenarioData Array from SaveData
    FieldScenarioData fieldScenarioData = LoadGameData(scenarioAsset.name);

    if (fieldScenarioData.IsConditionSatisfied)
    {
        void OnDialogueEnd()
        {
            fieldScenarioData.isRead = true;
            SaveGameData(fieldScenarioData);
        }

        DialogueManager.Instance.StartDialogue(scenarioAsset, OnDialogueEnd).Forget();
    }
}
```

씬에 들어오면 LoadGameData를 통해 ScenarioData를 불러와서 ScenarioData가 읽혔는지 아닌지를 판단해서 대화를 시작한다.

```csharp
DialogueManager.Instance.StartDialogue(scenarioAsset, OnDialogueEnd).Forget();
```

대화를 시작할 수 있는 Singleton DialogueManager를 통해 대화를 시작할 수 있으며, 대화가 끝나면 ScenarioData의 isRead를 true로 바꾸고 SaveGameData를 통해 저장한다.

<br>

---


```csharp
private FieldScenarioData LoadGameData(string eventName)
{
    TotalGameData totalGameData = GameDataManager.Instance.TotalGameData;
    FieldScenarioData[] loadData = totalGameData.LoadData<FieldScenarioData>();

    FieldScenarioData fieldScenarioData = loadData.FirstOrDefault(x => x.eventName == eventName);

    // If the ScenarioData does not exist, create a new ScenarioData
    if (fieldScenarioData == null)
    {
        fieldScenarioData = new FieldScenarioData(eventName);
    }

    return fieldScenarioData;
}
```

LoadGameData는 TotalGameData에서 FieldScenarioData를 불러오는 함수이다.

FieldScenarioData를 모두 불러온 뒤, eventName과 일치하는 FieldScenarioData가 있는지 확인하고 없다면 새로운 FieldScenarioData를 생성하고, 있다면 해당 FieldScenarioData를 반환한다.

이 씬에 있는 이 SceneScenarioEvent가 갖고 있는 시나리오 이벤트를 찾는 코드는 아래와 같은데

```csharp
FieldScenarioData fieldScenarioData = loadData.FirstOrDefault(x => x.eventName == eventName);
```

이렇듯, 특정 데이터를 찾는 코드는 각 데이터를 사용하는 클래스(이 경우에는 SceneScenarioEvent)에서 구현하도록 하였다.

---

<br>

```csharp
private void SaveGameData(FieldScenarioData fieldScenarioData)
{
    // Get T Data Array from TotalGameData
    TotalGameData totalGameData = GameDataManager.Instance.TotalGameData;
    FieldScenarioData[] loadDataArray = totalGameData.LoadData<FieldScenarioData>();

    FieldScenarioData loadData = loadDataArray.FirstOrDefault(x => x.eventName == fieldScenarioData.eventName);

    // Check if the ScenarioData is already in the GameData
    // If the ScenarioData is already in the GameData, update the data
    if (loadData != null)
    {
        loadData.isRead = true;
    }
    else // If the ScenarioData does not exist in the GameData, add the data
    {
        totalGameData.SaveData(fieldScenarioData);
    }
}
```

SaveGameData는 TotalGameData에 FieldScenarioData를 저장하는 함수이다.

FieldScenarioData를 저장하려고 할 때, 이미 해당 FieldScenarioData가 TotalGameData에 존재하는지 확인한다.
만약 이미 있다면 해당 FieldScenarioData의 isRead를 true로 바꾸고, 없다면 새로운 FieldScenarioData를 추가한다.
