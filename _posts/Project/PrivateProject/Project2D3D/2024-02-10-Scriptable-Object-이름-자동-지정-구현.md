---
title: Scriptable Object 이름 자동 저장 기능 구현
author: unwoo52
date: 2024-02-10 00:00:00 +09:00
categories: [Unity]
tags: [Unity, ScriptableObject, AssetPostprocessor, EditorUtility, INameDesignator, Automatize]
---

# Scriptable Object 이름 자동 저장 기능 구현

Scriptable Object 파일을 생성하면, 이름도 자동으로 정의되는 기능을 만들어보려고 한다.

![image](https://github.com/unwoo52/unwoo52.github.io/assets/73688472/f94a4a9a-0656-4bb2-a8bc-2896fcff65cf)

![image](https://github.com/unwoo52/unwoo52.github.io/assets/73688472/f71d333d-eaef-4e4b-ae1a-d9bb59ceddc8)


## 이름 변경 시 자동으로 name field의 값을 변경하기

이 기능은 OnValidate() 콜백을 이용해 쉽게 구현할 수 있다.

```cs
public class MonsterData_SO : ScriptableObject
{
    public string monsterName;
    public Sprite monsterSprite;
    public Animator animator;

    void OnValidate()
    {
        monsterName = this.name;
    }
```

아직은 파일을 생성함과 동시에 이름을 적을 땐 이름이 자동으로 변경되지 않는다.

## 생성 시 이름 자동으로 정의하기

코파일럿에게 이 기능을 구현해달라고 요청하여서 아래 코드를 받았다.

```cs
using UnityEditor;

namespace Tool.AssetsPostprocessor
{
    public class ScriptableObjectNameDefiner : AssetPostprocessor
    {
        private static void OnPostprocessAllAssets(string[] importedAssets, string[] deletedAssets,
            string[] movedAssets, string[] movedFromAssetPaths)
        {
            foreach (string str in importedAssets)
            {
                string fileName = System.IO.Path.GetFileNameWithoutExtension(str);
                MonsterData_SO data = AssetDatabase.LoadAssetAtPath<MonsterData_SO>(str);

                if (data != null)
                {
                    data.DesignName(fileName);
                    EditorUtility.SetDirty(data);
                }
            }
        }
    }
}
```

AssetPostprocessor의 OnPostprocessAllAssets() 콜백을 이용해 import된 파일으로부터 MonsterData_SO타입을 얻을 수 있다면 바로 이름을 적용하는 코드가 작성되었다.

하지만 이 코드는 MonsterData_SO 외에 새로운 type이 등장할 때 마다 ScriptableObjectNameDefiner가 수정되어야 하기 때문에 개방 폐쇄 원칙에 어긋난다.

이름을 자동으로 지정하기 위해 사용할 INameDesignator라는 인터페이스를 만들고, 위 코드에서 MonsterData_SO를 직접 명시하는것을 피하도록 하려고 한다.
 또한, OnPostprocessAllAssets함수의 매개변수인 importedAssets에서 이 인터페이스를 추출한 뒤 이름을 전달하는 식으로 개선하였다.

이를 통해 이름을 지정하는 행위는 OnPostprocessAllAssets에선 추상적으로 제시되고, 구체적인 행위는 INameDesignator를 구현한 클래스에서 이루어지도록 하였다.

## 결과물

```cs
using System;
using UnityEditor;
using UnityEngine;
using Object = UnityEngine.Object;

namespace Tool.AssetsPostprocessor
{
    public interface INameDesignator
    {
        void DesignName(string name);
    }

    public class ScriptableObjectNameDefiner : AssetPostprocessor
    {
        private static void OnPostprocessAllAssets(string[] importedAssets, string[] deletedAssets,
            string[] movedAssets, string[] movedFromAssetPaths)
        {
            foreach (string str in importedAssets)
            {
                string fileName = System.IO.Path.GetFileNameWithoutExtension(str);
                Type assetType = AssetDatabase.GetMainAssetTypeAtPath(str);

                if (assetType == null)
                {
                    Debug.LogWarning($"No main asset type at path: {str}");
                    continue;
                }

                Object asset = AssetDatabase.LoadAssetAtPath(str, assetType);

                if (asset == null)
                {
                    Debug.LogWarning($"Failed to load asset at path: {str}");
                    continue;
                }

                if (asset is INameDesignator nameDesignator)
                {
                    try
                    {
                        nameDesignator.DesignName(fileName);
                        EditorUtility.SetDirty(asset);
                    }
                    catch (Exception e)
                    {
                        Debug.LogError($"Failed to designate name for asset at path: {str}. Exception: {e}");
                    }
                }
            }
        }
    }
}
```
