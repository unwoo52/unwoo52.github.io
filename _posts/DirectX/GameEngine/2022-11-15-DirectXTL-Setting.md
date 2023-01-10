---
title: DirectX Tool Kit 사용하기
author: unwoo52
date: 2022-11-15 00:00:00 +09:00
categories: [DirectX, GameEngine ]
tags: [directX, Win32API, GameEngine, DirectXTK]
---

# DirectX Tool Kit 받기 [ DirectX로 게임 엔진 만들기 ]
## 1. directXTK 받기
구글에 DirectXTK를 검색해서 나오는 github에 접속. 

[DirectXTK github](https://github.com/microsoft/DirectXTK)

마이크로소프트에서 공식적으로 지원하는 라이브러리이다. 통채로 다운받아서 원하는 라이브러리만 분리해 내 프로젝트에 이식해서 사용하면 된다.

![imagename](/assets/image/DirectX/GameEngine/DirectXTL-Setting/DirectX_Engine_Post_001.png)

폴더를 열고 들어가 자신이 사용하는 비주얼스튜디오의 버전을 찾아 클릭해 실행한다.

![imagename](/assets/image/DirectX/GameEngine/DirectXTL-Setting/DirectX_Engine_Post_002.png)

툴킷을 사용하기 위해서는 빌드를 해서 파일로 만들어서 옮겨야 한다.

비주얼스튜디오에는 실행하는 옵션이 위의처럼 두개가 있는데, **Debug의 32비트와 64비트, release의 32비트와 64비트** 이렇게 4개를 빌드하면 된다.

![imagename](/assets/image/DirectX/GameEngine/DirectXTL-Setting/DirectX_Engine_Post_003.png)

각각의 옵션을 선택해서 4번 빌드해준다.

![imagename](/assets/image/DirectX/GameEngine/DirectXTL-Setting/DirectX_Engine_Post_004.png)

Inc 폴더에 각각의 헤더파일들이 모여있다. 카피해서

![imagename](/assets/image/DirectX/GameEngine/DirectXTL-Setting/DirectX_Engine_Post_005.png)

내가 만들 프로젝트로 옮겨주고 이름을 변경해준다.

![imagename](/assets/image/DirectX/GameEngine/DirectXTL-Setting/DirectX_Engine_Post_006.png)

Libs라는 이름으로 라이브러리를 옮길 폴더를 만들고 이 밑에 32비트와 64비트 폴더를 만든다.

![imagename](/assets/image/DirectX/GameEngine/DirectXTL-Setting/DirectX_Engine_Post_007.png)

그리고 32비트에 Debug와 Release, 64비트에도 Debug와 Release폴더를 생성한다.

여기에 방금 빌드한 라이브러리들을 넣으면 된다.

![imagename](/assets/image/DirectX/GameEngine/DirectXTL-Setting/DirectX_Engine_Post_008.png)

처음에 받았던 DirectXTK에 bin\Descktop20**\에 가서 Debu와 32비트와 64비트, Release의 32비트와 64비트에 있는 DirectXTk.lib 파일들을

![imagename](/assets/image/DirectX/GameEngine/DirectXTL-Setting/DirectX_Engine_Post_009.png)

내 프로젝트의 각자 위치에 맞게 옮겨준다.

이제 Libs밑의 4개의 폴더에 각각에 맞는 라이브러리 파일들이 들어가게 되었다.

이제 툴킷 프로젝트는 삭제한다.

## 2. 프로젝트 내에서 라이브러리 사용할 수 있게 하기

아직 파일만 넣어두었을 뿐, 프로젝트에서 라이브러리들을 사용하지는 않고있다. 라이브러리를 사용할 수 있게 프로젝트 속성을 수정해보자.

솔루션탐색기를 실행시키고 프로젝트 속성창을 킨다.

솔루션탐색기 > "내 프로젝트 이름" 우클릭 > 속성 > VC++ 디렉터리


#### *1. 라이브러리 디렉터리 편집*

![imagename](/assets/image/DirectX/GameEngine/DirectXTL-Setting/DirectX_Engine_Post_010.png)

일반 > 라이브러리 디렉터리 > 편집

![imagename](/assets/image/DirectX/GameEngine/DirectXTL-Setting/DirectX_Engine_Post_011.png)

...으로 뜨는 버튼을 클릭하면 경로를 지정할 수 있다.

![imagename](/assets/image/DirectX/GameEngine/DirectXTL-Setting/DirectX_Engine_Post_012.png)

사진의 1과 2의 옵션을 변경해가며 이번에도 Debu와 32비트와 64비트, Release의 32비트와 64비트에 4번에 걸쳐 경로를 지정해준다.

$(SolutionDir)ProjectName\경로... 를 통해서 경로를 지정한다.

여기서 $(SolutionDir)의 역할은 프로젝트의 위치가 바뀌어도 상관없게 프로젝트 최상위 폴더를 기준으로 탐색을 수행하게 해준다.

#### *2. 포함 디렉터리 편집

일반 > 포함 디렉터리 > 편집

![imagename](/assets/image/DirectX/GameEngine/DirectXTL-Setting/DirectX_Engine_Post_013.png)

구성과 플랫폼을 모든구성과 모든플랫폼으로 설정한 후 포함 디렉터리를 편집한다.

![imagename](/assets/image/DirectX/GameEngine/DirectXTL-Setting/DirectX_Engine_Post_014.png)

다음과 같은 경로로 수정한다.

## 3. 빌드해보기

![imagename](/assets/image/DirectX/GameEngine/DirectXTL-Setting/DirectX_Engine_Post_015.png)

라이브러리를 사용할 수 있게 한 뒤 빌드를 해보자.

![imagename](/assets/image/DirectX/GameEngine/DirectXTL-Setting/DirectX_Engine_Post_016.png)

![imagename](/assets/image/DirectX/GameEngine/DirectXTL-Setting/DirectX_Engine_Post_017.png)

성공적으로 사용할 수 있다는 것을 확인할 수있다!

## 연결된 포스트

>- 다음 포스트
>
>[애플리케이션 상태 관리](https://unwoo52.github.io/posts/%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98-%EC%83%81%ED%83%9C-%EA%B4%80%EB%A6%AC/)
