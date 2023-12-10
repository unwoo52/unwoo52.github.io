---
title: Learn NetCode with GoldenPath I
author: unwoo52
date: 2023-12-09 00:00:00 +09:00
categories: [UnityMultiplayer, Multiplayer, NetCode]
tags: [UnityMultiplayer, Multiplayer, NetCode]
---

# [Learning with Golden Path - I](https://docs-multiplayer.unity3d.com/netcode/current/tutorials/goldenpath_series/gp_intro/)

[NGO를 이용한 hello world를 만든 것](https://unwoo52.github.io/posts/NetCode-Hello-World/)에 이어서, Golden Path 포스트를 읽고 멀티플레이 네트워킹을 더 구현해보기로 한다.

---

### 같이 읽은 포스트들

[NetworkVariable](https://docs-multiplayer.unity3d.com/netcode/current/basics/networkvariable/)

```css
public class HelloWorldPlayer : NetworkBehaviour
    {
        public NetworkVariable<Vector3> Position = new NetworkVariable<Vector3>();
    }
```

[ServerRpc](https://docs-multiplayer.unity3d.com/netcode/current/advanced-topics/message-system/serverrpc/)

```css
public class HelloWorldPlayer : NetworkBehaviour
{
    [ServerRpc]
    void SubmitPositionRequestServerRpc(ServerRpcParams rpcParams = default)
    {
      Position.Value = GetRandomPositionOnPlane();
    }
}
```

---

### 결과

GUI로 host server client를 선택할 수 있게 했고, Player Prefab이 각 client에서 동기화 되는 것을 확인했다.

[Golden Path Module Two(unity 문서)](https://docs-multiplayer.unity3d.com/netcode/current/tutorials/goldenpath_series/goldenpath_two/#introducing-a-server-controlled-network-variable) 를 이어서 진행하였다. >> [Golden Path Module Two
를 진행한 포스트](https://unwoo52.github.io/posts/Learn-NetCode-with-GoldenPath-II/)

ServerRpc를 사용해 보았다.
