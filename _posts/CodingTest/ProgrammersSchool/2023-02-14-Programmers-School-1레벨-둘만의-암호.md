---
title: Programmers School 1레벨 둘만의 암호
author: unwoo52
date: 2023-02-14 00:00:00 +09:00
categories: [CodingTest, ProgrammersSchool]
tags: [CodingTest, ProgrammersSchool, Programmers]
---



# 프로그래머스 1레벨 

[둘만의 암호](https://school.programmers.co.kr/learn/courses/30/lessons/155652)

## 문제

두 문자열 s와 skip, 그리고 자연수 index가 주어질 때, 다음 규칙에 따라 문자열을 만들려 합니다. 암호의 규칙은 다음과 같습니다.

문자열 s의 각 알파벳을 index만큼 뒤의 알파벳으로 바꿔줍니다.
index만큼의 뒤의 알파벳이 z를 넘어갈 경우 다시 a로 돌아갑니다.
skip에 있는 알파벳은 제외하고 건너뜁니다.
예를 들어 s = "aukks", skip = "wbqd", index = 5일 때, a에서 5만큼 뒤에 있는 알파벳은 f지만 [b, c, d, e, f]에서 'b'와 'd'는 skip에 포함되므로 세지 않습니다. 따라서 'b', 'd'를 제외하고 'a'에서 5만큼 뒤에 있는 알파벳은 [c, e, f, g, h] 순서에 의해 'h'가 됩니다. 나머지 "ukks" 또한 위 규칙대로 바꾸면 "appy"가 되며 결과는 "happy"가 됩니다.

두 문자열 s와 skip, 그리고 자연수 index가 매개변수로 주어질 때 위 규칙대로 s를 변환한 결과를 return하도록 solution 함수를 완성해주세요.

## 입력

|s	|skip	|index|	result|
|-|-|-|-|
|"aukks"	|"wbqd"	|5	|"happy"|


## 출력

입출력 예 #1
본문 내용과 일치합니다.



## 코드

```cpp
#include <iostream>

//--------

#include <string>
#include <vector>
using namespace std;

void CheckASCII(char& paramChar) {
    if (paramChar > 'z') {
        paramChar = 'a';
    }
}

void IndexSearchingLoof(char& c, int index, string skip)
{
    int indexCount = 0;
    while (indexCount < index) {
        CheckASCII(++c);
        if (skip.find(c) == string::npos) {
            indexCount++;
        }
    }
}

string solution(string s, string skip, int index) {
    string answer = "";

    for (char c : s) {
        IndexSearchingLoof(c, index, skip);
        answer += c;
    }

    return answer;
}

//--------

int main() {
    cout << solution("aukks", "wbqd", 5);
}
```
