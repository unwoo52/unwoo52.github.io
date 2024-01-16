### bullet fire RPC method
서버는 RPC를 수신받으면, 발사를 실행한 owner 플레이어를 제외한 플레이어들에게 총알 발싸 RPC 함수가 실행되게 함

발사 위치는 노즐이기 때문에 변수로 발사한 owner의 id와 노즐의 vector3좌표를 받는다

### hit
총알을 발사하는 owner의 클라이언트에서는 예측 효과를 바로 실행(총알 발사와 궤적 표시 및 피격 효과 출력)

하지만 체력 감소는 서버에서 처리 후 수신받게

### health point sync

health point와 같은 정보는 netVar인 것은 맞는데, 이 변수가 있는 NetBehaivor은... 전부 enable상태인가?

boss room sample을 보니 맞다.

### todo

fps framework에서 1.총알 발사 부분, 2.총알 피격 부분, 3.health point 계산 부분을 찾아야 함

총알 발사 이벤트에서 데미지 계산 부분은 client에서는 없애고, server에서만 계산하게

