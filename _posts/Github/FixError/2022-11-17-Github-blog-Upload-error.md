---
title: Github blog Upload error
author: unwoo52
date: 2022-11-17 00:00:00 +09:00
categories: [Github, FixError ]
tags: [Github, FixError]
---

어느 순간부터 작성한 포스트가 블로그에 업로드되지 않는 것을 확인

![imagename](/assets/image/Github/FixError/001.png)

저장소로 가서 상단 탭의 Actions로 가보자 한 시점부터 에러로 인해 업로드가 안되는 것을 발견했다.

![imagename](/assets/image/Github/FixError/002.png)

해당 build기록을 클릭한 후 Annotations의 빨강색 에러 마크 클릭

![imagename](/assets/image/Github/FixError/003.png)

*posts/애플리케이션-상태-관리* 파일의 해당 줄에서 에러가 발생하였다.

![imagename](/assets/image/Github/FixError/004.png)

해당 포스트로 가서 확인해보니 링크의 괄호가 두번 쳐져있는 상태....

![imagename](/assets/image/Github/FixError/006.png)

해결!
