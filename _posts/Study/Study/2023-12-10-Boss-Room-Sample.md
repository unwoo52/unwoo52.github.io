---
title: Boss Room Sample
author: unwoo52
date: 2023-12-10 00:00:00 +09:00
categories: [UnityMultiplayer, Multiplayer, NetCode, BossRoom]
tags: [UnityMultiplayer, Multiplayer, NetCode, BossRoom]
---

# [Boss Room Sample](https://docs-multiplayer.unity3d.com/netcode/current/learn/bossroom/bossroom/)

넷코드와 유니티 멀티플레이를 이용한 Boss Room Sample은 유니티에서 제공하는 멀티플레이 게임 구현 학습을 위한 샘플이다.

Boss Room Sample은 최대 8명의 플레이어가 플레이어블 가능한 캐릭터를 조종하고 협동하여 보스를 물리치는 게임이다.

---

### 같이 읽은 포스터들


#### Boss Room 실행을 위한 UGS 서비스 목록
Boss Room은 UGS의 여러 서비스를 활용하여 플레이어 간의 연결을 용이하게 합니다. 프로젝트 내에서 이러한 서비스를 사용하려면 다음을 수행해야 합니다.

Unity 대시보드 내에서 조직을 생성하세요 .

> 개인을 사용

릴레이 서비스를 활성화합니다 .

> [Get started with Relay](https://docs.unity.com/ugs/manual/relay/manual/get-started)

+ [Use Relay with Netcode for GameObjects (NGO)](https://docs.unity.com/ugs/en-us/manual/relay/manual/relay-and-ngo)

로비 서비스를 활성화합니다 .

> [Game lobby sample](https://docs.unity.com/ugs/manual/lobby/manual/game-lobby-sample)


<br>


#### 멀티플레이 게임 테스트를 위한 문서

[멀티플레이어 게임을 로컬에서 테스트하기](https://docs-multiplayer.unity3d.com/netcode/current/tutorials/testing/testing_locally/)

> [ParrelSync](https://docs-multiplayer.unity3d.com/netcode/current/tutorials/testing/testing_locally/#parrelsync)는 유니티 editor에서 편집기 창을 열어서 프로젝트 빌드를 하지 않고 멀티플레이 테스트를 할 수 있는 오픈소스 확장 기능. 몹시 유용하다!
> <br> <br> 단, 각 ParrelSync 클론이 동일한 사용자로 로그인하므로 특정 시나리오를 테스트하기가 어렵다.
> <br> <br> 이를 해결하기 위해 이 오픈소스를 사용하지 않고 빌드를 통해 테스트 하거나, 혹은 [인증 SDK의 프로필 관리](https://docs.unity.com/authentication/ProfileManagement.html)를 사용하면, 각 복제본이 서로 다른 ID를 사용하도록 강제할 수 있다.
> ```css
> // ParrelSync should only be used within the Unity Editor so you should use the UNITY_EDITOR define
> #if UNITY_EDITOR
> if (ParrelSync.ClonesManager.IsClone())
> {
> // When using a ParrelSync clone, switch to a different authentication profile to force the clone
> // to sign in as a different anonymous user account.
> string customArgument = ParrelSync.ClonesManager.GetArgument();
> AuthenticationService.Instance.SwitchProfile($"Clone_{customArgument}_Profile");
> }> #endif
> ```

> 이 외에도 패키지에 대한 변경 사항을 동기화를 하지 않는 문제가 있다. 자세한 내용은 [ParrelSync](https://docs-multiplayer.unity3d.com/netcode/current/tutorials/testing/testing_locally/#parrelsync)를 읽으면서 확인할 것.

<br>

#### add post...

write text...

---

### 결과

결과...
