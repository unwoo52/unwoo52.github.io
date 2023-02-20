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

# 구현한 기능

## 건물 짓기

![imagename](/assets/image/Project/TeamProject/SurvivalProjectAboud/004.gif)

> 건물 아이템을 사용하여 건물을 지을 수 있는 기능을 구현하였습니다. 건물 아이템을 사용하면 에임 위치에 건물의 실루엣이 표시되며, 원하는 위치에 에임을 위치시키고 클릭하면 건물이 건설됩니다.
> 
> 건물 짓기 구현에서는 지으려는 위치의 지형에 따라 건물을 지을 수 있는지 없는지를 시작적으로 가이드 할 수 있게 구현하였습니다.

- [기술 문서](https://unwoo52.github.io/posts/Team-Project-%EA%B1%B4%EB%AC%BC-%EC%95%84%EC%9D%B4%ED%85%9C-%EC%84%A4%EC%B9%98-%EA%B8%B0%EB%8A%A5-%EA%B8%B0%EC%88%A0-%EB%AC%B8%EC%84%9C/)는 구현된 기능에 대해 시간순서대로 작동하는 코드와 인게임 사진을 나열하여 어떻게 구현되었는지를 설명합니다.

- [오브젝트와 전체코드 설명](https://unwoo52.github.io/posts/Team-Project-Buff-%EA%B8%B0%EB%8A%A5-%EC%98%A4%EB%B8%8C%EC%A0%9D%ED%8A%B8%EC%99%80-%EC%A0%84%EC%B2%B4%EC%BD%94%EB%93%9C-%EC%84%A4%EB%AA%85/](https://unwoo52.github.io/posts/Team-Project-%EA%B1%B4%EB%AC%BC-%EC%95%84%EC%9D%B4%ED%85%9C-%EC%84%A4%EC%B9%98-%EA%B8%B0%EB%8A%A5-%EC%98%A4%EB%B8%8C%EC%A0%9D%ED%8A%B8%EC%99%80-%EC%A0%84%EC%B2%B4%EC%BD%94%EB%93%9C-%EC%84%A4%EB%AA%85/)는 이 기능과 관련된 오브젝트들과, 오브젝트에 있는 스크립트의 전체 코드를 설명합니다.

기술 문서를 읽으시면 기능 구현에 대해 알 수 있습니다. 기술 문서의 내용이나 글이 미흡해 기능의 내용을 잘 파악하지 못했다면, 선택적으로 오브젝트와 전체코드 설명을 추가로 참고하며 읽으실 수 있습니다.

## 나무 벌목 및 바위 채광

![imagename](/assets/image/Project/TeamProject/SurvivalProjectAboud/001.gif)

![imagename](/assets/image/Project/TeamProject/SurvivalProjectAboud/001-1.gif)

> 나무와 바위의 벌목 및 채광 기능을 구현하였습니다. 자원 오브젝트가 조건에 따라 변하고, 입은 데미지만큼 아이템 오브젝트를 드랍하도록 구현하였습니다.

- [기술 문서](https://unwoo52.github.io/posts/Team-Project-%EC%B1%84%EA%B4%91-%EB%B0%8F-%EB%B2%8C%EB%AA%A9-%EA%B8%B0%EB%8A%A5-%EA%B8%B0%EC%88%A0-%EB%AC%B8%EC%84%9C/)는 구현된 기능에 대해 시간순서대로 작동하는 코드와 인게임 사진을 나열하여 어떻게 구현되었는지를 설명합니다.

- [오브젝트와 전체코드 설명](https://unwoo52.github.io/posts/Team-Project-%EC%B1%84%EA%B4%91-%EB%B0%8F-%EB%B2%8C%EB%AA%A9-%EA%B8%B0%EB%8A%A5-%EC%98%A4%EB%B8%8C%EC%A0%9D%ED%8A%B8%EC%99%80-%EC%A0%84%EC%B2%B4%EC%BD%94%EB%93%9C-%EC%84%A4%EB%AA%85/)는 이 기능과 관련된 오브젝트들과, 오브젝트에 있는 스크립트의 전체 코드를 설명합니다.

기술 문서를 읽으시면 기능 구현에 대해 알 수 있습니다. 기술 문서의 내용이나 글이 미흡해 기능의 내용을 잘 파악하지 못했다면, 선택적으로 오브젝트와 전체코드 설명을 추가로 참고하며 읽으실 수 있습니다.

## 버프 시스템

![imagename](/assets/image/Project/TeamProject/SurvivalProjectAboud/000.gif)

> 다양한 종류의 버프를 하나의 Buff Manager(팩토리메소드)에서 생성하도록 구현하였습니다. 버프를 테스트 할 수 있는
>
> 이 외에 플래그 연산을 통해, 지형 버프를 인식하는 비용(플레이어가 움직이는 매 프레임마다 접촉한 지형의 정보를 연산해야 함)을 줄이기 위해 노력하였습니다.


- [기술 문서](https://unwoo52.github.io/posts/Team-Project-Buff-%EA%B8%B0%EB%8A%A5-%EA%B8%B0%EC%88%A0-%EB%AC%B8%EC%84%9C/)는 구현된 기능에 대해 시간순서대로 작동하는 코드와 인게임 사진을 나열하여 어떻게 구현되었는지를 설명합니다.

- [오브젝트와 전체코드 설명](https://unwoo52.github.io/posts/Team-Project-Buff-%EA%B8%B0%EB%8A%A5-%EC%98%A4%EB%B8%8C%EC%A0%9D%ED%8A%B8%EC%99%80-%EC%A0%84%EC%B2%B4%EC%BD%94%EB%93%9C-%EC%84%A4%EB%AA%85/)는 이 기능과 관련된 오브젝트들과, 오브젝트에 있는 스크립트의 전체 코드를 설명합니다.

기술 문서를 읽으시면 기능 구현에 대해 알 수 있습니다. 기술 문서의 내용이나 글이 미흡해 기능의 내용을 잘 파악하지 못했다면, 선택적으로 오브젝트와 전체코드 설명을 추가로 참고하며 읽으실 수 있습니다.

# 팀 활동에 대한 포스트 링크

[Link](https://unwoo52.github.io/posts/Team-Project-About/)

> 팀원간 역할과 분담, 각자 구현한 내용 등에 대한 포스트입니다.

[PPT Link](https://unwoo52.github.io/posts/Team-Project-PPT/)

> 팀 협업의 결과물을 ppt로 설명하였습니다.
