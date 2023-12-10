---
title: Learn NetCode with GoldenPath II
author: unwoo52
date: 2023-12-10 00:00:00 +09:00
categories: [UnityMultiplayer, Multiplayer, NetCode]
tags: [UnityMultiplayer, Multiplayer, NetCode]
---

# [Learning with Golden Path - II](https://docs-multiplayer.unity3d.com/netcode/current/tutorials/goldenpath_series/goldenpath_two/#introducing-a-server-controlled-network-variable)

[Learning with Golden Path - I](https://unwoo52.github.io/posts/Learn-NetCode-with-GoldenPath-I/)를 진행한것에 이어서 , Golden Path 포스트를 읽고 멀티플레이 네트워킹을 더 구현해보기로 한다.

---

### 같이 읽은 포스트들

[ClientRpc](https://docs-multiplayer.unity3d.com/netcode/current/advanced-topics/message-system/clientrpc/)

```css
public class RpcTest : NetworkBehaviour
    {
        [ClientRpc]
        void TestClientRpc(int value)
        {
            if (IsClient)
            {
                Debug.Log("Client Received the RPC #" + value);
                TestServerRpc(value + 1);
            }
        }

        [ServerRpc]
        void TestServerRpc(int value)
        {
            Debug.Log("Server Received the RPC #" + value);
            TestClientRpc(value);
        }
    }
```

<br>

[RPC Params](https://docs-multiplayer.unity3d.com/netcode/current/advanced-topics/message-system/rpc-params/)

```css
private void DoSomethingServerSide(int clientId)
{
    // If isn't the Server/Host then we should early return here!
    if (!IsServer) return;


    // NOTE! In case you know a list of ClientId's ahead of time, that does not need change,
    // Then please consider caching this (as a member variable), to avoid Allocating Memory every time you run this function
    ClientRpcParams clientRpcParams = new ClientRpcParams
    {
        Send = new ClientRpcSendParams
        {
            TargetClientIds = new ulong[]{clientId}
        }
    };

    // Let's imagine that you need to compute a Random integer and want to send that to a client
    const int maxValue = 4;
    int randomInteger = Random.Range(0, maxValue);
    DoSomethingClientRpc(randomInteger, clientRpcParams);
}

[ClientRpc]
private void DoSomethingClientRpc(int randomInteger, ClientRpcParams clientRpcParams = default)
{
    if (IsOwner) return;

    // Run your client-side logic here!!
    Debug.LogFormat("GameObject: {0} has received a randomInteger with value: {1}", gameObject.name, randomInteger);
}
```

<br>

[Struct ClientRpcParams](https://docs.unity3d.com/Packages/com.unity.netcode.gameobjects@1.7/api/Unity.Netcode.ClientRpcParams.html)

```css
...
DoSomethingClientRpc(randomInteger, clientRpcParams);


[ClientRpc]
private void DoSomethingClientRpc(int randomInteger, ClientRpcParams clientRpcParams = default)
{
  if (IsOwner) return;

// Run your client-side logic here!!
Debug.LogFormat("GameObject: {0} has received a randomInteger with value: {1}", gameObject.name, randomInteger);
}
...
```

---

### 결과

Network Transform을 이용해서 플레이어 오브젝트를 움직이게 해 보았다.

서버 제어 네트워크 변수를 추가 해 보았다. 플레이어가 서버에 접속한 뒤 부터 0.5초마다 경과 시간을 출력하는 디버그 로그를 출력하였다.

ClientRpc를 사용 해 보았다.
