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

