---
title: Scriptable Onbject
author: unwoo52
date: 2022-11-13 00:00:00 +09:00
categories: [Unity, UI]
tags: [Unity, UI]
---

## Scriptable Onbject

[CreateAssetMenu(fileName = "ItemData", menuName = "ScriptableObjects/ItemData", order = 1)]
public class StudyItemData : ScriptableObject
{
    public string Name;
    public int Price;
    public int Value;
}
