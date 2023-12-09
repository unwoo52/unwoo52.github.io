---
title: NetCode Hello World
author: unwoo52
date: 2023-12-09 00:00:00 +09:00
categories: [UnityMultiplayer, Multiplayer, NetCode]
tags: [UnityMultiplayer, Multiplayer, NetCode]
---

# NetCode Hello World

멀티플레이 게임을 개발하기 위해 근 일주일동안 정보를 찾으면서 어떻게 개발 공부를 시작하면 되는지 찾아보았다.

유니티 엔진을 이용해 멀티플레이 게임을 구현할 수 있는 다양한 서비스들(유니티 멀티플레이, 포톤, 아마존 게임리프트 등)이 있음을 알게 되었고, 그 중 내게 적합한 방법이 무엇인지 고민해 본 결과, 교육 샘플 및 기술 문서가 잘 작성되어 있는 유니티 멀티플레이를 사용해 구현하기로 결정하였다.

그래서 이번 포스트에서는 NetCode를 이용하여 Hello World를 출력하는 프로그램을 만들어 보았다.

### [NetCode Hello World](https://docs-multiplayer.unity3d.com/netcode/current/tutorials/helloworld/index.html)

위 링크 문서를 따라 NetCode 학습을 진행하였다.


---------

### Hello World 포스트를 읽으면서 같이 읽은 문서들

[Install Netcode for GameObjects](https://docs-multiplayer.unity3d.com/netcode/current/installation/)

[네트워크 매니저](https://docs-multiplayer.unity3d.com/netcode/current/components/networkmanager/)

[Transport](https://docs-multiplayer.unity3d.com/netcode/current/advanced-topics/transports/)

---------

### 결과

CMD로 유니티 빌드를 실행하여서 server와 client build를 실행하는 것을 성공하였다.

다음과 같은 항목을 수행하였다.

1. Package Manager에서 NetCode를 설치
2. NetworkManager 추가 및 NetworkManager의 NetworkTransport에 Unity Transport를 연결
3. CLI로 빌드를 실행할 때 옵션을 받아 오는 방법을 알게 됌

포스트 가장 하단의 [Learning with Golden Path](https://docs-multiplayer.unity3d.com/netcode/current/tutorials/goldenpath_series/gp_intro/)를 이어서 학습하여야 한다.

