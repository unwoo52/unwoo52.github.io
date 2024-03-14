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

## ScenarioData 및 ScenarioData 사용자

### ScenarioData 사용자인 SceneScenarioEvent

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

[TODO ScenarioData 사용자인 SceneScenarioEvent 모노비헤비어 사진]

