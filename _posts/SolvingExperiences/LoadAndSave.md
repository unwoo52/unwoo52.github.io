﻿로드 앤 세이브에 개방폐쇄 원칙을 도입해보려 시도

GameData라는 큰 클래스를 만들고, 여기에 데이터가 모이게 해봄
새로운 기능을 추가해도, GameData는 이 추가된 class에 대한 구체적인 내용을 알 필요 없음

object[]로 먼저 시도. 여기에 serialize class를 넣는 방식으로 구현하려 해봄

실패
json은 제네릭으로써의 object[]는 저장하지 않음
빈 {}만 생성

개선방향
GameDataBase라는 구체적인 SuperClass를 만들고, 앞으로 만들 데이터들의 클래스를
이 클래스를 상속받게 해봄

성공

그런데 더 개선할 수 있었음

처음부터 string[]으로 변경하고, 데이터를 받을 때 type명 구분자를 앞에 넣은 string(json으로 변환된)
들을 모이게 해서 저장해봄.

GameDataBase는 안써도 되었었다. 처음부터 쉬운 길이었는데 돌아간 것 이었음.

