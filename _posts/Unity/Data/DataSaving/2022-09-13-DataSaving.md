---
title: Data Saving
author: unwoo52
date: 2022-11-13 00:00:00 +09:00
categories: [Unity, UI]
tags: [Unity, UI]
---

## Data Saving

![imagename](/assets/image/Unity/Data/DataSaving/001.png)

<details>
<summary>StudyFileManager</summary>
<div markdown="1">

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using System.IO;
using System.Runtime.Serialization.Formatters.Binary;

public class StudyFileManager : MonoBehaviour
{
    public static StudyFileManager instance = null;
    BinaryFormatter binaryFormatter = null;
    private void Awake()
    {
        instance = this;
        binaryFormatter= new BinaryFormatter();
    }

    public void SaveText(string filePath, string content)
    {
        File.WriteAllText(filePath, content);
    }

    public string LoadText(string filePath)
    {
        return File.ReadAllText(filePath);
    }

    public void SaveBinary<T>(string filePath, T data)
    {
        using (FileStream fileStream = File.Create(filePath))
        {
            binaryFormatter.Serialize(fileStream, data);
            fileStream.Close();
        }
        
    }

    public T LoadBinary<T>(string filePath)
    {
        T data = default;
        if (File.Exists(filePath))
        {
            using(FileStream fileStream = File.Open(filePath, FileMode.Open))
            {
                data = (T)binaryFormatter.Deserialize(fileStream);
                fileStream.Close();
            }
        }
        else
        {
            Debug.LogError(filePath + "파일이 존재하지 않습니다.");
        }
        return data;
    }
}
```


</div>
</details>


<details>
<summary>StudyFileData</summary>
<div markdown="1">

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[System.Serializable]
public struct StudyPlayerData
{
    public string Nick;
    public int Gold;
    public int Exp;
}

public class StudyFileData : MonoBehaviour
{
    public TMPro.TMP_Text Nick = null;
    public TMPro.TMP_Text Gold = null;
    public TMPro.TMP_Text Exp = null;
    bool TestBool = false;
    void Start()
    {
        // StudyFileManager.instance.SaveText(Application.dataPath + "\\Test.txt", "이건 테스트 입니다.");
        //myLabel.text = StudyFileManager.instance.LoadText(Application.dataPath + "\\Test.txt");

        if(TestBool)
        {
            StudyPlayerData data = new StudyPlayerData();
            data.Nick = "Nickname";
            data.Gold = 1000;
            data.Exp = 10;

            StudyFileManager.instance.SaveBinary<StudyPlayerData>(Application.dataPath + @"\TestPlayerData.playerdatafile", data);
        }

        StudyPlayerData studyPlayerData = StudyFileManager.instance.LoadBinary<StudyPlayerData>(Application.dataPath + @"\TestPlayerData.playerdatafile");

        Nick.text = studyPlayerData.Nick;
        Gold.text = studyPlayerData.Gold.ToString();
        Exp.text = studyPlayerData.Exp.ToString();




    }

    // Update is called once per frame
    void Update()
    {

    }
}
```

</div>
</details>
