---
title: Scriptable Onbject
author: unwoo52
date: 2022-09-07 00:00:00 +09:00
categories: [Unity, Data]
tags: [Unity, Data, Scriptable, ScriptableObject, Item, ItemData]
---

## Scriptable Object

```cs
[CreateAssetMenu(fileName = "ItemData", menuName = "ScriptableObjects/ItemData", order = 1)]
public class StudyItemData : ScriptableObject
{
    public string Name;
    public int Price;
    public int Value;
}
```
