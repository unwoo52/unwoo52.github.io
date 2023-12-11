---
title: Boss Room Sample
author: unwoo52
date: 2023-12-10 01:00:00 +09:00
categories: [UnityMultiplayer, Multiplayer, NetCode, BossRoom]
tags: [UnityMultiplayer, Multiplayer, NetCode, BossRoom]
---

# [Boss Room Sample](https://docs-multiplayer.unity3d.com/netcode/current/learn/bossroom/bossroom/)

넷코드와 유니티 멀티플레이를 이용한 Boss Room Sample은 유니티에서 제공하는 멀티플레이 게임 구현 학습을 위한 샘플이다.

Boss Room Sample은 최대 8명의 플레이어가 플레이어블 가능한 캐릭터를 조종하고 협동하여 보스를 물리치는 게임이다.

---

<br>

## 같이 읽은 포스터들

---

### [Getting Started with Boss Room](https://docs-multiplayer.unity3d.com/netcode/current/learn/bossroom/bossroom/)

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


---

### [Boss Room Architecture](https://docs-multiplayer.unity3d.com/netcode/current/learn/bossroom/bossroom-architecture/)

#### [Assembly structure](https://docs.unity3d.com/kr/2019.4/Manual/ScriptCompilationAssemblyDefinitionFiles.html)

![image](https://github.com/unwoo52/unwoo52.github.io/assets/73688472/ce4015c1-d071-4f32-af20-0f4e260267bf)

#### [Architectural patterns and decisions](https://docs-multiplayer.unity3d.com/netcode/current/learn/bossroom/bossroom-architecture/#architectural-patterns-and-decisions)

* DI ( Dependency Injection )
* [Client/Server code separation](https://docs-multiplayer.unity3d.com/netcode/current/learn/bossroom/bossroom-architecture/#clientserver-code-separation)
* Publisher-Subscriber Messaging

특히 유니티의 boss room 샘플 개발자들은 클라이언트/서버 코드가 클라이언트나 서버의 단일 컨텍스트에서 실행되는 경우가 많다는 점 때문에 많은 고민을 했었다.

boss room 샘플 개발자들은 다양한 클라이언트-서버 코드 분리 접근 방식을 모색했고, 처음에는 클라이언트와 서버별로 어셈블리하기도 했지만, 결과적으로 그들은 고전적인 도메인 기반 어셈블리 아키텍처 방법을 선택했다. >> [Client/Server code separation](https://docs-multiplayer.unity3d.com/netcode/current/learn/bossroom/bossroom-architecture/#clientserver-code-separation)

---

### [Boss Room Actions Walkthrough](https://docs-multiplayer.unity3d.com/netcode/current/learn/bossroom/bossroom-actions/)

#### 즉각 반응 액션 (클라이언트에서 입력과 동시에 예상 행동을 수행하는 경우)

Player Move Image hear...

> 클라이언트가 키 입력을 받음
>
> 서버에 rpc를 보냄과 동시에, 예상 애니메이션과 움직임을 먼저 수행
>
> 서버에서도 rpc를 받아서 pathfinding을 수행, 해당 player prefap의 NetVar을 업데이트
>
> rpc를 보냈던 클라이언트는 서버로부터 NetVar을 받아서 값을 다시 동기화

> 움직임과 같은 즉각적인 반응을 필요로 하는 액션은, 클라이언트에서 입력과 동시에 액션을 실행시킨다.

<br>

#### 매직 볼트(보낸 클라이언트는, 서버에 보낸 rpc를 회신받아서 행동을 수행)

magic bolt image hear...

> 클라이언트가 스킬 키 입력을 받음
>
> rpc를 보냄
>
> 서버에서 입력을 받음, 각 클라이언트에게 rpc를 전송
>
> 보낸 클라이언트를 포함한 클라이언트들은 Magic bolt 효과를 출력, 임프 체력 감소

> 뒤늦게 합류한 클라이언트는 이 Magic Bolt의 애니메이션을 보지 못하지만, 이는 무시할 수 있는 문제.


#### 화살 발사(화살은 NetworkObject이자 NetTransform을 이용해서 동기화)

image hear..

> 화살은 생성된 뒤에 NetObject로 존재하며 계속해서 자기 자신을 동기화

> 뒤늦게 합류한 플레이어도 계속해서 갱신되는 netObject를 볼 수 있음. 화살은 느리게 날라가기 때문에 netObject를 활용함

<br>

Rogue Sneak Image hear...

>client가 스킬 키 입력을 받음
>
>서버에서 해당 player의 netValue(invisible)을 true로 수정
>
>호출한 client와 다른 client들이 수신받아서 '은신 애니메이션'과 '은신 effect'를 출력, NetVal(invisible)을 수정

> 스킬 입력 이후, 클라이언트에서 action을 취하지 않고 서버에서 처리한 후 다시 클라이언트가 받아서 행동을 취함
>
> 그런데 이 행동은 즉각 반응이 불필요해서인지 움직임(패스파인딩을 포함한) 처럼 미리 예상 애니메이션을 먼저 실행시키지 않는다.
>
> 실제로 플레이 해 보니까, 움직임은 두 클라이언트에서 미세한 차이가 있다. 움직임을 실행한 클라이언트에서는 즉각 반응이 일어났지만, 다른 클라이언트는 서버 딜레이만큼 늦게 움직였다.
>
> 그러나 은신은 두 클라이언트가 동시에 애니메이션을 실행하고 은신 이펙트를 출력하는 것을 보았다.

---

### [NetworkObject Parenting inside Boss Room](https://docs-multiplayer.unity3d.com/netcode/current/learn/bossroom/networkobject-parenting/)

> 동적으로 생성된 NetObject는 다른 NetObject의 자식에 위치될 수 없다.
>
> Boss Room은 Server기반 PickUp 시스템을 통해 NetObject 부모지정을 수행한다.
>
> 무기를 장착하는 것에 대해 설명하는 것 처럼 보인다. PositionConstraint이라는 컴포넌트를 이용해서 캐릭터의 손뼈 포지션을 따라가게 구현하여서 마치 해당 트랜스폼의 자식 오브젝트가 된 것 처럼 보이게 할 수 있다는 것이다.

---

### [Boss Room에서의 NetworkRigidbody](https://docs-multiplayer.unity3d.com/netcode/current/learn/bossroom/networkrigidbody/)

> [TossAction.cs](https://github.com/Unity-Technologies/com.unity.multiplayer.samples.coop/blob/v2.2.0/Assets/Scripts/Gameplay/Action/ConcreteActions/TossAction.cs)

---

### [Boss Room 샘플의 성능 최적화](https://docs-multiplayer.unity3d.com/netcode/current/learn/bossroom/optimizing-bossroom/)

#### [RPCs vs. NetworkVariables](https://docs-multiplayer.unity3d.com/netcode/current/learn/bossroom/optimizing-bossroom/#rpcs-vs-networkvariables)

> 위(링크 달 것) Boss Room Actions Walkthrough에서 언급한 내용과 동일. 화살은 NetworkVariables을 사용하고, 매직볼트는 rpc를 사용해서 구현함.

> [RPCs vs NetworkVariables 예제](https://docs-multiplayer.unity3d.com/netcode/current/learn/rpcnetvarexamples/)에서 Boss Room의 UserInput/ClientInputSender.cs코드 등을 보여주며 rpc와 networkVariables의 비교를 더 자세히 보여줌.

##### networkVariables 구성

> [Netcode for GameObjects v1.4.0](https://github.com/Unity-Technologies/com.unity.netcode.gameobjects/releases/tag/ngo%2F1.4.0)에 도입된 NetworkTransform.UseHalfFloatPrecision을 이용해 Character 와 the Arrow prefab 동기화 비용을 5bytes내로 줄일 수 있었다.

> networkVariables는 임계값 이상의 변화가 생길 때 마다 틱단위로 동기화를 실행한다. 이 임계값을 늘려서 덜 정확하고 더 저렴한 비용의 동기화를 구현할 수 있다.

#### UTP 속성

> UTP를 게임 요구사항에 맞게 구성할 수 있다. UnityTransport의 인스펙터에서 이 속성을 수정할 수 있음.
>
> * Disconnect Timeout : 연결을 끊기 전에 서버 및 클라이언트가 기다리는 시간.
> * Max Connect Attempts : 연결 실패 선언까지 클라이언트가 연결을 시도하는 횟수.
> * Connect Timeout : 초당 연결을 시도하는 수.
> * Max Packet Queue Size : 단일 프레임동안 보내고 받을 수 있는 최대 패킷 수 지정.
> * * 클라이언트나 서버가 단일 프레임 동안 최대 패킷 큐 크기 값보다 더 많은 패킷을 보내려고 하면, 다음 프레임 때로 미룬다.
> * * 클라이언트나 서버가 단일 프레임 동안 최대 패킷 큐 크기 값보다 많은 패킷을 수신하는 경우 운영 체제는 추가 패킷을 버퍼링한다.

#### 패킷 크기와 틱 속도 조절

> 패킷 크기 조절
> > Boss Room 샘플에서는 Max Packet Queue Size 기본값이 너무 낮은 것을 발견하여 이를 늘렸습니다. 그러나 Boss Room은 NetworkVariable 업데이트, RPC 및 사용자 정의 메시지에 대해 신뢰할 수 있는 채널만 사용하기 때문에 최대 패킷 큐 크기를 너무 많이 늘리는 것은 의미가 없습니다. 신뢰할 수 있는 채널은 연결당 32개의 전송 중인 패킷만 지원합니다. <br><br>결과적으로, 단일 프레임에서 전송 또는 수신되는 신뢰할 수 있는 최대 패킷 수는 플레이어 수(호스트 제외)에 32(전송 중인 패킷의 최대 수)를 곱한 값입니다. 예를 들어, 8명의 플레이어로 구성된 게임에서는 최대 224(7의 32배)의 안정적인 기내 패킷을 갖게 됩니다. UTP 자체의 내부 트래픽(예: 신뢰할 수 있는 패킷 및 하트비트의 재전송/ACK)에 대해 약간의 여유를 허용하기 위해 약간 더 높은 값인 256을 선택했습니다.
>
> 틱 속도 조절
>
> > Boss Room 샘플에서는 동일한 시나리오를 사용하여 다양한 값을 수동으로 테스트하여 이상적인 틱율 값을 찾았습니다. 시나리오를 정의하기 위해 우리는 NetworkVariables를 사용하는 위치와 원활한 게임 플레이를 제공하기 위해 가장 자주 업데이트해야 하는 항목을 살펴보았습니다. 우리는 NetworkTransforms를 NetworkVariables의 가장 민감한 용도로 식별하여 플레이어가 움직이고, 화살표를 발사하고, 아이템을 던지는 시나리오를 정의했습니다. 우리는 이러한 시나리오의 비디오 캡처를 비교한 결과 클라이언트에게 부드럽고 정확한 게임 플레이를 제공하는 가장 작은 틱 속도 값이 초당 30틱이라는 것을 확인했습니다.

#### 추가 잠재적 최적화 가능성들

> > 위의 모든 최적화에도 불구하고 추가 최적화를 통해 Boss Room 샘플의 대역폭 요구 사항을 더욱 줄일 수 있습니다.
>
> 1. 모든 rpc가 반드시 신뢰도어야 하지는 않는다.
>
> >  네트워크 문제가 있는 클라이언트가 단일 Imp 공격의 시각적 효과를 트리거하는 RPC를 수신하지 못하는 경우에는 그다지 큰 문제가 되지 않습니다. 이는 클라이언트가 문제에서 회복된 후 임프 공격의 영향(위치 업데이트 및 체력 등)을 받기 때문입니다. 결과적으로 이 클라이언트가 이 특정 메시지를 제대로 수신하는지 확인하기 위해 서버와 게임 전체의 속도를 늦출 필요가 없습니다.
>
> 2. 전성되는 데이터의 크기를 줄인다.
>
> > 더 작은 기본 유형을 사용하거나 값을 비트 및 바이트로 인코딩하여 전송된 데이터의 크기를 줄일 수 있습니다. 위치 값은 일반적으로 3x32비트를 사용합니다(Vector3을 구성하는 세 개의 부동소수점에 대해). 그러나 게임에서 데시미터 크기를 초과하는 정밀도는 필수가 아니므로 위치 값에 10을 곱하여 직렬화한 다음 제품의 정수 부분만 인코딩할 수 있습니다. 이렇게 하면 더 적은 비트로 필요한 정밀도를 얻을 수 있습니다.
>
> 3. pathfinding에서 최적화를 하는 예제
>
> > [Pathfinding optimizations](https://docs-multiplayer.unity3d.com/netcode/current/learn/bossroom/optimizing-bossroom/#pathfinding-optimizations)

---

### [Zombie NetworkObjects문제를 해결하기](https://docs-multiplayer.unity3d.com/netcode/current/learn/bossroom/spawn-networkobjects/)

> 뒤늦게 합류한 플레이어의 클라이언트에서, 씬 로드에 의해 생성은 되었지만 NetCode를 통해 파괴되지 못한 오브젝트(즉, 좀비 오프젝트)가 존재할 수 있다.


---

## 결과

결과...
