---
title: 해시함수
author: unwoo52
date: 2023-01-03 10:00:00 +09:00
categories: [MathContent, Algorithm ]
tags: [MathContent, Algorithm, HashTable]
---

## 해시 함수

- 입력 원소가 해시 테이블 전체에 고루 저장되어야 한다.
- 계산이 간단해야 한다.

첫번째 성질이 가장 중요하다. 이 성질이 잘 만족되어야만 충돌이 적어지기 때문이다. 두번째 성질은 대게 신경쓰지 않아도 쉽게 만족한다. 곱하기 연산과 나눗셈 연산 등이 있다.

## 충돌

충돌은 해시 테이블 이론의 핵심이다.

## 충돌 해결 1. 체이닝

해시 연산 후 접근한 장소에 이미 원소가 존재한다면, 연결리스트로 매달아서 보관한다.

## 충돌 해결 2. 개방 주소 방법

개방 주소 방법에서는 접근한 장소에 이미 원소가 있다면, 정해진 방법에 따라 새로운 장소를 다시 찾는 것을 반복한다. 이런 개방 주소 방법에서 주소를 결정하는 방법으론 대표적으로 세가지 방법이 있다.

#### 2-1 선형 조사

가장 간단한 방법으로, 충돌이 일어난 장소 바로 뒤를 확인하는 것 이다. 테이블 경계를 넘어갈 경우, 가장 앞으로 다시 돌아간다.

- 단점

특정 영역에 원소가 몰릴 때에는 치명적으로 성능이 떨어진다. 이런 현상을 1차 군집이라고 한다.

#### 2-2 이차원 조사

선형 조사와 같이 뒤의 장소가 비어있는지를 확인하되, 뒤로 움직이는 보폭을 이차함수로 늘려나간다. 

- 단점

여러개의 원소가 동일한 초기 해시 함수값을 갖게 되면 모두 같은 순서로 조사를 할수밖에 없어 비효율적이 된다. 즉, 2차 군집이 형성되는 것이다.

#### 2-3 더블 해싱

충돌이 발생하면 또 다른 두번째 해시 함수값을 이용해 점프하는 것이다.

더블해싱에서 조심해야 하는 점은 두 번째 해시 함수 값f(x)가 테이블 크기와 서로 소인 값이여야 한다. 만약 f(x)와 해시 테이블 크기 m이 최소공약수d를 가지면 1/d밖에 보지 못하게 된다.

대게 해시테이블 크기m을 소수값으로 만들고, f(x)의 값은 m보다 작은 자연수가 되도폭 하면 쉽게 만족한다.

## 개방 주소 방법의 확장

적재율이 높아지면 효율이 급격히 떨어지므로, 적절한 임계점을 설정한 후, 이를 넘으면 해시 테이블 크기를 대략 두배로 키우고 모든 원소를 다시 해싱하는 경우가 일반적이다.

## 개방 주소 방법에서 삭제시 문제점

개방 주소 방법을 사용하면서 원소를 삭제하였을 때, delete 표시를 하여야 한다.
