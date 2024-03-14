---
title: File Data Handler 구현
author: unwoo52
date: 2024-03-08 00:00:00 +09:00
categories: [Project, PrivateProject, Project2D3D, CodeDetail]
tags: [Unity, ScriptableObject, Project2D3D, FileHandler, Json, Save, Load]
---

# File Data Handler 구현

Game Data Manager와 File Data Handler를 독립적으로 만듦으로써 게임 데이터를 저장하고 불러오는 기능이 변경될 경우도 대비할 수 있다.

이를테면 Cloud 저장으로 변경되거나, 다른 게임에서 Game Data Manager을 새로 만들 때 이 File Data Handler를 그대로 사용할 수 있다.

<details>
<summary> FileDataHandler.cs 전체 코드 [접기/펼치기]</summary>
<div markdown="1">

```csharp
using System;
using System.IO;
using UnityEngine;

namespace Game_Data
{
    /// <summary>
    /// Json을 받아서 파일로 저장하거나, 파일을 읽어서 Json으로 변환하는 클래스
    /// </summary>
    public class FileDataHandler<T> where T : class
    {
        private string dataDirPath = "";
        private string dataFileName = "";
        private bool useEncryption = false;

        private const string EncryptionKey = "MyEncryptionKey";

        public FileDataHandler(string dataDirPath, string dataFileName, bool useEncryption = true)
        {
            this.dataDirPath = dataDirPath;
            this.dataFileName = dataFileName;
            this.useEncryption = useEncryption;
        }

        public void Save(T data)
        {
            string fullPath = Path.Combine(dataDirPath, dataFileName);
            try
            {
                // 파일이 없다면 디렉토리와 파일을 생성
                Directory.CreateDirectory(Path.GetDirectoryName(fullPath));

                // 인자로 받은 data를 Json으로 변환
                string dataToStore = JsonUtility.ToJson(data, true);
                if(useEncryption)
                    dataToStore = Encrypt(dataToStore);

                // 파일에 Json을 쓰기
                using (FileStream stream = new FileStream(fullPath, FileMode.Create))
                {
                    using (StreamWriter writer = new StreamWriter(stream))
                    {
                        writer.Write(dataToStore);
                    }
                }
            }
            catch (Exception e)
            {
                Debug.LogError($"Error occured while saving file: {fullPath} \n {e}");
            }
        }

        public T Load()
        {
            string fullPath = Path.Combine(dataDirPath, dataFileName);

            T data = null;

            if (File.Exists(fullPath))
            {
                try
                {
                    string dataAsJson = "";
                    using (FileStream stream = new FileStream(fullPath, FileMode.Open))
                    {
                        using (StreamReader reader = new StreamReader(stream))
                        {
                            dataAsJson = reader.ReadToEnd();
                        }
                    }

                    if(useEncryption)
                        dataAsJson = Encrypt(dataAsJson);

                    data = JsonUtility.FromJson<T>(dataAsJson);
                }
                catch (Exception e)
                {
                    Debug.LogError($"Error occured while loading file: {fullPath} \n {e}");
                }
            }

            return data;
        }

        private string Encrypt(string data)
        {
            string result = string.Empty;
            for (int i = 0; i < data.Length; i++)
            {
                result += (char)(data[i] ^ EncryptionKey[i % EncryptionKey.Length]);
            }
            return result;
        }
    }
}

```

</div>
</details>

## 코드 설명

### Save and Load

```csharp
public void Save(T data)
{
    string fullPath = Path.Combine(dataDirPath, dataFileName);
    try
    {
        // 파일이 없다면 디렉토리와 파일을 생성
        Directory.CreateDirectory(Path.GetDirectoryName(fullPath));

        // 인자로 받은 data를 Json으로 변환
        string dataToStore = JsonUtility.ToJson(data, true);
        if(useEncryption)
            dataToStore = Encrypt(dataToStore);

        // 파일에 Json을 쓰기
        using (FileStream stream = new FileStream(fullPath, FileMode.Create))
        {
            using (StreamWriter writer = new StreamWriter(stream))
            {
                writer.Write(dataToStore);
            }
        }
    }
    catch (Exception e)
    {
        Debug.LogError($"Error occured while saving file: {fullPath} \n {e}");
    }
}

public T Load()
{
    string fullPath = Path.Combine(dataDirPath, dataFileName);

    T data = null;

    if (File.Exists(fullPath))
    {
        try
        {
            string dataAsJson = "";
            using (FileStream stream = new FileStream(fullPath, FileMode.Open))
            {
                using (StreamReader reader = new StreamReader(stream))
                {
                    dataAsJson = reader.ReadToEnd();
                }
            }

            if(useEncryption)
                dataAsJson = Encrypt(dataAsJson);

            data = JsonUtility.FromJson<T>(dataAsJson);
        }
        catch (Exception e)
        {
            Debug.LogError($"Error occured while loading file: {fullPath} \n {e}");
        }
    }

    return data;
}
```

Save와 Load 함수는 각각 인자로 받은 data를 Json으로 변환하여 파일에 쓰고, 파일을 읽어서 Json으로 변환한 후 반환한다.

### Encrypt

```csharp
private string Encrypt(string data)
{
    string result = string.Empty;
    for (int i = 0; i < data.Length; i++)
    {
        result += (char)(data[i] ^ EncryptionKey[i % EncryptionKey.Length]);
    }
    return result;
}
```

Encrypt 함수는 받은 data를 XOR 연산을 통해 암호화한다.

Json은 읽기 쉽고 간단하게 수정될 수 있기 때문에, 이러한 암호화를 통해 게임의 데이터가 노출되거나 수정되는 것을 방지해야 한다.
