---
title: UGS Development Plan
author: unwoo52
date: 2024-02-08 00:00:00 +09:00
categories: [Project, PrivateProject, Project2D3D]
tags: [Project, Project2D3D, 2D, 3D]
---

## 2D와 3D 필드 배치 자동화

### 기술 실증 가능성 검색

팔레트에서 각 격자의 대상을 갖고 올 수 있는가?

성공

### 디버거 구현

Scene 화면에서 클릭을 통해 팔레트 이름을 검색할 수 있는 간단한 코드를 구현

## 애니메이션 구현 자동화

모든 스프라이트 분할이 동일하고, 각 몬스터의 애니메이션이 동일하게 구성될 수 있기 때문에 자동화가 가능하다고 판단

0~2, 3~5, 6~8, 9~11 번호가 각각 up left right down 방향으로의 move애니메이션이 되며,
0~1, 3~4,, 이하 같음 이 up left right down 방향의 idle 애니메이션이 된다.

위 8개의 애니메이션 생성 자동화와 애니메이터 배치까지 자동화를 구현해보겠다.
