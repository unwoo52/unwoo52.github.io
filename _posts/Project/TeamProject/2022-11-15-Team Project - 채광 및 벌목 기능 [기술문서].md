---
title: Team Project - 채광 및 벌목 기능 [기술문서]
author: unwoo52
date: 2022-11-15 00:00:00 +09:00
categories: [Unity, TeamProj]
tags: [Unity, TeamProj, Team, Mining, Logging, Resource, TechnicalDocument, Docs, Document]
---

# <span style="color:yellow">기술 문서 개요</span>

### 프로젝트, 기술문서, 스토리보드 링크

> [팀프로젝트 문서](https://unwoo52.github.io/posts/Team-Project-About/)는 프로젝트의 전체 내용과 개요 등에 대해 적혀있습니다.
>
> [youtube 채광 및 벌목 시연 영상](https://youtu.be/XYon_3MIK5E?t=92)에서 버프 작동을 유튜브로 볼 수 있습니다.
>
> <span style="color:yellow">기술 문서는 전체 코드와 코드들에 대한 설명이 적혀있습니다.</span>
> 
> [스토리보드]()에는 해당 기능이 어떻게 게임에서 실행되는지 시간순서대로 읽기 쉽게 스토리로 작성하였습니다.


## 채광 및 벌목 구현 개요

게임 내의 자원인 나무와 광석을 파밍할 수 있는 시스템을 구현하였다.

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingSystem/000.gif)

나무를 벌목하면 통나무와 잎사귀 오브젝트 아이템들이 생성되어 떨어진다.

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingSystem/000-1.gif)

바위를 채굴하면 돌맹이 아이템이 생성되어 떨어지고, 일정 체력 이하로 떨어질 때 마다 바위의 크기가 작아진다.


<br>

----------


# 채광 및 벌목 관련 오브젝트들

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingSystem/001.png)

> 바위 오브젝트. 채굴하면 바위 아이템을 떨어트리며, 채굴하면 할수록 작아진다.

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingSystem/001-1.png)

> 바위 오브젝트에서 드롭되는 아이템.

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingSystem/002.png)

> 나무 오브젝트. 체력을 모두 소모시키면 쓰러지면서 나무 통과 잎사귀 아이템들을 일정량 드롭한다.

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingSystem/002-1.png)

> 나무 오브젝트를 벌목하면 바닥에 남는 오브젝트.

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingSystem/002-2.png)

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingSystem/002-3.png)

![imagename](/assets/image/Project/TeamProject/MiningAndLoggingSystem/002-4.png)


----------
