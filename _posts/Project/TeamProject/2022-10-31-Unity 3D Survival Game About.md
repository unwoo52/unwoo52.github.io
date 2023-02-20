---
title: Unity 3D Survival Game About
author: unwoo52
date: 2022-10-31 00:00:00 +09:00
categories: [Unity, TeamProj]
tags: [Unity, TeamProj, PPT, Github, SurvivalGame,About]
---

# 개요

아텐츠게임아카데미에서 팀프로젝트 활동을 통해 진행한 '생존게임' 프로젝트의 개요를 설명합니다.

**팀프로젝트 내에 제가 구현한 기능들만에 대해서만 기술 문서와 스토리 보드를 작성하였습니다. 개인 포트폴리오를 위한 포스팅이므로 다른 팀원이 개발한 내용에 대한 코드는 포함시키지 않았습니다.**

# 내가 구현한 기능

## 건물 짓기

![imagename](/assets/image/Project/TeamProject/SurvivalProjectAboud/004.gif)

> 건물 아이템을 사용하여 건물을 지을 수 있는 기능을 구현하였습니다. 건물 아이템을 사용하면 에임 위치에 건물의 실루엣이 표시되며, 원하는 위치에 에임을 위치시키고 클릭하면 건물이 건설됩니다.
> 
> 건물 짓기 구현에서는 지으려는 위치의 지형에 따라 건물을 지을 수 있는지 없는지를 시작적으로 가이드 할 수 있게 구현하였습니다.

## 나무 벌목 및 바위 채광

![imagename](/assets/image/Project/TeamProject/SurvivalProjectAboud/001.gif)

![imagename](/assets/image/Project/TeamProject/SurvivalProjectAboud/001-1.gif)

> 나무와 바위의 벌목 및 채광 기능을 구현하였습니다. 자원 오브젝트가 조건에 따라 변하고, 입은 데미지만큼 아이템 오브젝트를 드랍하도록 구현하였습니다.

## 버프 시스템

![imagename](/assets/image/Project/TeamProject/SurvivalProjectAboud/000.gif)

> 다양한 종류의 버프를 하나의 Buff Manager(팩토리메소드)에서 생성하도록 구현하였습니다. 버프를 테스트 할 수 있는
>
> 이 외에 플래그 연산을 통해, 지형 버프를 인식하는 비용(플레이어가 움직이는 매 프레임마다 접촉한 지형의 정보를 연산해야 함)을 줄이기 위해 노력하였습니다.
