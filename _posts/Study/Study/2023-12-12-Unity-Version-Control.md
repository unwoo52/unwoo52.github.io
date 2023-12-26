---
title: Unity Version Control
author: unwoo52
date: 2023-12-12 00:00:00 +09:00
categories: [UnityVersionControl]
tags: [UnityVersionControl, VCS, Git]
---

# Unity Version Control

### 대중적인 VCS들

유니티 개발을 위한 버전 관리를 위해 다양한 VCS를 사용할 수 있다.

나는 처음에는 GitDesktop을 사용했고, 그 뒤에는 fork를 사용했다.

rider을 구입한 뒤에는 rider에서 제공하는 VCS와 fork를 병행해서 사용했다.

### 유니티 협업 개발을 위해선 부족해

그런데 유니티 개발을 하게 되면 이런 VCS만으로는 부족한 점들을 느낄 때가 많다. 내 첫 직장에서 같은 파일을 두 명 이상이 작업하느라 충돌이 발생하는 것을 빈번히 보았다.

이러한 문제를 해결하기 위해 더 전문적인 VCS가 몇개 있다는 것을 알게 되었다.

바로 Collaborate나 plastic scm다.

그런데 plastic scm은 지원을 중단했다. 대신 이를 계승하고 브랜드명을 변경한 unty version control이라는 것이 생겼다.

### Unity Version Control

유니티 에디터 창 안에서 실행 가능하고, UGS(Unity Game Service)와 연동되며 유니티에 더 전문적인 개발 관리 툴 이다.

> * 병렬 개발 문제를 해소 가능
> * 대규모 파일 및 저장소에서도 높은 성능과 반응성을 제공
> * 아티스트와 프로그래머 협업을 더 매끄럽게 도와줌
>   * (개발자는 중앙 집중형 또는 분산형 환경에서 완전 브랜칭 및 병합 솔루션을 활용하며 코딩을 할 수 있고, 아티스트는 파일 기반 워크플로와 직관적 UI를 사용하며 창작할 수 있습니다.)
> * JIRA, Rider, TeamCity, Jenkin 등의 IDE, 이슈 트래킹, 협업 및 DevOps 툴과도 연동
> * 브랜칭과 병합 모두 지원
> * 전용 클라우드 서버를 설정하여 전 세계 어디서든 팀이 협업하도록 지원
> * 3명과 월 5GB으로 무료로 시작할 수 있으며, 추가 요금은 인원수와 저장소 용량 추가마다 붙는다.
>   * [정가 가격 세부 정보](https://unity.com/kr/products/unity-devops#pricing)

