카메라 화면 내에 등장하는 대상들이 휙휙 바뀌어야 하는 프로젝트
키 입력 하나만으로 보이는 장면이 바뀌어야 한다.

disable을 사용하면 너무 많은 비용을 소모한다. 화면 전환 할 때 마다 렉이 걸릴 수 있다고 생각

cam의 layer을 사용하기로 함

문제 발생

특정 레이어 외에는 cam에 잡히지 않음

여러 방면으로 시도를 해봄

모든 layer을 테스트해봄. 전혀 규칙적이지 않게 일부 layer만 편파적으로 culling mask에 잡힘

아마 이전에 설치한 에셋에서 변경된 프로젝트 셋팅에 답이 있지 않을까?

Version Controller로 변경 내역을 확인

첫번째 시도

/ProjectSettings/DynamicsManager.asset에 m_LayerCollisionMatrix에서 변경 내역이 있는 것을 발견

근데 Unity 내의 ProjectSettings의 어디에 있는 옵션인지 찾을 수 없음

rider로 실행, show in unity 버튼을 눌러서 유니티에서 실행되는 것을 확인할 수 있었음

LayerCollisionMatrix... 분명 이건 아니다. 충돌 처리 등에 활용되는 부분이기도 하고, 체크도 이상 없음

근데 혹시 이 많은 체크박스들과 LayerCollisionMatrix이 서로 다른것일 수 도 있지 않냐. 확인이 필요하다.

그래도 검증을 위해, 셋팅을 바꾼 후 Version Controller를 확인, LayerCollisionMatrix의 변경 내역은 이 부분을 변경한것이 확실하다. 안심.
